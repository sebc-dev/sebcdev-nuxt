# Guide technique Nuxt 4.2.x pour blog SSG sur Cloudflare Pages

Ce guide présente les bonnes pratiques actualisées (décembre 2025) pour développer un blog statique avec Nuxt 4.2.x, Vue 3, et Nuxt Content 3, optimisé pour Cloudflare Pages et le développement assisté par Claude Code. Les configurations présentées garantissent d'excellents scores Lighthouse tout en maintenant une expérience développeur fluide.

## Configuration SSG optimale pour Cloudflare Pages

La génération statique dans Nuxt 4 repose entièrement sur Nitro, avec une migration importante des options de configuration par rapport à Nuxt 3. Le preset `cloudflare_pages` reste recommandé pour les blogs SSG, bien que `cloudflare_module` offre des fonctionnalités avancées pour les déploiements hybrides.

### Configuration complète nuxt.config.ts

```typescript
// nuxt.config.ts - Configuration SSG blog pour Cloudflare Pages
export default defineNuxtConfig({
  // Requis pour Nuxt 4 - date de compatibilité des features
  compatibilityDate: '2024-11-01',
  
  // SSR activé obligatoirement pour SSG (défaut: true)
  ssr: true,
  
  // Configuration Nitro pour SSG
  nitro: {
    preset: 'cloudflare_pages',
    
    // Configuration Cloudflare spécifique
    cloudflare: {
      deployConfig: true,    // Génère wrangler.json automatiquement
      nodeCompat: true,      // Compatibilité Node.js
      pages: {
        routes: {
          // Optimise _routes.json (limite 100 routes Cloudflare)
          exclude: ['/blog/*', '/categories/*', '/tags/*']
        }
      }
    },
    
    // Configuration du prerendering
    prerender: {
      crawlLinks: true,         // Découvre automatiquement les liens internes
      routes: [                 // Routes additionnelles à prérender
        '/',
        '/sitemap.xml',
        '/robots.txt',
        '/feed.xml'
      ],
      ignore: [                 // Routes à exclure
        '/api/**',
        '/admin/**',
        '/_nuxt/**'
      ],
      failOnError: false,       // Continue malgré les erreurs
      concurrency: 4,           // Prerendering parallèle
      retry: 3,                 // Tentatives en cas d'échec
      retryDelay: 500
    },
    
    // Correspondance avec les règles Cloudflare
    autoSubfolderIndex: false
  },

  // Règles de routes pour rendu hybride
  routeRules: {
    // Pages statiques pré-rendues
    '/': { prerender: true },
    '/blog/**': { prerender: true },
    '/categories/**': { prerender: true },
    '/tags/**': { prerender: true },
    '/about': { prerender: true },
    
    // Cache statique avec headers optimisés
    '/assets/**': {
      headers: {
        'cache-control': 'public, max-age=31536000, immutable'
      }
    },
    
    // Pages HTML avec revalidation CDN
    '/**': {
      headers: {
        'cache-control': 's-maxage=3600, stale-while-revalidate=86400'
      }
    }
  },

  // Features expérimentales recommandées
  experimental: {
    componentIslands: true,      // Active les Server Components
    payloadExtraction: true,     // Extraction JSON pour navigation client
    inlineRouteRules: true       // defineRouteRules() dans les composants
  },
  
  // Modules recommandés pour un blog
  modules: [
    '@nuxt/content',
    '@nuxt/image',
    '@nuxtjs/seo'
  ]
})
```

### Paramètres Cloudflare Dashboard critiques

Certains paramètres Cloudflare provoquent des erreurs d'hydratation Vue et doivent être désactivés impérativement :

- **Speed > Optimization > Rocket Loader™** : Désactiver
- **Speed > Optimization > Mirage** : Désactiver  
- **Scrape Shield > Email Obfuscation** : Désactiver

Ces optimisations automatiques modifient le HTML rendu et cassent la correspondance entre le HTML serveur et l'hydratation Vue côté client.

### Breaking change majeur : generate vers nitro.prerender

```typescript
// ❌ Nuxt 3 (obsolète dans Nuxt 4)
export default defineNuxtConfig({
  generate: {
    exclude: ['/admin'],
    routes: ['/sitemap.xml']
  }
})

// ✅ Nuxt 4 (correct)
export default defineNuxtConfig({
  nitro: {
    prerender: {
      ignore: ['/admin'],        // Remplace generate.exclude
      routes: ['/sitemap.xml']   // Remplace generate.routes
    }
  }
})
```

Le dossier de sortie change également de `dist/` vers `.output/public/`.

## Vue 3 Composition API avec TypeScript

Le pattern `<script setup lang="ts">` constitue le standard obligatoire pour tout nouveau composant Vue 3 dans Nuxt 4. Cette syntaxe offre une inférence TypeScript complète et élimine le boilerplate.

### Structure de composant standard

```vue
<script setup lang="ts">
// TypeScript + Composition API - standard moderne Vue
// Tous les imports sont automatiquement exposés au template

interface BlogPost {
  id: string
  slug: string
  title: string
  content: string
  publishedAt: Date
  tags: string[]
}

const props = defineProps<{
  post: BlogPost
}>()

const emit = defineEmits<{
  share: [platform: string]
}>()

// Réactivité - auto-importée par Nuxt
const isExpanded = ref(false)
const formattedDate = computed(() => 
  new Intl.DateTimeFormat('fr-FR').format(props.post.publishedAt)
)

// Lifecycle - auto-importé par Nuxt
onMounted(() => {
  console.log('Composant monté')
})
</script>

<template>
  <article>
    <h1>{{ post.title }}</h1>
    <time>{{ formattedDate }}</time>
  </article>
</template>
```

### Patterns de composables réutilisables

Les composables suivent des conventions strictes pour garantir réutilisabilité et maintenabilité. Le préfixe `use` est obligatoire, et le retour doit être un objet contenant des refs (jamais un `reactive`).

```typescript
// app/composables/useBlogPost.ts
import type { MaybeRefOrGetter } from 'vue'

interface BlogPost {
  id: string
  slug: string
  title: string
  content: string
  excerpt: string
  author: { name: string; avatar: string }
  publishedAt: string
  tags: string[]
  readingTime: number
}

interface UseBlogPostReturn {
  post: Readonly<Ref<BlogPost | null>>
  isLoading: Readonly<Ref<boolean>>
  error: Readonly<Ref<Error | null>>
  refresh: () => Promise<void>
}

export function useBlogPost(
  slug: MaybeRefOrGetter<string>
): UseBlogPostReturn {
  // shallowRef pour objets complexes - évite la réactivité profonde
  const post = shallowRef<BlogPost | null>(null)
  const isLoading = ref(false)
  const error = shallowRef<Error | null>(null)

  const fetchPost = async () => {
    const slugValue = toValue(slug) // Unwrap ref, getter ou valeur
    if (!slugValue) return

    isLoading.value = true
    error.value = null
    
    try {
      post.value = await $fetch(`/api/posts/${slugValue}`)
    } catch (e) {
      error.value = e as Error
      post.value = null
    } finally {
      isLoading.value = false
    }
  }

  // watchEffect track automatiquement les dépendances
  watchEffect(() => {
    toValue(slug) // Track les changements de slug
    fetchPost()
  })

  return {
    post: readonly(post),
    isLoading: readonly(isLoading),
    error: readonly(error),
    refresh: fetchPost
  }
}
```

**Utilisation flexible avec MaybeRefOrGetter :**

```vue
<script setup lang="ts">
const route = useRoute()

// Toutes ces syntaxes fonctionnent grâce à MaybeRefOrGetter
useBlogPost('mon-article')                    // Valeur directe
useBlogPost(ref('mon-article'))               // Ref
useBlogPost(() => route.params.slug as string) // Getter réactif
</script>
```

### Réactivité avancée pour grandes listes

Pour les listes dépassant **10 000 éléments**, utilisez `shallowRef` au lieu de `ref` pour éviter la création coûteuse de proxies profonds :

```typescript
// app/composables/useBlogArchive.ts
export function useBlogArchive() {
  // ✅ shallowRef - ne track que .value, pas les propriétés internes
  const posts = shallowRef<BlogPost[]>([])
  
  const addPost = (post: BlogPost) => {
    // Obligation de remplacer le tableau entier pour déclencher réactivité
    posts.value = [...posts.value, post]
  }
  
  const updatePost = (id: string, updates: Partial<BlogPost>) => {
    posts.value = posts.value.map(p => 
      p.id === id ? { ...p, ...updates } : p
    )
  }

  // computed avec comparaison intelligente (Vue 3.4+)
  const stats = computed((oldValue) => {
    const newValue = {
      total: posts.value.length,
      published: posts.value.filter(p => p.status === 'published').length
    }
    
    // Évite de déclencher les effets si valeurs identiques
    if (oldValue?.total === newValue.total && 
        oldValue?.published === newValue.published) {
      return oldValue
    }
    return newValue
  })

  return { posts, addPost, updatePost, stats }
}
```

### Auto-imports Nuxt 4

Nuxt 4 auto-importe automatiquement toutes les APIs Vue et les composables du projet :

```typescript
// Ces imports sont automatiques - ne pas les déclarer explicitement
// ref, computed, watch, watchEffect, onMounted, etc.
// useRoute, useRouter, useFetch, useState, etc.
// Tous les composables dans app/composables/

// Pour import explicite si nécessaire :
import { ref, computed } from '#imports'
```

**Structure des composables pour auto-import :**

```
app/
├── composables/
│   ├── index.ts              // Scanné ✅
│   ├── useBlogPost.ts        // Scanné ✅
│   ├── useBlogSearch.ts      // Scanné ✅
│   └── utils/
│       └── helpers.ts        // NON scanné par défaut
```

Pour scanner les sous-dossiers :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  imports: {
    dirs: ['composables', 'composables/**']
  }
})
```

## Stratégies d'hydratation et performance

Nuxt 4 intègre nativement les directives de lazy hydration introduites dans Nuxt 3.16, permettant un contrôle précis sur le moment où les composants deviennent interactifs côté client.

### Directives de lazy hydration

```vue
<template>
  <div>
    <!-- Hydrate quand visible dans viewport -->
    <LazyCommentsSection hydrate-on-visible />
    
    <!-- Avec options IntersectionObserver -->
    <LazyRelatedPosts :hydrate-on-visible="{ rootMargin: '200px' }" />
    
    <!-- Hydrate quand navigateur idle ou après timeout -->
    <LazyNewsletter :hydrate-on-idle="2000" />
    
    <!-- Hydrate après interaction utilisateur -->
    <LazyShareButtons hydrate-on-interaction />
    
    <!-- Interactions spécifiques -->
    <LazySearchModal :hydrate-on-interaction="['click', 'focus']" />
    
    <!-- Hydrate selon media query -->
    <LazyMobileMenu hydrate-on-media-query="(max-width: 768px)" />
    
    <!-- Hydrate après délai -->
    <LazyFooter :hydrate-after="3000" />
    
    <!-- Hydrate conditionnellement -->
    <LazyAdBanner :hydrate-when="adsEnabled" />
    
    <!-- Jamais hydraté - HTML statique uniquement -->
    <LazyStaticFooter hydrate-never />
  </div>
</template>
```

### Contrôle programmatique avec defineLazyHydrationComponent

```vue
<script setup lang="ts">
// Définition programmatique de composants lazy
const LazyComments = defineLazyHydrationComponent(
  'visible',
  () => import('./components/Comments.vue')
)

const LazyAnalytics = defineLazyHydrationComponent(
  'idle',
  () => import('./components/Analytics.vue')
)

const LazyShareButtons = defineLazyHydrationComponent(
  'interaction',
  () => import('./components/ShareButtons.vue')
)

// Composant jamais hydraté (contenu statique pur)
const LazyFooter = defineLazyHydrationComponent(
  'never',
  () => import('./components/Footer.vue')
)

// Event de fin d'hydratation
function onHydrated() {
  console.log('Composant hydraté!')
}
</script>

<template>
  <LazyComments 
    :hydrate-on-visible="{ rootMargin: '100px' }"
    @hydrated="onHydrated"
  />
</template>
```

### Server Components pour zéro JavaScript client

Les fichiers `.server.vue` sont rendus exclusivement côté serveur avec **zéro JavaScript** envoyé au client :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  experimental: {
    componentIslands: true,
    // Pour composants client imbriqués :
    // componentIslands: { selectiveClient: true }
  }
})
```

```vue
<!-- components/BlogContent.server.vue -->
<script setup>
import MarkdownIt from 'markdown-it'
import Prism from 'prismjs'

const props = defineProps<{
  content: string
}>()

const md = new MarkdownIt({
  highlight: (str, lang) => {
    if (lang && Prism.languages[lang]) {
      return Prism.highlight(str, Prism.languages[lang], lang)
    }
    return ''
  }
})

const renderedContent = md.render(props.content)
</script>

<template>
  <article class="prose" v-html="renderedContent" />
</template>
```

**Cas d'usage idéaux pour Server Components :**
- Rendu Markdown avec syntax highlighting
- Contenu statique (footer, header, navigation)
- Calculs lourds côté serveur
- Contenu SEO-critique sans interactivité

### ClientOnly pour widgets tiers

```vue
<template>
  <!-- Widget de partage social -->
  <ClientOnly>
    <ShareButtons :url="post.url" :title="post.title" />
    <template #fallback>
      <div class="share-skeleton">Partager cet article</div>
    </template>
  </ClientOnly>

  <!-- Commentaires Disqus -->
  <ClientOnly fallback-tag="section">
    <DisqusComments :identifier="post.id" />
    <template #fallback>
      <p>Chargement des commentaires...</p>
    </template>
  </ClientOnly>

  <!-- Embed Twitter/X -->
  <ClientOnly>
    <TwitterEmbed :tweet-id="tweetId" />
    <template #fallback>
      <a :href="tweetUrl">Voir le tweet</a>
    </template>
  </ClientOnly>
</template>
```

### Optimisation Core Web Vitals pour blog

**Seuils 2025 à respecter :**

| Métrique | Bon | Acceptable | Mauvais |
|----------|-----|------------|---------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5-4.0s | > 4.0s |
| **INP** (Interaction to Next Paint) | ≤ 200ms | 200-500ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1-0.25 | > 0.25 |

**Page article optimisée :**

```vue
<!-- pages/blog/[slug].vue -->
<template>
  <article>
    <!-- Contenu critique - rendu immédiat -->
    <h1>{{ post.title }}</h1>
    
    <!-- Image hero avec priorité LCP -->
    <NuxtImg 
      :src="post.heroImage" 
      preload 
      loading="eager"
      fetchpriority="high"
      format="webp"
      width="1200"
      height="630"
      :alt="post.title"
    />
    
    <!-- Contenu principal - Server Component si statique -->
    <BlogContent :content="post.body" />
    
    <!-- Sous le fold - hydratation visible -->
    <LazyAuthorBio 
      hydrate-on-visible 
      :author="post.author" 
    />
    
    <!-- Commentaires - hydratation interaction -->
    <LazyCommentsSection 
      hydrate-on-interaction
      :post-id="post.id"
    />
    
    <!-- Widgets sociaux - client only -->
    <ClientOnly>
      <ShareButtons :url="post.url" />
      <template #fallback>
        <div class="h-10" /> <!-- Réserve espace CLS -->
      </template>
    </ClientOnly>
    
    <!-- Articles liés - visible avec marge -->
    <LazyRelatedPosts 
      :hydrate-on-visible="{ rootMargin: '200px' }"
      :category="post.category"
    />
  </article>
</template>
```

## Routing et middleware Nuxt 4

### Structure de fichiers pour blog

```
app/
├── pages/
│   ├── index.vue                    # → /
│   ├── blog/
│   │   ├── index.vue               # → /blog
│   │   ├── [slug].vue              # → /blog/:slug
│   │   └── page/
│   │       └── [page].vue          # → /blog/page/:page
│   ├── categories/
│   │   ├── index.vue               # → /categories
│   │   └── [slug]/
│   │       └── index.vue           # → /categories/:slug
│   ├── tags/
│   │   └── [tag].vue               # → /tags/:tag
│   └── [...slug].vue               # → catch-all 404
├── middleware/
│   ├── 01.analytics.global.ts      # Global - tracking
│   ├── 02.redirects.global.ts      # Global - redirections
│   └── draft-protection.ts         # Named - protection drafts
└── plugins/
    ├── formatting.ts               # Universal
    └── analytics.client.ts         # Client only
```

### Validation de routes avec definePageMeta

```vue
<!-- pages/blog/[slug].vue -->
<script setup lang="ts">
definePageMeta({
  validate(route) {
    const slug = route.params.slug as string
    // Valide format slug (lettres, chiffres, tirets)
    if (!/^[a-z0-9-]+$/.test(slug)) {
      return {
        statusCode: 400,
        statusMessage: 'Format de slug invalide'
      }
    }
    return true
  },
  // Middleware appliqué à cette page
  middleware: ['draft-protection']
})

const route = useRoute()
const { data: post } = await useFetch(`/api/posts/${route.params.slug}`)
</script>
```

### Middleware patterns essentiels

**Règle critique : NE JAMAIS utiliser `next()` (pattern Vue Router 3)**

```typescript
// app/middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
  const user = useState('user')
  
  // ✅ Correct : navigateTo pour redirection
  if (!user.value?.isAuthenticated) {
    return navigateTo('/login')
  }
  
  // ✅ Correct : navigateTo avec code redirect
  if (to.path === '/ancien-chemin') {
    return navigateTo('/nouveau-chemin', { redirectCode: 301 })
  }
  
  // ✅ Correct : abortNavigation pour bloquer
  if (user.value?.isBanned) {
    return abortNavigation()
  }
  
  // ✅ Correct : erreur personnalisée
  if (!hasPermission(to)) {
    return abortNavigation(
      createError({
        statusCode: 403,
        statusMessage: 'Accès interdit'
      })
    )
  }
  
  // ✅ Correct : return void pour continuer
  return
})
```

**Middleware global de redirections :**

```typescript
// app/middleware/02.redirects.global.ts
const redirects: Record<string, string> = {
  '/articles': '/blog',
  '/posts': '/blog',
  '/tag': '/tags',
}

export default defineNuxtRouteMiddleware((to) => {
  const redirect = redirects[to.path]
  if (redirect) {
    return navigateTo(redirect, { redirectCode: 301 })
  }
})
```

**Middleware protection drafts :**

```typescript
// app/middleware/draft-protection.ts
export default defineNuxtRouteMiddleware(async (to) => {
  if (!to.params.slug) return
  
  const { data: post } = await useFetch(
    `/api/posts/${to.params.slug}`,
    { key: `post-${to.params.slug}` }
  )
  
  if (post.value?.status === 'draft') {
    const user = useState('user')
    if (!user.value?.isAdmin) {
      return abortNavigation(
        createError({ statusCode: 404, statusMessage: 'Article non trouvé' })
      )
    }
  }
})
```

## Plugins et modules

### Plugin de formatage blog

```typescript
// app/plugins/formatting.ts
export default defineNuxtPlugin(() => {
  return {
    provide: {
      formatDate: (date: string | Date, locale = 'fr-FR') => {
        return new Intl.DateTimeFormat(locale, {
          year: 'numeric',
          month: 'long',
          day: 'numeric'
        }).format(new Date(date))
      },
      
      readingTime: (content: string, wordsPerMinute = 200) => {
        const text = content.replace(/<[^>]*>/g, '')
        const words = text.trim().split(/\s+/).length
        return Math.ceil(words / wordsPerMinute)
      },
      
      excerpt: (content: string, length = 150) => {
        const text = content.replace(/<[^>]*>/g, '')
        return text.length > length 
          ? text.substring(0, length) + '...' 
          : text
      },
      
      slugify: (text: string) => {
        return text
          .toLowerCase()
          .normalize('NFD')
          .replace(/[\u0300-\u036f]/g, '')
          .replace(/[^a-z0-9]+/g, '-')
          .replace(/(^-|-$)/g, '')
      }
    }
  }
})
```

**Utilisation dans les templates :**

```vue
<template>
  <article>
    <time>{{ $formatDate(post.publishedAt) }}</time>
    <span>{{ $readingTime(post.content) }} min de lecture</span>
    <p>{{ $excerpt(post.content, 200) }}</p>
  </article>
</template>
```

### Plugin analytics client-only

```typescript
// app/plugins/analytics.client.ts
export default defineNuxtPlugin({
  name: 'analytics',
  parallel: true, // Ne bloque pas les autres plugins
  
  setup(nuxtApp) {
    const config = useRuntimeConfig()
    const router = useRouter()
    
    // Initialisation Google Analytics
    if (config.public.googleAnalyticsId) {
      // ... init GA
    }
    
    // Track navigation
    router.afterEach((to) => {
      if (typeof gtag !== 'undefined') {
        gtag('event', 'page_view', {
          page_path: to.fullPath,
          page_title: document.title
        })
      }
    })
  }
})
```

### Configuration runtime sécurisée

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    // Privé (serveur uniquement) - NUXT_API_SECRET
    apiSecret: '',
    contentApiKey: '',
    
    // Public (exposé au client) - NUXT_PUBLIC_*
    public: {
      siteUrl: 'https://monblog.fr',
      siteName: 'Mon Blog Nuxt',
      googleAnalyticsId: '',
      postsPerPage: 10
    }
  }
})
```

```env
# .env
NUXT_API_SECRET=secret-api-key
NUXT_CONTENT_API_KEY=content-key
NUXT_PUBLIC_SITE_URL=https://monblog.fr
NUXT_PUBLIC_GOOGLE_ANALYTICS_ID=G-XXXXXXXXXX
```

## Bonnes pratiques Claude Code

Pour optimiser le développement avec Claude Code, structurez votre projet avec des conventions claires et documentées.

### Structure de projet recommandée

```
project-root/
├── .claude/                     # Config Claude Code
│   └── settings.json
├── app/
│   ├── components/
│   │   ├── blog/               # Composants blog groupés
│   │   │   ├── Card.vue
│   │   │   ├── Content.server.vue
│   │   │   └── Hero.vue
│   │   └── ui/                 # Composants UI génériques
│   ├── composables/
│   │   ├── useBlogPost.ts
│   │   └── useBlogSearch.ts
│   ├── pages/
│   ├── middleware/
│   └── plugins/
├── content/                     # Nuxt Content
│   └── blog/
├── server/
│   └── api/
├── types/                       # Types TypeScript partagés
│   └── blog.ts
├── nuxt.config.ts
└── CONVENTIONS.md               # Documentation conventions
```

### Fichier CONVENTIONS.md pour Claude Code

```markdown
# Conventions Projet Blog Nuxt 4

## Composants Vue
- Toujours `<script setup lang="ts">`
- Props typées avec `defineProps<T>()`
- Emits typés avec `defineEmits<T>()`
- Préfixe Lazy pour composants lazy-loaded

## Composables
- Préfixe `use` obligatoire
- Retourner objet avec refs, jamais reactive
- Utiliser `MaybeRefOrGetter<T>` pour inputs flexibles
- `shallowRef` pour objets/arrays complexes

## Middleware
- `navigateTo()` pour redirections
- `abortNavigation()` pour blocage
- JAMAIS `next()`

## Performance
- `hydrate-on-visible` pour composants sous le fold
- `hydrate-on-interaction` pour widgets interactifs
- `.server.vue` pour contenu statique lourd
- `<ClientOnly>` pour widgets tiers

## Fichiers
- `.client.ts` / `.server.ts` pour contexte spécifique
- `.global.ts` pour middleware global
- Nommage numérique pour ordre (01.setup.global.ts)
```

### Commentaires patterns réutilisables

```typescript
// app/composables/useBlogPosts.ts

/**
 * Composable de gestion des articles de blog
 * 
 * @example
 * ```ts
 * const { posts, isLoading, fetchPosts } = useBlogPosts({
 *   perPage: 10,
 *   category: 'tech'
 * })
 * ```
 * 
 * @param options - Options de configuration
 * @param options.perPage - Nombre d'articles par page (défaut: 10)
 * @param options.category - Filtrer par catégorie
 * @returns Objet avec posts, état de chargement et méthodes
 */
export function useBlogPosts(options: UseBlogPostsOptions = {}) {
  // ... implementation
}
```

### Types partagés centralisés

```typescript
// types/blog.ts

export interface BlogPost {
  id: string
  slug: string
  title: string
  content: string
  excerpt: string
  coverImage: string
  author: Author
  category: Category
  tags: Tag[]
  publishedAt: string
  updatedAt: string
  status: 'draft' | 'published' | 'archived'
  readingTime: number
}

export interface Author {
  id: string
  name: string
  avatar: string
  bio: string
}

export interface Category {
  id: string
  name: string
  slug: string
  description: string
}

export interface Tag {
  id: string
  name: string
  slug: string
}

export interface PaginatedResponse<T> {
  data: T[]
  meta: {
    currentPage: number
    totalPages: number
    totalItems: number
    perPage: number
  }
}
```

## Commandes de build et déploiement

```bash
# Développement local
npx nuxi dev

# Génération statique pure (SSG)
npx nuxi generate
# Output: .output/public/

# Build hybride avec prerendering
NITRO_PRESET=cloudflare_pages npx nuxi build

# Analyse du bundle
npx nuxi analyze

# Preview local de la build
npx nuxi preview
```

**Configuration Cloudflare Pages :**

| Paramètre | Valeur |
|-----------|--------|
| Build command | `npx nuxi generate` |
| Output directory | `.output/public` |
| Framework preset | Nuxt (auto-détecté) |

## Récapitulatif des breaking changes Nuxt 4

| Aspect | Nuxt 3 | Nuxt 4 |
|--------|--------|--------|
| Structure | `pages/`, `components/` | `app/pages/`, `app/components/` |
| Generate config | `generate: {}` | `nitro: { prerender: {} }` |
| Output | `dist/` | `.output/public/` |
| Exclude routes | `generate.exclude` | `nitro.prerender.ignore` |
| TypeScript | Single tsconfig | Project references |
| Compatibility | N/A | `compatibilityDate` requis |

Cette configuration complète fournit une base solide pour développer un blog performant avec Nuxt 4.2.x, optimisé pour Cloudflare Pages et prêt pour un développement efficace avec Claude Code.