# SEO Patterns

## Title Tags

### titleTemplate dans app.vue

Le `titleTemplate` doit être défini dans `app.vue` pour supporter les fonctions dynamiques (impossible dans `nuxt.config.ts`).

```typescript
<!-- app/app.vue -->
<script setup lang="ts">
// Title template global avec fonction (contrôle total)
useHead({
  titleTemplate: (titleChunk) => {
    return titleChunk ? `${titleChunk} | sebc.dev` : 'sebc.dev'
  }
})

// Meta statiques server-only (performance SSG)
if (import.meta.server) {
  useSeoMeta({
    robots: 'index, follow',
    ogSiteName: 'sebc.dev',
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

### Longueur titre recommandée

| Aspect | Valeur | Raison |
|--------|--------|--------|
| **Caractères** | 50-60 max | Évite troncature SERPs |
| **Pixels** | ~600px | Limite affichage Google |
| **Format** | `{titre} \| {site}` | Pattern standard efficace |

## useSeoMeta() Patterns

### Pattern page statique

```typescript
<!-- app/pages/about.vue -->
<script setup lang="ts">
// ✅ Pattern recommandé Nuxt 4
useSeoMeta({
  title: 'À propos',                              // 50-60 caractères max
  description: 'Découvrez mon parcours et mes projets en développement web.',
  // og:title NE hérite PAS du titleTemplate - toujours le définir explicitement
  ogTitle: 'À propos | sebc.dev',
  ogDescription: 'Découvrez mon parcours et mes projets en développement web.',
  ogImage: 'https://sebc.dev/og/about.jpg',
  ogType: 'website'
})
</script>
```

### Pattern page dynamique (articles)

```typescript
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
import type { Collections } from '@nuxt/content'

const route = useRoute()
const { locale } = useI18n()
const siteConfig = useSiteConfig()

const collection = `articles_${locale.value}` as keyof Collections

const { data: post } = await useAsyncData(
  `blog-${route.path}`,
  () => queryCollection(collection).path(route.path).first()
)

if (!post.value) {
  throw createError({ statusCode: 404, statusMessage: 'Article non trouvé' })
}

// Computed avec fallbacks
const title = computed(() => post.value?.title ?? 'Article')
const description = computed(() => {
  const desc = post.value?.description ?? ''
  return desc.length > 155 ? `${desc.slice(0, 152)}...` : desc
})
const image = computed(() => {
  const img = post.value?.image
  if (!img) return `${siteConfig.url}/og-default.jpg`
  return img.startsWith('http') ? img : `${siteConfig.url}${img}`
})
const canonical = computed(() => `${siteConfig.url}${route.path}`)
const publishedDate = computed(() =>
  post.value?.publishedAt ? new Date(post.value.publishedAt).toISOString() : undefined
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
  robots: () => post.value?.draft ? 'noindex, nofollow' : 'index, follow'
})
</script>
```

### Anti-patterns à éviter

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
  title: 'Guide SEO Nuxt 4',              // 17 caractères
  ogTitle: 'Guide SEO Nuxt 4 | sebc.dev'  // Version complète pour social
})
```

## definePageMeta() vs useSeoMeta()

`definePageMeta()` est une **macro build-time** qui ne peut pas contenir de valeurs dynamiques.

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

## Composable useContentSeo()

Composable réutilisable pour les pages de contenu avec fallbacks automatiques.

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
  const siteConfig = useSiteConfig()
  const route = useRoute()

  const {
    defaultDescription = 'Blog technique sur le développement web',
    defaultImage = '/og-default.jpg',
    truncateAt = 155
  } = options

  const siteUrl = siteConfig.url

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

**Usage :**

```typescript
<script setup lang="ts">
const { data: post } = await useAsyncData(...)
const { description, image, canonical } = useContentSeo(post)

useSeoMeta({
  description,
  ogImage: image,
  ogUrl: canonical
})
</script>
```

## Canonical URLs

### Configuration SSG avec Cloudflare

Chaque page doit avoir une **URL canonique absolue unique**. Cloudflare redirige par défaut sans trailing slash.

```typescript
<script setup lang="ts">
const siteConfig = useSiteConfig()
const route = useRoute()

// URL canonique absolue (sans trailing slash pour cohérence Cloudflare)
const canonicalUrl = computed(() => {
  const path = route.path.toLowerCase()
  // Normaliser sans trailing slash (comportement par défaut Cloudflare)
  const normalizedPath = path.endsWith('/') && path !== '/'
    ? path.slice(0, -1)
    : path
  return `${siteConfig.url}${normalizedPath}`
})

useHead({
  link: [{ rel: 'canonical', href: canonicalUrl }]
})

useSeoMeta({
  ogUrl: canonicalUrl
})
</script>
```

### Erreurs canoniques courantes

```typescript
// ❌ MAUVAIS : URL relative
useHead({
  link: [{ rel: 'canonical', href: '/ma-page' }]
})

// ❌ MAUVAIS : Canonical vers une page noindex
useSeoMeta({ robots: 'noindex' })
useHead({
  link: [{ rel: 'canonical', href: 'https://sebc.dev/page/' }]
})
// Signaux contradictoires !

// ✅ BON : URL absolue
useHead({
  link: [{ rel: 'canonical', href: 'https://sebc.dev/page' }]
})
```

### Canonical cross-language (CRITIQUE)

**Règle absolue** : Chaque version linguistique DOIT avoir un canonical **auto-référencé**. Ne JAMAIS pointer le canonical vers une autre langue — cela **invalide tous les signaux hreflang**.

```html
<!-- Sur /fr/page - CORRECT -->
<link rel="canonical" href="https://example.com/fr/page" />
<link rel="alternate" hreflang="fr-FR" href="https://example.com/fr/page" />
<link rel="alternate" hreflang="en-US" href="https://example.com/page" />

<!-- Sur /fr/page - INCORRECT (invalide tout le hreflang) -->
<link rel="canonical" href="https://example.com/page" />  <!-- ❌ Pointe vers EN -->
```

| Erreur | Conséquence |
|--------|-------------|
| Canonical FR → EN | Google ignore tous les hreflang |
| Canonical EN → FR | Google ignore tous les hreflang |
| Canonical auto-référencé | ✅ hreflang fonctionne correctement |

**Note importante** : Le contenu traduit n'est **PAS** considéré comme dupliqué par Google. Seul le contenu identique non traduit pose problème.

## Robots Directives

### Configuration via routeRules

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    // Pages à ne pas indexer
    '/admin/**': { robots: 'noindex, nofollow' },
    '/preview/**': { robots: 'noindex' },
    '/merci': { robots: 'noindex' },           // Page confirmation
    '/recherche': { robots: 'noindex' },       // Recherche interne

    // Pages paginées
    '/blog/page/**': { robots: 'noindex, follow' },
  }
})
```

### Directive dynamique par page

```typescript
<script setup lang="ts">
const isDraft = computed(() => post.value?.draft === true)

useSeoMeta({
  robots: () => isDraft.value ? 'noindex, nofollow' : 'index, follow'
})
</script>
```

### robots.txt vs meta robots

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

Sitemap: https://sebc.dev/sitemap.xml
```

### Crawlers AI dans robots.txt

Les principaux crawlers AI respectent robots.txt. Configuration pour contrôler l'indexation vs l'entraînement :

```
# public/robots.txt - Ajouts pour crawlers AI

# Autoriser les crawlers de recherche AI (citations dans réponses)
User-agent: OAI-SearchBot
User-agent: ChatGPT-User
User-agent: ClaudeBot
User-agent: PerplexityBot
Allow: /

# Optionnel: bloquer l'entraînement des modèles
User-agent: GPTBot
User-agent: anthropic-ai
User-agent: Google-Extended
Disallow: /
```

**User-agents AI à connaître :**

| User-agent | Propriétaire | Fonction |
|------------|--------------|----------|
| `GPTBot` | OpenAI | Entraînement modèles GPT |
| `OAI-SearchBot` | OpenAI | ChatGPT Search (citations) |
| `ChatGPT-User` | OpenAI | Plugins/browsing ChatGPT |
| `ClaudeBot` | Anthropic | Citations Claude |
| `anthropic-ai` | Anthropic | Entraînement Claude |
| `PerplexityBot` | Perplexity | Index Perplexity AI |
| `Google-Extended` | Google | Contrôle Gemini/Bard |

**Note importante** : Bloquer `GPTBot` empêche l'entraînement mais n'affecte pas les citations dans ChatGPT Search (géré par `OAI-SearchBot`).

## Schema.org avec nuxt-schema-org

### Schema Article pour blog posts

```typescript
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
import { defineArticle, useSchemaOrg } from '#imports'

const siteConfig = useSiteConfig()

// ... fetch post data ...

useSchemaOrg([
  defineArticle({
    '@type': 'BlogPosting',
    headline: () => post.value?.title,
    description: () => post.value?.description,
    image: () => absoluteImageUrl.value,
    datePublished: () => post.value?.publishedAt
      ? new Date(post.value.publishedAt).toISOString()
      : undefined,
    dateModified: () => post.value?.updatedAt
      ? new Date(post.value.updatedAt).toISOString()
      : undefined,
    author: {
      '@type': 'Person',
      name: 'Sébastien C.',
      url: siteConfig.url
    },
    publisher: {
      '@type': 'Organization',
      name: 'sebc.dev',
      logo: `${siteConfig.url}/logo.png`
    },
    keywords: () => post.value?.tags,
    articleSection: () => post.value?.pillar,
    inLanguage: () => locale.value === 'fr' ? 'fr' : 'en'
  })
])
</script>
```

### TechArticle pour contenu technique

La distinction entre `Article` et `TechArticle` est sémantique. Utilisez **TechArticle** pour :
- Tutoriels et guides how-to
- Documentation technique
- Troubleshooting et debugging

TechArticle hérite toutes les propriétés d'Article avec deux ajouts spécifiques :

| Propriété | Valeurs | Usage |
|-----------|---------|-------|
| `proficiencyLevel` | `'Beginner'` \| `'Expert'` | Niveau requis du lecteur |
| `dependencies` | `string[]` | Prérequis techniques |

```typescript
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
useSchemaOrg([
  defineArticle({
    '@type': 'TechArticle',  // Résulte en ['Article', 'TechArticle']

    headline: () => post.value?.title,
    description: () => post.value?.description,
    datePublished: () => post.value?.publishedAt
      ? new Date(post.value.publishedAt).toISOString()
      : undefined,
    dateModified: () => post.value?.updatedAt
      ? new Date(post.value.updatedAt).toISOString()
      : undefined,

    // Propriétés TechArticle spécifiques
    proficiencyLevel: () => {
      const level = post.value?.level
      return level === 'beginner' ? 'Beginner' : 'Expert'
    },

    // Métadonnées additionnelles
    articleSection: () => post.value?.pillar,
    keywords: () => post.value?.tags,
    inLanguage: () => locale.value,
    wordCount: () => post.value?.body?.split(/\s+/).length,
  })
])
</script>
```

**Note format de date** : Utilisez toujours le format **ISO 8601 avec timezone** (ex: `2025-12-28T10:00:00+01:00`). Sans timezone, Google utilise celle de Googlebot.

### Layout avec WebSite schema

```typescript
<!-- app/layouts/default.vue -->
<script setup lang="ts">
import { defineWebSite, defineWebPage, useSchemaOrg } from '#imports'

const siteConfig = useSiteConfig()
const route = useRoute()

useSchemaOrg([
  defineWebSite({
    name: 'sebc.dev',
    description: 'Blog technique sur le développement web',
    url: siteConfig.url,
    inLanguage: ['fr', 'en']
  }),
  defineWebPage({
    '@id': `${siteConfig.url}${route.path}`,
    url: `${siteConfig.url}${route.path}`
  })
])
</script>
```

## Breaking Changes Nuxt 3 → Nuxt 4 (SEO)

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

### Checklist migration SEO

| Ancien (Nuxt 3) | Nouveau (Nuxt 4) |
|-----------------|------------------|
| `useServerSeoMeta()` | `if (import.meta.server) { useSeoMeta() }` |
| `generate.routes` | `nitro.prerender.routes` |
| `generate.exclude` | `nitro.prerender.ignore` |

## Intégration Frontmatter avec asSeoCollection()

Le wrapper `asSeoCollection()` active les clés frontmatter SEO dans les fichiers Markdown :

```yaml
---
title: "Mon article"
description: "Description SEO (max 160 caractères)"
image: "/images/cover.jpg"

# Overrides OG optionnels (si différents du titre/description)
ogTitle: "Titre optimisé pour les réseaux sociaux"
ogDescription: "Description plus courte pour OG"

# Schema.org structuré (optionnel)
schemaOrg:
  - "@type": "BlogPosting"
    headline: "Mon article"
    author:
      "@type": "Person"
      name: "Sébastien C."
    datePublished: "2025-01-15"

# Contrôle sitemap (optionnel)
sitemap:
  lastmod: 2025-01-16
  priority: 0.8

# Contrôle robots (optionnel)
robots:
  noindex: false
  nofollow: false
---
```

## Open Graph Spécifications

### Dimensions et formats recommandés

| Propriété | Valeur | Notes |
|-----------|--------|-------|
| **og:image** | **1200×630px** | Ratio 1.91:1, format **PNG** (pas WebP - LinkedIn incompatible) |
| **og:title** | ≤65 caractères | Troncation après ~70 caractères |
| **og:description** | 150-200 caractères | Maximum visible selon plateforme |
| **og:type** | `article` | Active les propriétés `article:*` |
| **article:published_time** | ISO 8601 | `2025-12-29T09:00:00+01:00` (timezone obligatoire) |

### Pattern complet OG pour articles

```typescript
useSeoMeta({
  // Titres
  title: () => article.value?.title,
  ogTitle: () => article.value?.ogTitle || article.value?.title,

  // Descriptions
  description: () => article.value?.description,
  ogDescription: () => article.value?.ogDescription || article.value?.description,

  // Image (URL absolue obligatoire)
  ogImage: () => article.value?.image || '/default-og.png',

  // Type et métadonnées article
  ogType: 'article',
  articlePublishedTime: () => article.value?.date,
  articleModifiedTime: () => article.value?.updatedAt,
  articleAuthor: () => article.value?.author,
  articleSection: () => article.value?.category,
})
```

## Twitter Cards

### Comportement des fallbacks

Twitter utilise l'attribut `name` (pas `property`) et **requiert obligatoirement `twitter:card`**. Cependant, les autres propriétés tombent sur leurs équivalents OG :

| Twitter | Fallback OG | Obligatoire |
|---------|------------|-------------|
| `twitter:card` | ❌ Aucun | **OUI** |
| `twitter:title` | `og:title` | Non |
| `twitter:description` | `og:description` | Non |
| `twitter:image` | `og:image` | Non |

```typescript
useSeoMeta({
  // Ces props OG servent de fallback pour Twitter
  ogTitle: 'Mon Article',
  ogDescription: 'Description de mon article',
  ogImage: 'https://sebc.dev/og.png',

  // twitter:card est OBLIGATOIRE (pas de fallback)
  twitterCard: 'summary_large_image',

  // Optionnels mais recommandés
  twitterSite: '@sebcdev',     // Compte du site
  twitterCreator: '@sebcdev', // Compte de l'auteur
})
```

### `summary_large_image` vs `summary`

| Type | Dimensions image | Recommandé pour |
|------|-----------------|-----------------|
| `summary_large_image` | 1200×628px min | **Blog** (image proéminente) |
| `summary` | 144×144px | Profils, pages sans visuel |

**Recommandation :** Toujours utiliser `summary_large_image` pour un blog.

## Génération d'images OG avec nuxt-og-image

### Configuration Zero Runtime (SSG)

Le mode `zeroRuntime: true` génère les images au build time via Satori, sans server functions. **100% compatible Cloudflare Pages gratuit**.

```typescript
// nuxt.config.ts
ogImage: {
  zeroRuntime: true,           // ESSENTIEL pour SSG pur
  runtimeCacheStorage: false,  // Pas de cache runtime en SSG
  defaults: {
    renderer: 'satori',        // Vue → SVG → PNG au build
    width: 1200,
    height: 630,               // Ratio 1.91:1 standard
  }
}
```

### Composant template OG Image

```vue
<!-- app/components/OgImage/BlogPost.vue -->
<script setup lang="ts">
withDefaults(defineProps<{
  title?: string
  description?: string
  siteName?: string
}>(), {
  title: 'Article',
  siteName: 'sebc.dev',
})
</script>

<template>
  <div class="h-full w-full flex flex-col justify-between p-16 bg-slate-800">
    <div class="flex flex-col gap-4">
      <h1 class="text-white text-6xl font-bold leading-tight">{{ title }}</h1>
      <p v-if="description" class="text-gray-200 text-2xl">{{ description }}</p>
    </div>
    <span class="text-white text-xl font-semibold">{{ siteName }}</span>
  </div>
</template>
```

### Utilisation avec defineOgImageComponent()

```typescript
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
// ... fetch article data ...

// Associer le template OG à cette page
defineOgImageComponent('BlogPost', {
  title: article.value?.title,
  description: article.value?.description,
})
</script>
```

### Performance build

| Métrique | Valeur |
|----------|--------|
| Temps par image | ~50-100ms |
| 100 articles | ~10 secondes |
| Output | `.output/public/__og-image__/` |
| Cache CDN | Illimité (assets statiques) |

## Multilingue (og:locale)

### Format og:locale vs BCP 47

Le format `og:locale` utilise un **underscore** (`fr_FR`) alors que BCP 47 utilise un tiret (`fr-FR`). **Nuxt SEO Utils convertit automatiquement**.

```typescript
// i18n.config.ts - Utiliser BCP 47
locales: [
  { code: 'fr', language: 'fr-FR', name: 'Français' },
  { code: 'en', language: 'en-US', name: 'English' },
]
```

### Rendu automatique avec useLocaleHead()

```html
<!-- Généré automatiquement -->
<link rel="alternate" hreflang="fr-FR" href="https://sebc.dev/blog/article">
<link rel="alternate" hreflang="en-US" href="https://sebc.dev/en/blog/article">
<link rel="alternate" hreflang="x-default" href="https://sebc.dev/blog/article">
<meta property="og:locale" content="fr_FR">
<meta property="og:locale:alternate" content="en_US">
```

## hreflang x-default et Comportement Crawlers

### Qu'est-ce que x-default ?

Le `x-default` désigne l'URL de **fallback** pour les utilisateurs dont la langue n'est pas supportée. Avec `strategy: 'prefix_except_default'`, l'URL sans préfixe (`/` = anglais) devient naturellement le candidat x-default.

**Important** : @nuxtjs/i18n **ne génère PAS** x-default via `useLocaleHead()`. Cependant, @nuxtjs/sitemap l'ajoute automatiquement dans le `sitemap.xml`.

### Comportement par moteur de recherche

| Moteur | Support hreflang | Notes |
|--------|------------------|-------|
| **Google** | ✅ Complet | HTML, HTTP headers, ou sitemap XML. Exige liens bidirectionnels |
| **Bing** | ❌ Non officiel | Utilise ses propres signaux (Bing Webmaster Tools) |
| **Yandex** | ✅ Partiel | Supporte hreflang basique |

### Sitemap i18n avec @nuxtjs/sitemap

L'intégration sitemap-i18n fonctionne **automatiquement** quand les deux modules sont installés. Le sitemap génère les balises `<xhtml:link rel="alternate" hreflang="">` incluant x-default :

```xml
<url>
  <loc>https://sebc.dev/</loc>
  <xhtml:link rel="alternate" href="https://sebc.dev/" hreflang="x-default" />
  <xhtml:link rel="alternate" href="https://sebc.dev/" hreflang="en-US" />
  <xhtml:link rel="alternate" href="https://sebc.dev/fr" hreflang="fr-FR" />
</url>
```

### _i18nTransform pour URLs dynamiques

Pour les URLs provenant d'une API ou d'un CMS, utilisez `_i18nTransform: true` pour générer automatiquement toutes les variantes localisées :

```typescript
// server/api/__sitemap__/urls.ts
export default defineSitemapEventHandler(() => [
  // Génère /about, /fr/a-propos avec hreflang complet
  { loc: '/about', _i18nTransform: true },

  // Pour les articles de blog
  { loc: '/blog/my-article', _i18nTransform: true },
])
```

**Désactiver l'intégration auto (rare) :**

```typescript
// nuxt.config.ts
sitemap: {
  autoI18n: false  // Désactive l'intégration automatique
}
```

## Outils de Validation

Après chaque mise à jour de meta tags, **purger le cache des crawlers** :

| Plateforme | Outil | URL |
|------------|-------|-----|
| **Facebook** | Sharing Debugger | `developers.facebook.com/tools/debug/sharing/` |
| **Twitter** | Card Validator | `typefully.com/tools/twitter-card-validator` |
| **LinkedIn** | Post Inspector | `linkedin.com/post-inspector/` |

**Erreurs courantes :**
- Image inaccessible (vérifier robots.txt)
- Dimensions incorrectes (utiliser exactement 1200×630)
- Format WebP (LinkedIn ne supporte pas - utiliser **PNG**)
- URL non HTTPS

## BreadcrumbList Schema

### Composable useBreadcrumbSchema()

Génération automatique du breadcrumb basée sur la route actuelle.

```typescript
// app/composables/useBreadcrumbSchema.ts
export function useBreadcrumbSchema() {
  const route = useRoute()
  const { t } = useI18n()

  const segments = route.path.split('/').filter(Boolean)
  const items = [
    { name: t('nav.home'), item: '/' }
  ]

  let currentPath = ''
  segments.forEach((segment, index) => {
    currentPath += `/${segment}`
    const isLast = index === segments.length - 1

    items.push({
      name: formatSegmentName(segment),
      // Pas de 'item' pour le dernier élément (convention Google)
      ...(isLast ? {} : { item: currentPath })
    })
  })

  useSchemaOrg([
    defineBreadcrumb({
      itemListElement: items
    })
  ])
}

function formatSegmentName(segment: string): string {
  // Convertit "mon-article" en "Mon article"
  return segment
    .replace(/-/g, ' ')
    .replace(/\b\w/g, (c) => c.toUpperCase())
}
```

### Usage dans une page article

```typescript
<script setup lang="ts">
// Breadcrumb manuel pour contrôle total
useSchemaOrg([
  defineBreadcrumb({
    itemListElement: [
      { name: 'Accueil', item: '/' },
      { name: 'Blog', item: '/blog' },
      { name: article.value?.title }  // Pas d'item = page courante
    ]
  })
])
</script>
```

**Exigences Google** : Au minimum **deux ListItems** pour l'éligibilité aux rich results. La propriété `position` est auto-calculée par le module.

## Changements Google 2024-2025

Récapitulatif des évolutions impactant les rich results Schema.org :

| Fonctionnalité | Statut | Date | Impact |
|----------------|--------|------|--------|
| **Sitelinks Search Box** | Supprimé | Nov 2024 | `SearchAction` n'affecte plus les SERPs |
| **FAQ Rich Results** | Limité | Août 2023 | Réservé aux sites gov/santé uniquement |
| **How-To Rich Results** | Supprimé | 2023 | Plus de rich snippets pour tutoriels |
| **Article author.url** | Ajouté | 2024 | Améliore la désambiguïsation des auteurs |

**Implications pour le projet :**
- Ne pas implémenter `SearchAction` (inutile)
- Ne pas implémenter `FAQPage` (blogs techniques non éligibles)
- Se concentrer sur `Article`, `BreadcrumbList`, `Person`, `WebSite`

## Validation et Debugging Schema.org

### Outils de validation

| Outil | URL | Usage |
|-------|-----|-------|
| **Google Rich Results Test** | `search.google.com/test/rich-results` | Validation des rich snippets Google |
| **Schema Markup Validator** | `validator.schema.org` | Validation syntaxique Schema.org |
| **Nuxt DevTools** | Onglet Schema.org | Visualisation en développement |

### Endpoint de debug

En développement, accédez à `/__schema-org__/debug.json` pour visualiser le schéma JSON-LD généré.

```bash
# Exemple en développement local
curl http://localhost:3000/__schema-org__/debug.json | jq
```

### Validation conditionnelle en dev

```typescript
<script setup lang="ts">
// Afficher le schéma dans la console en développement
if (import.meta.dev) {
  const schemaData = useSchemaOrg()
  console.log('Schema.org:', JSON.stringify(schemaData, null, 2))
}
</script>
```

## GEO (Generative Engine Optimization)

### Ratio optimal de contenu

La densité factuelle détermine les chances de citation par les moteurs IA. Ratio recommandé :

| Métrique | Ratio optimal | Exemple (3000 mots) |
|----------|---------------|---------------------|
| Statistiques | 1 / 150-200 mots | 15-20 stats sourcées |
| Réponses directes | 6 / 1000 mots | 18 answer capsules |
| Citations externes | 5-8 / article | Sources .edu, .gov, académiques |

**Pattern quotable fact :**

```markdown
❌ Inefficace : "Notre solution améliore considérablement les performances."

✅ GEO-optimisé : "Notre solution réduit le temps de réponse de 4 minutes
à 45 secondes, augmentant l'efficacité de 82% (Source, 2025)."
```

### Answer Capsules

L'answer capsule est une réponse autonome placée après un H2, conçue pour extraction verbatim par les LLM.

| Contexte | Longueur | Liens |
|----------|----------|-------|
| Réponse principale (après H1) | 40-60 mots | **Aucun** |
| Answer capsule avec preuves | 80-120 mots | **Aucun** |
| Paragraphe de développement | 150-200 mots | Oui (sous la capsule) |

**Découverte critique** : **91%** des capsules citées par les IA ne contiennent aucun lien. Les liens suggèrent que la réponse autoritaire est ailleurs.

```markdown
## Comment optimiser les Core Web Vitals ?

Les Core Web Vitals s'optimisent en trois axes : LCP sous 2.5s via
lazy loading images et preload fonts, INP sous 200ms via code splitting
et hydratation différée, CLS à 0 via dimensions explicites sur les médias.

Paragraphe de développement avec [liens vers sources](url) et détails
techniques additionnels placés ici, sous la capsule.
```

### H2 formulés en questions

Les H2 en format question correspondent aux requêtes utilisateurs vers les assistants IA :

| À éviter | Préférer |
|----------|----------|
| "Conseils performance" | "Comment améliorer les performances ?" |
| "Démarrage" | "Quelles sont les étapes pour commencer ?" |
| "Nos solutions" | "Quelle solution choisir pour [cas d'usage] ?" |
| "Configuration" | "Comment configurer [fonctionnalité] ?" |

### Paramètres structurels Markdown

Pour Nuxt Content 3, respectez ces limites pour un parsing optimal par les LLM :

| Élément | Limite | Raison |
|---------|--------|--------|
| Paragraphes | Max 120 mots (idéal 40-60) | Extraction autonome |
| Phrases | 15-20 mots max | Parsing propre |
| H1 | 1 unique par page | Titre dans frontmatter |
| H2 | Questions alignées intention utilisateur | Correspondance requêtes IA |
| H3 | Détails de support, sous-réponses | Hiérarchie claire |

### TL;DR / Quick Answer (BLUF)

Le principe BLUF (Bottom Line Up Front) place la réponse clé en introduction :

```markdown
---
title: "Comment implémenter le dark mode dans Nuxt 4 ?"
---

## TL;DR

Le dark mode dans Nuxt 4 s'implémente via @nuxtjs/color-mode avec
la classe .dark de Tailwind. Configuration en 3 étapes : installer
le module, définir classSuffix vide, ajouter le script anti-FOUC
dans app.head. Temps d'implémentation : 15 minutes.

## Étape 1 : Installation du module

[Contenu détaillé...]
```

### Éléments bio auteur (E-E-A-T)

Les IA recherchent des preuves de crédentials avant de citer. Bio auteur requise :

1. **Nom complet** - Réel et vérifiable
2. **Crédentials** - Diplômes, certifications
3. **Titre et affiliation** - Poste actuel
4. **Expérience chiffrée** - Années, domaines
5. **Résultat concret** - Cas d'étude, accomplissement
6. **Liens profils** - LinkedIn, GitHub, site personnel

**Pattern page auteur :**

```markdown
<!-- content/auteurs/sebastien-c.md -->
---
name: "Sébastien C."
title: "Lead Developer"
image: "/images/authors/sebastien.jpg"
credentials: "10 ans d'expérience Vue.js/Nuxt"
social:
  github: "https://github.com/sebcdev"
  linkedin: "https://linkedin.com/in/sebcdev"
---

Développeur full-stack spécialisé Vue.js et Nuxt depuis 2015.
Contributeur open source avec 50+ PR merged sur l'écosystème Nuxt.
Auteur de 3 modules Nuxt totalisant 10K+ téléchargements mensuels.
```

### Différences entre plateformes IA

Chaque moteur génératif a des préférences distinctes :

| Critère | ChatGPT Search | Perplexity | Google AI Overviews |
|---------|---------------|------------|---------------------|
| **Longueur préférée** | 2500-3000 mots | Modérée, haute densité | Variable |
| **Poids fraîcheur** | Faible | **Très élevé** (90 jours) | Modéré |
| **Ton** | Encyclopédique | Conversationnel | Autoritaire |
| **Index source** | **Bing** (73% corrélation) | Temps réel | Index Google |
| **Signal fort** | Structure Wikipedia | Exemples communautaires | SEO traditionnel |

**Implications pratiques :**
- **Perplexity** : Contenu >9 mois = -35% citations. Mettre à jour `updatedAt` régulièrement
- **ChatGPT** : Optimiser pour Bing optimise aussi ChatGPT Search
- **Google AI** : 92% des citations viennent du top 10 SEO existant

### Métriques GEO vs SEO

| SEO traditionnel | Équivalent GEO |
|------------------|----------------|
| Position 1-10 | Inclusion dans la citation (cité ou non) |
| Taux de clic (CTR) | Fréquence et proéminence de citation |
| Trafic organique | Trafic référé IA (`utm_source=chatgpt.com`) |
| Profil de backlinks | Vélocité des mentions de marque |

**Tracking pratique :**
- Google Analytics : Trafic référent `perplexity.ai`, `chatgpt.com`
- ChatGPT ajoute automatiquement `utm_source=chatgpt.com`
- Logs serveur : Activité GPTBot, PerplexityBot, ClaudeBot

### Composant MDC ::faq-item

Pattern pour FAQ sémantique dans Nuxt Content 3 (note : FAQ rich results limités aux sites gov/santé, mais utile pour structure) :

```vue
<!-- components/content/FaqItem.vue -->
<script setup lang="ts">
defineProps<{
  question: string
}>()
</script>

<template>
  <details class="border-b border-gray-200 py-4">
    <summary class="cursor-pointer font-medium text-lg">
      {{ question }}
    </summary>
    <div class="mt-2 text-gray-600">
      <slot />
    </div>
  </details>
</template>
```

**Usage dans Markdown :**

```markdown
## FAQ

::faq-item{question="Comment installer nuxt-schema-org ?"}
Exécutez `npx nuxt module add schema-org` dans votre projet Nuxt.
Le module s'ajoute automatiquement à nuxt.config.ts et génère
les schemas JSON-LD au build time.
::

::faq-item{question="Le module supporte-t-il Nuxt 4 ?"}
Oui, nuxt-schema-org v5.0.10 est pleinement compatible avec
Nuxt 4 et compatibilityDate. Aucune configuration spéciale requise.
::
```
