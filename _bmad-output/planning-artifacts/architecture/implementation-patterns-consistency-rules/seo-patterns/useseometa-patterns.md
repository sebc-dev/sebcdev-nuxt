# useSeoMeta() Patterns

## Pattern page statique

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

## Pattern page dynamique (articles)

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

## Anti-patterns à éviter

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
