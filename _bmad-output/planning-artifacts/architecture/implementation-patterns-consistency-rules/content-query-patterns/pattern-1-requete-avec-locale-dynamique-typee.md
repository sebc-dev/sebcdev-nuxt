# Pattern 1 : Requête avec locale dynamique typée

Utiliser le cast TypeScript `as keyof Collections` pour les noms de collections dynamiques :

```typescript
// composables/useArticles.ts
import type { Collections } from '@nuxt/content'

export function useArticles() {
  const { locale } = useI18n()

  const getCollection = () => {
    // Cast sécurisé pour TypeScript - collection dynamique selon locale
    return `articles_${locale.value}` as keyof Collections
  }

  const { data: articles } = useAsyncData(
    `articles-${locale.value}`,
    () => queryCollection(getCollection())
      .where('draft', '=', false)
      .order('publishedAt', 'DESC')
      .all()
  )

  return { articles }
}
```

**Avantage collections séparées :** La requête interroge uniquement la table de la langue courante, réduisant les rows_read D1.
