# Open Graph Spécifications

## Dimensions et formats recommandés

| Propriété | Valeur | Notes |
|-----------|--------|-------|
| **og:image** | **1200×630px** | Ratio 1.91:1, format **PNG** (pas WebP - LinkedIn incompatible) |
| **og:title** | ≤65 caractères | Troncation après ~70 caractères |
| **og:description** | 150-200 caractères | Maximum visible selon plateforme |
| **og:type** | `article` | Active les propriétés `article:*` |
| **article:published_time** | ISO 8601 | `2025-12-29T09:00:00+01:00` (timezone obligatoire) |

## Pattern complet OG pour articles

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
