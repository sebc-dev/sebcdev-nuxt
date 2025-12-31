# Pattern 15 : `.select()` pour optimisation des listes

Pour les pages de listing, **ne pas charger le contenu complet** des articles. Utiliser `.select()` pour réduire drastiquement le payload.

## Pattern listing optimisé

```typescript
// pages/blog/index.vue
const { data: articles } = await useAsyncData(
  `blog-list-${locale.value}`,
  () => queryCollection(collection.value)
    .where('draft', '=', false)
    .order('publishedAt', 'DESC')
    // Sélectionner uniquement les champs nécessaires à l'affichage
    .select('title', 'path', 'description', 'publishedAt', 'pillar', 'image')
    .all()
)
```

## Comparaison payload

| Sans `.select()` | Avec `.select()` |
|------------------|------------------|
| ~50KB pour 10 articles | ~5KB pour 10 articles |
| Contenu MDC complet inclus | Métadonnées uniquement |
| Parsing MDC côté client | Pas de parsing nécessaire |

## Champs recommandés par contexte

| Contexte | Champs à sélectionner |
|----------|----------------------|
| **Liste blog** | `title`, `path`, `description`, `publishedAt`, `image` |
| **Sidebar récents** | `title`, `path`, `publishedAt` |
| **Articles liés** | `title`, `path`, `pillar`, `tags` |
| **Sitemap** | `path`, `publishedAt`, `updatedAt` |
