# Pattern 3 : Fallback vers locale par défaut

Graceful degradation pour les articles non traduits :

```typescript
// pages/blog/[...slug].vue
<script setup lang="ts">
import { withLeadingSlash } from 'ufo'
import type { Collections } from '@nuxt/content'

const route = useRoute()
const { locale, defaultLocale } = useI18n()

const slug = computed(() => withLeadingSlash(String(route.params.slug || '')))

const { data: result } = await useAsyncData(
  `article-${locale.value}-${slug.value}`,
  async () => {
    // Tentative de récupération dans la collection de la locale courante
    const collection = `articles_${locale.value}` as keyof Collections
    const content = await queryCollection(collection)
      .path(slug.value)
      .first()

    if (content) {
      return { content, isFallback: false }
    }

    // Fallback vers locale par défaut si contenu manquant
    if (locale.value !== defaultLocale.value) {
      const fallbackCollection = `articles_${defaultLocale.value}` as keyof Collections
      const fallbackContent = await queryCollection(fallbackCollection)
        .path(slug.value)
        .first()

      if (fallbackContent) {
        console.warn(`[i18n] Article "${slug.value}" non traduit en ${locale.value}, fallback vers ${defaultLocale.value}`)
        return { content: fallbackContent, isFallback: true }
      }
    }

    return { content: null, isFallback: false }
  },
  { watch: [locale] }
)

const article = computed(() => result.value?.content)
const isFallback = computed(() => result.value?.isFallback ?? false)
</script>

<template>
  <div v-if="isFallback" class="bg-amber-100 dark:bg-amber-900/30 p-4 rounded-lg mb-6">
    <p class="text-amber-800 dark:text-amber-200">
      Cet article n'est pas encore traduit dans votre langue.
    </p>
  </div>

  <article v-if="article">
    <ContentRenderer :value="article" />
  </article>

  <div v-else>
    <!-- 404 ou message d'erreur -->
  </div>
</template>
```

**UX recommandée :**
- Afficher un bandeau "Cet article n'est pas encore traduit" si fallback utilisé
- Proposer un lien vers la version originale
