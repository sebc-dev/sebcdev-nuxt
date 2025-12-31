# Pattern 10 : Fallback locale avec message explicite

Afficher un message clair quand un article n'est pas traduit :

```vue
<!-- pages/blog/[...slug].vue -->
<script setup lang="ts">
import type { Collections } from '@nuxt/content'

const route = useRoute()
const { locale, defaultLocale, t } = useI18n()

const slug = computed(() => `/${(route.params.slug as string[]).join('/')}`)

const { data: result } = await useAsyncData(
  `article-${slug.value}`,  // Clé sans locale pour cache partagé
  async () => {
    const collection = `articles_${locale.value}` as keyof Collections
    let content = await queryCollection(collection).path(slug.value).first()

    if (content) {
      return { content, isFallback: false, displayLocale: locale.value }
    }

    // Fallback vers locale par défaut
    if (locale.value !== defaultLocale.value) {
      const fallbackCollection = `articles_${defaultLocale.value}` as keyof Collections
      content = await queryCollection(fallbackCollection).path(slug.value).first()

      if (content) {
        return { content, isFallback: true, displayLocale: defaultLocale.value }
      }
    }

    return { content: null, isFallback: false, displayLocale: locale.value }
  },
  { watch: [locale] }
)

const article = computed(() => result.value?.content)
const isFallback = computed(() => result.value?.isFallback ?? false)
const displayLocale = computed(() => result.value?.displayLocale ?? locale.value)
</script>

<template>
  <!-- Bandeau fallback -->
  <div
    v-if="isFallback"
    class="bg-amber-50 dark:bg-amber-950 border-l-4 border-amber-500 p-4 mb-6"
    role="alert"
  >
    <div class="flex items-start gap-3">
      <Icon name="lucide:globe" class="w-5 h-5 text-amber-600 flex-shrink-0 mt-0.5" />
      <div>
        <p class="font-medium text-amber-800 dark:text-amber-200">
          {{ t('content.notTranslated') }}
        </p>
        <p class="text-sm text-amber-700 dark:text-amber-300 mt-1">
          {{ t('content.showingIn', { locale: displayLocale }) }}
        </p>
      </div>
    </div>
  </div>

  <!-- Contenu article -->
  <article v-if="article">
    <ContentRenderer :value="article" />
  </article>

  <!-- 404 -->
  <FallbackContent v-else :message="t('content.notFound')" />
</template>
```

## Traductions pour le message fallback

```json
// locales/fr.json
{
  "content": {
    "notTranslated": "Cet article n'est pas encore traduit en français.",
    "showingIn": "Affichage en {locale}.",
    "notFound": "Article introuvable."
  }
}

// locales/en.json
{
  "content": {
    "notTranslated": "This article is not yet translated to English.",
    "showingIn": "Showing in {locale}.",
    "notFound": "Article not found."
  }
}
```
