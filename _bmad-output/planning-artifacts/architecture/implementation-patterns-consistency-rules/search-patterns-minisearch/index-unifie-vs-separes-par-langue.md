# Index Unifié vs Séparés par Langue

Pour un blog SSG bilingue, deux stratégies sont possibles :

| Critère | Index unifié | Index séparés |
|---------|-------------|---------------|
| Fichiers générés | 1 (`search-index.json`) | 2 (`search-index-fr.json`, `search-index-en.json`) |
| Recherche cross-language | ✅ Possible | ❌ Non |
| Complexité build | Simple | Plus élevée |
| Lazy loading | 1 requête | 1 requête par langue |
| Taille totale | ~10% plus grand | Légèrement plus petit |
| Stemming | Auto-détection | Optimal par langue |

**Recommandation** : Index **unifié** pour <1000 articles. Filtrage par locale au moment de la recherche :

```typescript
// Recherche avec filtre de langue
function searchPosts(
  index: MiniSearch<BlogSearchDocument>,
  query: string,
  options?: { lang?: 'fr' | 'en'; limit?: number }
) {
  const { lang, limit = 10 } = options || {}

  return index.search(query, {
    filter: lang ? (result) => result.locale === lang : undefined,
    prefix: true,
    fuzzy: 0.2
  }).slice(0, limit)
}

// Usage
const results = searchPosts(miniSearch, 'vue composables', { lang: 'fr' })
```

L'index **séparé** n'est recommandé que pour >1000 articles par langue ou si le stemming parfait est critique.
