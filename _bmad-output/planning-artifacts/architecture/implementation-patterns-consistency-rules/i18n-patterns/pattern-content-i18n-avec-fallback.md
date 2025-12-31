# Pattern Content + i18n avec Fallback

Page dynamique pour contenu multilingue avec fallback vers la langue par défaut :

```vue
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
import { withLeadingSlash, joinURL } from 'ufo'
import type { Collections } from '@nuxt/content'

const { locale, t } = useI18n()
const route = useRoute()

// Construire le chemin du contenu
const contentPath = computed(() =>
  withLeadingSlash(joinURL('blog', ...(
    Array.isArray(route.params.slug)
      ? route.params.slug
      : [route.params.slug]
  )))
)

// Query avec collection basée sur la locale
const { data: article } = await useAsyncData(
  `blog-${contentPath.value}-${locale.value}`,
  async () => {
    const collection = `articles_${locale.value}` as keyof Collections
    let content = await queryCollection(collection)
      .path(contentPath.value)
      .first()

    // Fallback vers anglais si contenu manquant
    if (!content && locale.value !== 'en') {
      content = await queryCollection('articles_en')
        .path(contentPath.value)
        .first()
      if (content) content._isFallback = true
    }

    return content
  },
  { watch: [locale] }  // ⚠️ OBLIGATOIRE pour re-fetch au changement de langue
)

// SEO avec hreflang automatique
const head = useLocaleHead({ addSeoAttributes: true })
useHead(() => ({
  htmlAttrs: { lang: head.value.htmlAttrs?.lang },
  link: [...(head.value.link || [])],
  meta: [...(head.value.meta || [])],
  title: article.value?.title || t('blog.title')
}))
</script>

<template>
  <article v-if="article">
    <!-- Notice si contenu non disponible dans la langue demandée -->
    <div
      v-if="article._isFallback"
      class="rounded-md bg-muted p-4 text-sm text-muted-foreground mb-6"
    >
      {{ t('content.notAvailableInLanguage') }}
    </div>

    <h1>{{ article.title }}</h1>
    <time>{{ new Date(article.date).toLocaleDateString(locale) }}</time>
    <ContentRenderer :value="article" />
  </article>
</template>
```

**Points clés du pattern :**

| Aspect | Implementation | Raison |
|--------|----------------|--------|
| `watch: [locale]` | Dans `useAsyncData` | Re-fetch automatique au changement de langue |
| `_isFallback` flag | Ajouté dynamiquement | Permet d'afficher une notice utilisateur |
| Collection dynamique | `articles_${locale.value}` | Évite les if/else multiples |
| Clé cache unique | `blog-${path}-${locale}` | Évite les collisions de cache |

**Traduction pour la notice fallback :**

```json
// i18n/locales/fr.json
{
  "content": {
    "notAvailableInLanguage": "Cet article n'est pas encore disponible en français. Vous consultez la version anglaise."
  }
}
```
