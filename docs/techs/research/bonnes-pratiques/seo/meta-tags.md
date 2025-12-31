# Bonnes pratiques SEO pour meta tags dans Nuxt 4

L'implémentation SEO dans Nuxt 4.2.x avec SSG et Cloudflare Pages repose sur **Unhead v2.0.13**, le composable `useSeoMeta()` étant la méthode recommandée pour les meta tags. Ce guide couvre la configuration complète pour votre stack technique exacte, avec les breaking changes majeurs de Nuxt 4 et les spécificités de Cloudflare Pages.

## Configuration de base dans nuxt.config.ts

La configuration globale établit les fondations SEO de votre site. L'ordre des modules est **critique** : les modules SEO doivent précéder `@nuxt/content`.

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Modules - L'ORDRE EST IMPORTANT
  modules: [
    'nuxt-schema-org',        // v5.0.10
    '@nuxtjs/sitemap',        // v7.5.0+
    '@nuxt/content'           // v3.10.0+ - TOUJOURS après les modules SEO
  ],

  // Configuration site obligatoire pour canonicals et sitemap
  site: {
    url: process.env.NUXT_SITE_URL || 'https://example.com',
    name: 'Mon Site',
    description: 'Description par défaut du site',
    defaultLocale: 'fr'
  },

  // Schema.org identity
  schemaOrg: {
    identity: {
      type: 'Organization',
      name: 'Ma Société',
      logo: '/logo.png',
      url: 'https://example.com',
      sameAs: [
        'https://twitter.com/moncompte',
        'https://linkedin.com/company/masociete'
      ]
    }
  },

  // Sitemap SSG
  sitemap: {
    zeroRuntime: true,         // Build-time uniquement (SSG)
    autoLastmod: true,
    discoverImages: true,
    sources: ['/api/__sitemap__/urls']
  },

  // Robots.txt
  robots: {
    sitemap: '/sitemap.xml'
  },

  // Nitro preset Cloudflare
  nitro: {
    preset: 'cloudflare-pages',
    prerender: {
      crawlLinks: true,
      routes: ['/', '/sitemap.xml', '/robots.txt']
    }
  },

  // App head defaults
  app: {
    trailingSlash: 'append',   // Match le comportement Cloudflare
    head: {
      htmlAttrs: { lang: 'fr' },
      link: [
        { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
      ]
    }
  }
})
```

## Title tags : format optimal et titleTemplate

Google n'impose pas de limite stricte de caractères pour les titres, mais les **50-60 caractères** (ou **~600 pixels**) évitent la troncature dans les SERPs. Le format `{titre} | {site_name}` reste le pattern le plus efficace.

### Configuration du titleTemplate dans app.vue

Le `titleTemplate` doit être défini dans `app.vue` pour supporter les fonctions dynamiques—impossible dans `nuxt.config.ts`.

```typescript
<!-- app/app.vue -->
<script setup lang="ts">
// Title template global avec fonction (contrôle total)
useHead({
  titleTemplate: (titleChunk) => {
    return titleChunk ? `${titleChunk} | Mon Site` : 'Mon Site'
  }
})

// Meta statiques server-only (performance SSG)
if (import.meta.server) {
  useSeoMeta({
    robots: 'index, follow',
    ogSiteName: 'Mon Site',
    twitterSite: '@moncompte',
    twitterCard: 'summary_large_image'
  })
}
</script>

<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>
```

### useSeoMeta() pour les pages

Le composable `useSeoMeta()` offre **100+ types TypeScript**, une protection XSS native, et évite les erreurs courantes (`name` vs `property`).

```typescript
<!-- app/pages/about.vue -->
<script setup lang="ts">
// ✅ Pattern recommandé Nuxt 4
useSeoMeta({
  title: 'À propos de notre équipe',     // 50-60 caractères max
  description: 'Découvrez notre équipe passionnée de développeurs Nuxt. Experts Vue.js depuis 2018.',
  // og:title NE hérite PAS du titleTemplate - toujours le définir explicitement
  ogTitle: 'À propos de notre équipe | Mon Site',
  ogDescription: 'Découvrez notre équipe passionnée de développeurs Nuxt.',
  ogImage: 'https://example.com/og/about.jpg',
  ogType: 'website'
})
</script>
```

### Anti-patterns title tags à éviter

```typescript
// ❌ MAUVAIS : Titre trop long (sera tronqué)
useSeoMeta({
  title: 'Découvrez notre guide complet sur les meilleures pratiques SEO pour Nuxt 4 en 2025'
})

// ❌ MAUVAIS : Oublier ogTitle (n'hérite pas du template)
useSeoMeta({
  title: 'Ma Page'  // og:title sera vide ou incorrecte
})

// ❌ MAUVAIS : Titre générique/boilerplate
useSeoMeta({
  title: 'Accueil'  // Pas descriptif
})

// ✅ BON : Titre concis + ogTitle explicite
useSeoMeta({
  title: 'Guide SEO Nuxt 4',           // 17 caractères
  ogTitle: 'Guide SEO Nuxt 4 | Mon Site'  // Version complète pour social
})
```

## Meta description : gestion dynamique et fallbacks

Google affiche **120-155 caractères** (desktop) et réécrit les descriptions dans **plus de 70% des cas** si elles ne correspondent pas à la requête utilisateur. La clé : des descriptions uniques, descriptives et alignées avec le contenu réel.

### Pattern avec fallback pour Nuxt Content

```typescript
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
const route = useRoute()
const config = useRuntimeConfig()

const { data: post } = await useAsyncData(
  `blog-${route.path}`,
  () => queryCollection('blog').path(route.path).first()
)

if (!post.value) {
  throw createError({ statusCode: 404, statusMessage: 'Article non trouvé' })
}

// Fallback chain pour description
const description = computed(() => {
  // Priorité : frontmatter > excerpt auto > défaut
  if (post.value?.description) return post.value.description
  if (post.value?.excerpt) return post.value.excerpt.slice(0, 155)
  return `Lisez ${post.value?.title} sur notre blog`
})

// Truncate à 155 caractères max
const truncatedDescription = computed(() => {
  const desc = description.value
  return desc.length > 155 ? `${desc.slice(0, 152)}...` : desc
})

useSeoMeta({
  title: () => post.value?.title,
  description: truncatedDescription,
  ogTitle: () => post.value?.title,
  ogDescription: truncatedDescription,
  ogImage: () => {
    const img = post.value?.image
    if (!img) return `${config.public.siteUrl}/og-default.jpg`
    return img.startsWith('http') ? img : `${config.public.siteUrl}${img}`
  },
  ogType: 'article',
  articlePublishedTime: () => post.value?.date 
    ? new Date(post.value.date).toISOString() 
    : undefined,
  articleAuthor: () => post.value?.author?.name
})
</script>
```

### Composable réutilisable avec fallbacks

```typescript
// app/composables/useContentSeo.ts
interface ContentSeoOptions {
  defaultDescription?: string
  defaultImage?: string
  truncateAt?: number
}

export function useContentSeo(
  content: Ref<any>,
  options: ContentSeoOptions = {}
) {
  const config = useRuntimeConfig()
  const route = useRoute()
  
  const {
    defaultDescription = 'Bienvenue sur notre site',
    defaultImage = '/og-default.jpg',
    truncateAt = 155
  } = options

  const siteUrl = config.public.siteUrl || 'https://example.com'

  const description = computed(() => {
    let desc = content.value?.description 
      ?? content.value?.excerpt 
      ?? defaultDescription
    
    if (desc.length > truncateAt) {
      desc = `${desc.slice(0, truncateAt - 3)}...`
    }
    return desc
  })

  const image = computed(() => {
    const img = content.value?.image ?? content.value?.ogImage
    if (!img) return `${siteUrl}${defaultImage}`
    return img.startsWith('http') ? img : `${siteUrl}${img}`
  })

  const canonical = computed(() => 
    content.value?.canonical ?? `${siteUrl}${route.path}`
  )

  return { description, image, canonical }
}
```

### definePageMeta() vs useSeoMeta()

`definePageMeta()` est une **macro build-time** qui ne peut pas contenir de valeurs dynamiques—utilisez-la uniquement pour les metadata statiques extraites à la compilation.

```typescript
// ✅ BON : definePageMeta pour métadonnées statiques de routing
definePageMeta({
  layout: 'blog',
  middleware: 'auth'
})

// ✅ BON : useSeoMeta pour SEO (supporte réactivité)
const title = ref('Mon titre dynamique')
useSeoMeta({
  title: () => title.value,
  description: () => `Description de ${title.value}`
})

// ❌ MAUVAIS : definePageMeta avec valeurs dynamiques
definePageMeta({
  title: someVariable  // Ne fonctionnera pas - extrait au build
})
```

## Canonical URLs : configuration absolue pour SSG

Chaque page doit avoir une **URL canonique absolue unique**. Google utilise environ **40 signaux** pour déterminer la canonique finale—votre balise est un indice fort mais pas une directive.

### Configuration dans useHead()

```typescript
<script setup lang="ts">
const config = useRuntimeConfig()
const route = useRoute()

// URL canonique absolue
const canonicalUrl = computed(() => {
  // Normaliser : lowercase, trailing slash cohérent
  const path = route.path.toLowerCase()
  const normalizedPath = path.endsWith('/') ? path : `${path}/`
  return `${config.public.siteUrl}${normalizedPath}`
})

useHead({
  link: [
    { rel: 'canonical', href: canonicalUrl }
  ]
})

useSeoMeta({
  ogUrl: canonicalUrl
})
</script>
```

### Intégration avec @nuxtjs/sitemap

Le sitemap doit refléter exactement les URLs canoniques. Configurez la source dynamique pour Nuxt Content :

```typescript
// content.config.ts
import { defineCollection, defineContentConfig, z } from '@nuxt/content'
import { asSitemapCollection } from '@nuxtjs/sitemap/content'

export default defineContentConfig({
  collections: {
    blog: defineCollection(
      asSitemapCollection({
        type: 'page',
        source: 'blog/**/*.md',
        schema: z.object({
          title: z.string(),
          description: z.string().optional(),
          date: z.coerce.date(),
          image: z.string().optional(),
          canonical: z.string().optional(),
          noindex: z.boolean().default(false),
          sitemap: z.object({
            lastmod: z.coerce.date().optional(),
            changefreq: z.enum(['always', 'hourly', 'daily', 'weekly', 'monthly', 'yearly', 'never']).optional(),
            priority: z.number().min(0).max(1).optional()
          }).optional()
        })
      })
    )
  }
})
```

### Frontmatter avec sitemap

```yaml
---
title: 'Guide complet SEO Nuxt 4'
description: 'Apprenez à optimiser le SEO de votre application Nuxt 4.'
date: 2025-12-29
image: /blog/seo-guide/cover.jpg
canonical: https://example.com/blog/guide-seo-nuxt-4/
sitemap:
  lastmod: 2025-12-29
  changefreq: monthly
  priority: 0.8
---
```

### Erreurs canoniques courantes à éviter

```typescript
// ❌ MAUVAIS : URL relative
useHead({
  link: [{ rel: 'canonical', href: '/ma-page' }]
})

// ❌ MAUVAIS : Incohérence trailing slash
// Cloudflare redirige /page → /page/ (308)
// Si canonical pointe vers /page, c'est incohérent
useHead({
  link: [{ rel: 'canonical', href: 'https://example.com/page' }]  // Sans slash
})

// ❌ MAUVAIS : Canonical vers une page noindex
useSeoMeta({ robots: 'noindex' })
useHead({
  link: [{ rel: 'canonical', href: 'https://example.com/page/' }]
})
// Signaux contradictoires !

// ✅ BON : URL absolue, trailing slash cohérent avec Cloudflare
useHead({
  link: [{ rel: 'canonical', href: 'https://example.com/page/' }]
})
```

## Robots directives : index, follow et cas noindex

La directive par défaut `index, follow` n'a pas besoin d'être spécifiée explicitement (c'est le comportement standard). Concentrez-vous sur les cas `noindex`.

### Configuration globale et route rules

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    // Pages à ne pas indexer
    '/admin/**': { robots: 'noindex, nofollow' },
    '/preview/**': { robots: 'noindex' },
    '/merci': { robots: 'noindex' },           // Page confirmation
    '/recherche': { robots: 'noindex' },       // Recherche interne
    
    // Pages paginées : débat en cours
    // Option 1 : noindex (évite duplicate content)
    '/blog/page/**': { robots: 'noindex, follow' },
    // Option 2 : canonical vers page 1 + index
  }
})
```

### Directives par page

```typescript
<script setup lang="ts">
// Page preview/brouillon
const isDraft = computed(() => post.value?.draft === true)

useSeoMeta({
  robots: () => isDraft.value ? 'noindex, nofollow' : 'index, follow'
})
</script>
```

### robots.txt vs meta robots : quand utiliser quoi

| Aspect | robots.txt | Meta robots |
|--------|------------|-------------|
| **Fonction** | Bloque le crawl | Bloque l'indexation |
| **Portée** | Patterns URL | Page spécifique |
| **Si bloqué par robots.txt** | Google ne voit PAS le meta robots | — |
| **Usage SEO** | Économiser le crawl budget | Contrôler l'indexation |

**Point critique** : Si vous bloquez une page via `robots.txt`, Google ne peut pas lire la balise `<meta name="robots">`, donc `noindex` sera ignoré.

```
# public/robots.txt - Configuration SSG
User-agent: *
Allow: /

# Ne pas bloquer les pages que vous voulez noindex
# (Google doit pouvoir les crawler pour voir le meta noindex)
Disallow: /api/
Disallow: /_nuxt/

Sitemap: https://example.com/sitemap.xml
```

### Configuration _headers Cloudflare pour previews

Cloudflare Pages génère des URLs de preview (`*.pages.dev`) qui ne doivent pas être indexées :

```
# public/_headers
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin

# Bloquer l'indexation des preview deployments
https://:project.pages.dev/*
  X-Robots-Tag: noindex

https://*.:project.pages.dev/*
  X-Robots-Tag: noindex

# Cache assets statiques
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

/sitemap.xml
  Cache-Control: public, max-age=3600

/robots.txt
  Cache-Control: public, max-age=3600
```

## Intégration nuxt-schema-org pour structured data

Le module nuxt-schema-org v5.0.10 génère automatiquement les JSON-LD pour Schema.org. L'intégration avec les meta tags renforce la cohérence des signaux SEO.

### Schema Article pour blog posts

```typescript
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
import { defineArticle, useSchemaOrg } from '#imports'

const config = useRuntimeConfig()
const route = useRoute()

// ... fetch post data ...

// Schema.org BlogPosting
useSchemaOrg([
  defineArticle({
    '@type': 'BlogPosting',
    headline: () => post.value?.title,
    description: () => post.value?.description,
    image: () => absoluteImageUrl.value,
    datePublished: () => post.value?.date 
      ? new Date(post.value.date).toISOString() 
      : undefined,
    dateModified: () => post.value?.updatedAt 
      ? new Date(post.value.updatedAt).toISOString() 
      : undefined,
    author: {
      '@type': 'Person',
      name: () => post.value?.author?.name ?? 'Auteur',
      url: () => post.value?.author?.url
    },
    publisher: {
      '@type': 'Organization',
      name: config.public.siteName,
      logo: `${config.public.siteUrl}/logo.png`
    },
    keywords: () => post.value?.tags,
    articleSection: () => post.value?.category,
    inLanguage: 'fr'
  })
])
</script>
```

### Layout avec WebSite et Organization

```typescript
<!-- app/layouts/default.vue -->
<script setup lang="ts">
import { defineWebSite, defineWebPage, useSchemaOrg } from '#imports'

const config = useRuntimeConfig()
const route = useRoute()

useSchemaOrg([
  defineWebSite({
    name: config.public.siteName,
    description: config.public.siteDescription,
    url: config.public.siteUrl,
    inLanguage: 'fr'
  }),
  defineWebPage({
    '@id': `${config.public.siteUrl}${route.path}`,
    url: `${config.public.siteUrl}${route.path}`,
    name: () => useHead().title ?? config.public.siteName
  })
])
</script>
```

## Breaking changes Nuxt 3 → Nuxt 4 pour SEO

### useServerSeoMeta() déprécié

```typescript
// ❌ ANCIEN (Nuxt 3) - Déprécié
useServerSeoMeta({
  description: 'Ma description'
})

// ✅ NOUVEAU (Nuxt 4) - Utiliser import.meta.server
if (import.meta.server) {
  useSeoMeta({
    description: 'Ma description'
  })
}
```

### Configuration generate → nitro.prerender

```typescript
// ❌ ANCIEN (Nuxt 3)
export default defineNuxtConfig({
  generate: {
    routes: ['/page1', '/page2'],
    exclude: ['/admin']
  }
})

// ✅ NOUVEAU (Nuxt 4)
export default defineNuxtConfig({
  nitro: {
    prerender: {
      routes: ['/page1', '/page2', '/sitemap.xml'],
      ignore: ['/admin'],
      crawlLinks: true
    }
  }
})
```

### Structure répertoires app/

Nuxt 4 utilise le dossier `app/` pour le code application :

```
my-nuxt-app/
├─ app/
│  ├─ pages/
│  ├─ components/
│  ├─ composables/
│  ├─ layouts/
│  └─ app.vue
├─ content/
│  └─ blog/
├─ public/
│  ├─ _headers
│  └─ robots.txt
├─ server/
└─ nuxt.config.ts
```

## Exemple complet : page blog dynamique

```typescript
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
import { defineArticle, useSchemaOrg } from '#imports'

const route = useRoute()
const config = useRuntimeConfig()
const siteUrl = config.public.siteUrl

// Fetch article
const { data: post } = await useAsyncData(
  `blog-${route.path}`,
  () => queryCollection('blog').path(route.path).first()
)

if (!post.value) {
  throw createError({ statusCode: 404, statusMessage: 'Article non trouvé' })
}

// Computed avec fallbacks
const title = computed(() => post.value?.title ?? 'Article')
const description = computed(() => {
  const desc = post.value?.description ?? post.value?.excerpt ?? ''
  return desc.length > 155 ? `${desc.slice(0, 152)}...` : desc
})
const image = computed(() => {
  const img = post.value?.image
  if (!img) return `${siteUrl}/og-default.jpg`
  return img.startsWith('http') ? img : `${siteUrl}${img}`
})
const canonical = computed(() => 
  post.value?.canonical ?? `${siteUrl}${route.path}`
)
const publishedDate = computed(() => 
  post.value?.date ? new Date(post.value.date).toISOString() : undefined
)

// Canonical URL
useHead({
  link: [{ rel: 'canonical', href: canonical }]
})

// SEO Meta
useSeoMeta({
  title: title,
  description: description,
  ogTitle: title,
  ogDescription: description,
  ogImage: image,
  ogUrl: canonical,
  ogType: 'article',
  twitterCard: 'summary_large_image',
  twitterTitle: title,
  twitterDescription: description,
  twitterImage: image,
  articlePublishedTime: publishedDate,
  articleAuthor: () => post.value?.author?.name,
  robots: () => post.value?.noindex ? 'noindex' : 'index, follow'
})

// Schema.org
useSchemaOrg([
  defineArticle({
    '@type': 'BlogPosting',
    headline: title,
    description: description,
    image: image,
    datePublished: publishedDate,
    dateModified: () => post.value?.updatedAt 
      ? new Date(post.value.updatedAt).toISOString() 
      : publishedDate.value,
    author: {
      '@type': 'Person',
      name: () => post.value?.author?.name ?? 'Auteur'
    },
    keywords: () => post.value?.tags,
    inLanguage: 'fr'
  })
])
</script>

<template>
  <article v-if="post">
    <header>
      <h1>{{ title }}</h1>
      <time v-if="post.date" :datetime="publishedDate">
        {{ new Date(post.date).toLocaleDateString('fr-FR') }}
      </time>
    </header>
    
    <img 
      v-if="post.image" 
      :src="post.image" 
      :alt="title"
      loading="eager"
    />
    
    <ContentRenderer :value="post" />
  </article>
</template>
```

## Conclusion

L'implémentation SEO dans Nuxt 4.2.x avec SSG sur Cloudflare Pages repose sur trois piliers : `useSeoMeta()` pour les meta tags typés et XSS-safe, `useHead()` pour le titleTemplate et les canonicals, et `nuxt-schema-org` pour les données structurées. Les changements majeurs de Nuxt 4 incluent la dépréciation de `useServerSeoMeta()` (remplacé par `import.meta.server`), la migration vers `nitro.prerender`, et l'importance de configurer `site.url` pour les URLs absolues.

Pour Cloudflare Pages spécifiquement, adoptez les trailing slashes (`app.trailingSlash: 'append'`), utilisez le fichier `_headers` pour bloquer l'indexation des previews, et activez `zeroRuntime: true` dans le sitemap pour une génération purement statique. La cohérence entre canonical URLs, sitemap, et comportement de redirect Cloudflare est essentielle pour éviter les signaux contradictoires vers Google.