# Page Routing Patterns

**Route catch-all pour blog avec Content 3 (collections par langue):**

```vue
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
import type { Collections } from '@nuxt/content'

const route = useRoute()
const { locale } = useI18n()

// Pour /blog/2024/mon-article → slug = ['2024', 'mon-article']
const pathArray = route.params.slug as string[]
const fullPath = computed(() => `/blog/${pathArray?.join('/')}`)

// Collection dynamique selon la locale
const collection = computed(() => `articles_${locale.value}` as keyof Collections)

// Récupération avec clé unique pour cache (inclut locale)
const { data: article } = await useAsyncData(
  `article-${locale.value}-${fullPath.value}`,
  () => queryCollection(collection.value).path(fullPath.value).first(),
  { watch: [locale] }  // Re-fetch au changement de langue
)

// ⚠️ CRITIQUE pour SSG : fatal: true génère une vraie page 404
if (!article.value) {
  throw createError({
    statusCode: 404,
    statusMessage: 'Article non trouvé',
    fatal: true  // ← OBLIGATOIRE pour SSG/prerendering
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

**Distinction `[...slug].vue` vs `[[...slug]].vue` :**

| Syntaxe | Match `/blog/` | Match `/blog/article` | Usage |
|---------|----------------|----------------------|-------|
| `[...slug].vue` | ❌ Non | ✅ Oui | Avec `index.vue` séparé |
| `[[...slug]].vue` | ✅ Oui | ✅ Oui | Route unique pour tout |

**Validation format-only dans definePageMeta :**

> Privilégiez la validation de **format uniquement** dans `validate`, et gérez l'existence dans le composant avec `createError({ fatal: true })`. Cela évite les requêtes doubles.

```typescript
definePageMeta({
  validate: async (route) => {
    const slug = route.params.slug
    const slugArray = Array.isArray(slug) ? slug : [slug]

    // Validation FORMAT uniquement (pas de requête DB)
    if (slugArray.some(s => !/^[a-z0-9-]+$/.test(s))) {
      return { statusCode: 400, statusMessage: 'Format de slug invalide' }
    }

    return true  // L'existence est vérifiée dans le composant
  }
})
```
