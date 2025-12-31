# Pattern 11 : Opérateurs SQL dans `.where()`

La méthode `.where()` utilise des opérateurs SQL. Liste complète des opérateurs disponibles :

## Opérateurs de comparaison

| Opérateur | Usage | Exemple |
|-----------|-------|---------|
| `=` | Égalité exacte | `.where('category', '=', 'news')` |
| `>` | Supérieur à | `.where('date', '>', '2024-01-01')` |
| `<` | Inférieur à | `.where('views', '<', 1000)` |
| `<>` | Différent de | `.where('status', '<>', 'draft')` |
| `IN` | Dans une liste | `.where('pillar', 'IN', ['ai', 'engineering'])` |
| `BETWEEN` | Dans un intervalle | `.where('publishedAt', 'BETWEEN', ['2024-01-01', '2024-12-31'])` |
| `LIKE` | Pattern matching | `.where('path', 'LIKE', '/blog%')` |
| `IS NULL` | Valeur nulle | `.where('image', 'IS NULL', true)` |
| `IS NOT NULL` | Valeur non-nulle | `.where('featured', 'IS NOT NULL', true)` |

## Exemples de requêtes

```typescript
// Filtrage par date et catégorie
queryCollection(collection)
  .where('publishedAt', '>', '2024-01-01')
  .where('category', '=', 'tutorial')
  .all()

// Pattern matching avec LIKE (% = wildcard)
queryCollection(collection)
  .where('path', 'LIKE', '/blog%')  // Tous les articles dans /blog/*
  .all()

// Recherche dans les tags (tableau sérialisé)
queryCollection(collection)
  .where('tags', 'LIKE', '%"vue"%')  // Guillemets pour match exact
  .all()

// Vérification de valeur non-nulle
queryCollection(collection)
  .where('image', 'IS NOT NULL', true)
  .where('featured', '=', true)
  .all()
```
