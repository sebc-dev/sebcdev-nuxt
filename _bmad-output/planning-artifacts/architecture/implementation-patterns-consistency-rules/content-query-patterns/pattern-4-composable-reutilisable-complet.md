# Pattern 4 : Composable réutilisable complet

```typescript
// composables/useLocalizedContent.ts
import type { Collections } from '@nuxt/content'

interface LocalizedContentResult<T> {
  content: T | null
  isFallback: boolean
  originalLocale: string
}

interface UseLocalizedContentOptions {
  fallbackToDefault?: boolean
}

export function useLocalizedContent(
  path: MaybeRef<string>,
  options: UseLocalizedContentOptions = { fallbackToDefault: true }
) {
  const { locale, defaultLocale } = useI18n()
  const resolvedPath = toRef(path)

  const { data, pending, error } = useAsyncData(
    `content-${locale.value}-${resolvedPath.value}`,
    async (): Promise<LocalizedContentResult<unknown>> => {
      // Collection de la locale courante
      const collection = `articles_${locale.value}` as keyof Collections
      const content = await queryCollection(collection)
        .path(resolvedPath.value)
        .first()

      if (content) {
        return { content, isFallback: false, originalLocale: locale.value }
      }

      // Fallback vers locale par défaut
      if (options.fallbackToDefault && locale.value !== defaultLocale.value) {
        const fallbackCollection = `articles_${defaultLocale.value}` as keyof Collections
        const fallbackContent = await queryCollection(fallbackCollection)
          .path(resolvedPath.value)
          .first()

        if (fallbackContent) {
          return {
            content: fallbackContent,
            isFallback: true,
            originalLocale: defaultLocale.value
          }
        }
      }

      return { content: null, isFallback: false, originalLocale: locale.value }
    },
    { watch: [locale, resolvedPath] }
  )

  const content = computed(() => data.value?.content ?? null)
  const isFallback = computed(() => data.value?.isFallback ?? false)
  const originalLocale = computed(() => data.value?.originalLocale ?? locale.value)

  return { content, isFallback, originalLocale, pending, error }
}
```

**Usage :**

```vue
<script setup lang="ts">
const route = useRoute()
const { content, isFallback, originalLocale } = useLocalizedContent(() => route.path)
</script>

<template>
  <div v-if="isFallback" class="bg-amber-100 p-4 rounded">
    Cet article est affiché en {{ originalLocale }} car il n'est pas encore traduit.
  </div>
  <ContentRenderer v-if="content" :value="content" />
</template>
```
