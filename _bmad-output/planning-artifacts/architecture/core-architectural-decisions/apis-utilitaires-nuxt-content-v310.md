# APIs Utilitaires @nuxt/content v3.10+

La version 3.10+ introduit des APIs supplémentaires pour les cas d'usage courants :

```typescript
import type { Collections } from '@nuxt/content'

const { locale } = useI18n()
const collection = `articles_${locale.value}` as keyof Collections

// Compter les résultats (évite de charger toutes les données)
const count = await queryCollection(collection)
  .where('draft', '=', false)
  .count()

// Navigation prev/next pour article courant
const [prev, next] = await queryCollectionItemSurroundings(
  collection,
  route.path,
  { fields: ['title', 'path', 'publishedAt'] }
)
```

**Cas d'usage :**

| API | Usage | Alternative |
|-----|-------|-------------|
| `count()` | Afficher "X articles" sans charger les données | `all().length` (moins performant) |
| `queryCollectionItemSurroundings()` | Navigation prev/next article | Requête manuelle avec `order()` |

**Note :** `queryCollectionSearchSections()` existe mais **non recommandé** pour ce projet - MiniSearch (index pré-généré) est plus performant et ne consomme pas le quota D1.
