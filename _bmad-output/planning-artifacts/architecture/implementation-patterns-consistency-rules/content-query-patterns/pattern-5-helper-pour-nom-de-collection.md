# Pattern 5 : Helper pour nom de collection

Pour éviter la répétition du cast TypeScript :

```typescript
// composables/useContentCollection.ts
import type { Collections } from '@nuxt/content'

/**
 * Retourne le nom de collection articles pour la locale courante
 */
export function useArticlesCollection() {
  const { locale } = useI18n()

  const collectionName = computed(() =>
    `articles_${locale.value}` as keyof Collections
  )

  return collectionName
}

// Usage simplifié
const collection = useArticlesCollection()
const posts = await queryCollection(collection.value).all()
```
