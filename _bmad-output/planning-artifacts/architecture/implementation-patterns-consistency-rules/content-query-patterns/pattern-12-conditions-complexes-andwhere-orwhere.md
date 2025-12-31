# Pattern 12 : Conditions complexes `.andWhere()` / `.orWhere()`

Pour des requêtes avec conditions imbriquées, utiliser `.andWhere()` et `.orWhere()` :

## Syntaxe avec callback

```typescript
// AND imbriqué
queryCollection(collection)
  .where('published', '=', true)
  .andWhere(q => q
    .where('date', '>', '2024-01-01')
    .where('category', '=', 'news')
  )
  .all()
// SQL: WHERE published = true AND (date > '2024-01-01' AND category = 'news')

// OR logique
queryCollection(collection)
  .where('featured', '=', true)
  .orWhere(q => q
    .where('pillar', '=', 'ai')
    .where('level', '=', 'advanced')
  )
  .all()
// SQL: WHERE featured = true OR (pillar = 'ai' AND level = 'advanced')
```

## Pattern pour filtrage multi-tags

```typescript
// Tous les articles qui ont TOUS les tags demandés
function filterByAllTags(tags: string[]) {
  let query = queryCollection(collection)
    .where('draft', '=', false)

  if (tags.length > 0) {
    query = query.andWhere(q => {
      tags.forEach(tag => {
        q.where('tags', 'LIKE', `%"${tag}"%`)
      })
      return q
    })
  }

  return query.all()
}
```
