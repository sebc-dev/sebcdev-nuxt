# Guide complet routing et middleware Nuxt 4.2.X pour un blog Cloudflare Pages

**L'architecture de routing dans Nuxt 4.2.X a évolué significativement** avec le déplacement du code client dans le dossier `app/` et l'abandon de plusieurs patterns Vue Router classiques. Pour un blog utilisant Nuxt Content 3 sur Cloudflare Pages, les bonnes pratiques reposent sur trois piliers : le file-based routing avec routes catch-all `[...slug].vue`, la validation via `definePageMeta({ validate })` intégrée aux collections Content, et les middleware basés sur les valeurs de retour (`navigateTo`/`abortNavigation`) plutôt que le pattern `next()` obsolète.

Ce guide documente les patterns TypeScript optimaux pour la génération de code avec Claude Code, en tenant compte des contraintes edge runtime de Cloudflare Workers (128MB mémoire, APIs Node.js limitées) et des spécificités de Nuxt Content 3 qui utilise désormais un système de **Collections avec stockage SQL** via D1 Database.

---

## Structure de fichiers recommandée pour un blog Nuxt 4.x

Le changement majeur de Nuxt 4 est le déplacement du code client dans le dossier `app/`. Cette nouvelle structure sépare clairement les préoccupations serveur et client.

```
nuxt-blog/
├── app/
│   ├── components/
│   │   ├── blog/
│   │   │   ├── BlogCard.vue
│   │   │   └── BlogList.vue
│   │   └── content/              # Composants MDC pour Markdown
│   │       └── ProseCode.vue
│   ├── layouts/
│   │   ├── default.vue
│   │   └── blog.vue
│   ├── middleware/
│   │   ├── 01.analytics.global.ts
│   │   ├── 02.redirects.global.ts
│   │   └── auth.ts               # Named middleware
│   ├── pages/
│   │   ├── index.vue                      → /
│   │   ├── blog/
│   │   │   ├── index.vue                  → /blog
│   │   │   ├── [...slug].vue              → /blog/mon-article
│   │   │   └── category/
│   │   │       └── [category].vue         → /blog/category/tech
│   │   └── about.vue                      → /about
│   └── app.vue
├── content/
│   └── blog/
│       ├── 2024/
│       │   └── mon-article.md
│       └── 2025/
│           └── nouvel-article.md
├── server/
│   ├── api/
│   └── middleware/               # Nitro middleware (différent!)
├── content.config.ts             # Configuration Collections Nuxt Content 3
└── nuxt.config.ts
```

Chaque page doit impérativement avoir **un seul élément racine** dans son template pour permettre les transitions de route. Le système de pages est optionnel : créer le dossier `app/pages/` l'active automatiquement.

---

## Paramètres dynamiques et routes catch-all

La syntaxe de routing Nuxt utilise les crochets pour définir les segments dynamiques. Les **crochets simples** `[slug]` définissent un paramètre requis, tandis que les **doubles crochets** `[[slug]]` créent un paramètre optionnel.

```typescript
// app/pages/blog/[...slug].vue - Route catch-all pour Nuxt Content
<script setup lang="ts">
const route = useRoute()

// Pour /blog/2024/tech/mon-article → slug = ['2024', 'tech', 'mon-article']
const pathArray = route.params.slug as string[]
const fullPath = computed(() => `/blog/${pathArray?.join('/')}`)

// Récupération du contenu avec queryCollection (API Nuxt Content 3)
const { data: article } = await useAsyncData(
  `blog-${fullPath.value}`,
  () => queryCollection('blog').path(fullPath.value).first()
)

// Gestion 404 avec fatal: true (important pour SSG)
if (!article.value) {
  throw createError({
    statusCode: 404,
    statusMessage: 'Article non trouvé',
    fatal: true
  })
}

// SEO automatisé
useSeoMeta({
  title: article.value.title,
  description: article.value.description,
  ogImage: article.value.image,
})
</script>

<template>
  <article v-if="article">
    <h1>{{ article.title }}</h1>
    <ContentRenderer :value="article" class="prose" />
  </article>
</template>
```

La différence entre `[...slug].vue` et `[[...slug]].vue` est cruciale : le premier requiert au moins un segment (ne match pas `/blog/`), tandis que le second fonctionne aussi pour la route de base. Pour un blog, utilisez `[...slug].vue` combiné avec un `index.vue` séparé pour la page de liste.

---

## Configuration Nuxt Content 3 avec Collections

Nuxt Content 3 abandonne `queryContent()` au profit de `queryCollection()` et introduit un système de **Collections typées avec Zod**. Cette approche apporte une validation de schéma au build time et un stockage optimisé via SQLite/D1.

```typescript
// content.config.ts (à la racine du projet)
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { z } from 'zod'

export default defineContentConfig({
  collections: {
    blog: defineCollection({
      type: 'page',
      source: 'blog/**/*.md',
      schema: z.object({
        title: z.string(),
        description: z.string(),
        date: z.date(),
        image: z.string().optional(),
        tags: z.array(z.string()).default([]),
        author: z.string().default('Admin'),
        published: z.boolean().default(true),
        readingTime: z.number().optional(),
      }),
      // Index pour optimiser les requêtes fréquentes
      indexes: [
        { columns: ['date'] },
        { columns: ['published'] },
      ]
    }),
    
    categories: defineCollection({
      type: 'data',
      source: 'categories/*.yml',
      schema: z.object({
        name: z.string(),
        slug: z.string(),
        description: z.string(),
      })
    }),
  }
})
```

Les types sont **auto-générés** basés sur ce fichier, permettant un typage complet dans les composants. L'ancienne API `\<ContentDoc\>` et `\<ContentList\>` n'existe plus : utilisez uniquement `\<ContentRenderer\>` pour le rendu.

---

## Validation de routes avec definePageMeta

La fonction `validate` dans `definePageMeta` permet de vérifier les paramètres de route avant le rendu. Elle retourne soit un booléen, soit un objet d'erreur personnalisé avec `statusCode` et `statusMessage`.

```typescript
// app/pages/blog/[...slug].vue
<script setup lang="ts">
definePageMeta({
  validate: async (route) => {
    const slug = route.params.slug
    const path = Array.isArray(slug) ? `/blog/${slug.join('/')}` : `/blog/${slug}`
    
    // Validation format uniquement (léger)
    if (Array.isArray(slug) && slug.some(s => !/^[a-z0-9-]+$/.test(s))) {
      return { statusCode: 400, statusMessage: 'Format de slug invalide' }
    }
    
    // Validation existence via queryCollection (pour pages critiques)
    try {
      const count = await queryCollection('blog')
        .path(path)
        .count()
      
      if (count === 0) {
        return { statusCode: 404, statusMessage: 'Article non trouvé' }
      }
      
      return true
    } catch {
      return { statusCode: 500, statusMessage: 'Erreur serveur' }
    }
  }
})
</script>
```

Pour éviter les requêtes doubles (une dans validate, une dans le composant), privilégiez une validation de **format uniquement** dans `validate`, et gérez l'existence dans le composant avec `createError({ fatal: true })`. Le paramètre `fatal: true` est **indispensable pour SSG** car il garantit la génération d'une page d'erreur appropriée.

---

## Patterns de middleware pour un blog

Nuxt distingue trois types de middleware : **global** (suffix `.global.ts`), **nommé** (chargé à la demande), et **inline** (défini dans `definePageMeta`). L'ordre d'exécution est alphabétique pour les globaux, puis selon l'ordre du tableau pour les autres.

```typescript
// app/middleware/01.analytics.global.ts
// Le préfixe numérique contrôle l'ordre (trié comme string: 01 < 02 < 10)
export default defineNuxtRouteMiddleware((to, from) => {
  // Skip côté serveur et pendant l'hydratation
  if (import.meta.server) return
  
  const nuxtApp = useNuxtApp()
  if (nuxtApp.isHydrating && nuxtApp.payload.serverRendered) return
  
  // Analytics tracking
  if (typeof gtag !== 'undefined') {
    gtag('config', 'GA_MEASUREMENT_ID', { page_path: to.fullPath })
  }
})
```

```typescript
// app/middleware/auth.ts - Named middleware
export default defineNuxtRouteMiddleware((to, from) => {
  const { loggedIn } = useUserSession()
  
  // Éviter boucle infinie : exclure la page de login
  if (to.path === '/login') return
  
  if (!loggedIn.value) {
    // Sauvegarder destination pour redirect post-login
    return navigateTo(`/login?redirect=${encodeURIComponent(to.fullPath)}`)
  }
})

// Usage dans une page protégée
definePageMeta({
  middleware: ['auth']
})
```

Le pattern `next()` de Vue Router **n'existe pas** dans Nuxt 3/4. Le contrôle se fait exclusivement via les valeurs de retour : `return` ou `undefined` continue la navigation, `return navigateTo()` redirige, et `return abortNavigation()` bloque.

---

## Navigation guards et redirections SEO

Les fonctions `navigateTo()` et `abortNavigation()` sont les seuls mécanismes de contrôle de navigation dans les middleware Nuxt. La distinction entre navigation interne et externe est critique.

```typescript
// Redirections SEO pour anciennes URLs
// app/middleware/02.legacy-redirects.global.ts
const redirectMap: Record<string, string> = {
  '/blog/old-post': '/articles/new-post',
  '/category/tech': '/blog/category/technology',
}

export default defineNuxtRouteMiddleware((to) => {
  const newPath = redirectMap[to.path]
  if (newPath) {
    // 301 pour SEO (redirection permanente)
    return navigateTo(newPath, { redirectCode: 301 })
  }
})
```

```typescript
// Suppression trailing slash avec 301
// app/middleware/03.trailing-slash.global.ts
export default defineNuxtRouteMiddleware(({ path, query, hash }) => {
  if (path === '/' || !path.endsWith('/')) return
  
  const nextPath = path.replace(/\/+$/, '') || '/'
  return navigateTo({ path: nextPath, query, hash }, { redirectCode: 301 })
})
```

Pour les URLs externes, l'option `external: true` est **obligatoire** : `navigateTo('https://externe.com', { external: true })`. L'option `redirectCode` n'a d'effet que côté serveur lors du SSR initial ; les navigations client-side sont toujours des transitions SPA.

---

## Configuration Cloudflare Pages optimisée

Le déploiement sur Cloudflare Pages nécessite une configuration spécifique pour Nitro et une compréhension des contraintes edge runtime. Le preset `cloudflare_pages` est détecté automatiquement via l'intégration Git, mais une configuration explicite améliore la prévisibilité.

```typescript
// nuxt.config.ts - Configuration complète pour Cloudflare Pages
export default defineNuxtConfig({
  compatibilityDate: '2024-09-19',
  
  modules: [
    '@nuxt/content',
    '@nuxtjs/sitemap',
  ],

  nitro: {
    preset: 'cloudflare_pages',
    
    cloudflare: {
      deployConfig: true,
      nodeCompat: true, // Active nodejs_compat pour APIs Node.js
      
      pages: {
        routes: {
          // Limite de 100 exclusions dans _routes.json
          exclude: ['/blog/*', '/docs/*']
        }
      }
    },

    prerender: {
      crawlLinks: true,
      routes: ['/', '/sitemap.xml', '/robots.txt'],
      autoSubfolderIndex: false,
    }
  },

  routeRules: {
    '/': { prerender: true },
    '/blog/**': { swr: 3600 }, // SWR 1h (équivalent ISR sur Cloudflare)
    '/admin/**': { ssr: false },
    '/_nuxt/**': { 
      headers: { 'cache-control': 'public, max-age=31536000, immutable' }
    },
  },

  content: {
    database: {
      type: 'd1',
      binding: 'DB' // Binding D1 dans Cloudflare Dashboard
    }
  },

  site: {
    url: 'https://monblog.com',
    name: 'Mon Blog'
  }
})
```

Nuxt Content 3 **requiert D1 Database** sur Cloudflare pour stocker l'index du contenu. Les limites Free tier sont **500MB** de stockage et **10M reads/mois**. Pour les variables d'environnement, n'utilisez jamais `process.env` dans le scope global : accédez-y via `useRuntimeConfig(event)` dans les handlers.

---

## Pièges courants à éviter avec Claude Code

La génération de code pour Nuxt 4.x doit tenir compte de plusieurs changements majeurs par rapport aux versions antérieures et aux patterns Vue Router classiques.

**Ne jamais utiliser `useRoute()` dans un middleware** — les paramètres `to` et `from` fournissent toutes les informations nécessaires. `useRoute()` n'a pas de "route courante" pendant la navigation.

**Toujours retourner `navigateTo()`** — un appel sans `return` n'aura aucun effet. Le middleware continuera son exécution normale.

```typescript
// ❌ INCORRECT - navigateTo sans return
export default defineNuxtRouteMiddleware((to) => {
  if (!isAuth()) {
    navigateTo('/login') // NE BLOQUE PAS la navigation!
  }
})

// ✅ CORRECT
export default defineNuxtRouteMiddleware((to) => {
  if (!isAuth() && to.path !== '/login') {
    return navigateTo('/login')
  }
})
```

**API Nuxt Content 3 vs v2** — `queryContent()` est remplacé par `queryCollection()`, les composants `\<ContentDoc\>` et `\<ContentList\>` n'existent plus, et les préfixes `_path`/`_id` deviennent `path`/`id`.

**Variables réactives dans definePageMeta** — ne jamais référencer de `ref()` car le contenu est hoisted hors du composant au build time.

---

## Bonnes pratiques pour génération Claude Code

Pour optimiser la génération de code avec Claude Code, structurez vos demandes autour de ces patterns documentés et spécifiez explicitement la version Nuxt 4.2.X.

Les commentaires dans le code généré doivent inclure le contexte Cloudflare : `// Edge-compatible: pas d'accès filesystem` ou `// D1 required: queryCollection uses SQL storage`.

Pour les nouveaux fichiers, Claude Code devrait générer systématiquement :
- Le type-checking avec `lang="ts"` dans les scripts
- Les clés uniques pour `useAsyncData` basées sur `route.path`
- Le pattern `fatal: true` dans `createError` pour les pages SSG
- Les imports explicites des types Collections pour l'autocomplétion

La structure `app/` est obligatoire pour Nuxt 4 : tous les composants, pages, middleware, et composables doivent résider sous ce dossier racine, séparant clairement le code client du code serveur dans `server/`.

---

## Conclusion

L'architecture routing/middleware Nuxt 4.2.X pour un blog Cloudflare Pages repose sur une **séparation stricte** entre validation de format (dans `definePageMeta.validate`), logique de navigation (dans les middleware), et récupération de données (dans le composant avec `useAsyncData`).

Les changements clés à retenir : structure `app/` obligatoire, `queryCollection()` remplace `queryContent()`, pattern `return navigateTo()` au lieu de `next()`, et configuration D1 Database indispensable pour Nuxt Content 3 sur Cloudflare. Le pre-rendering via `crawlLinks: true` et les routeRules `swr` optimisent les performances tout en respectant les contraintes edge runtime de 128MB mémoire et 10ms CPU par requête en Free tier.