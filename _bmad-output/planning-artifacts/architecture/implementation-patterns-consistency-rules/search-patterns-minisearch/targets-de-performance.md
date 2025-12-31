# Targets de Performance

Objectifs mesurables pour valider l'implémentation de la recherche :

| Métrique | Target | Mesure |
|----------|--------|--------|
| **Index size** | < 5MB par locale | `ls -la .output/public/search-index.json` |
| **Index load time** | < 500ms sur 3G | DevTools Network (Slow 3G) |
| **Search latency** | < 50ms après debounce | `console.time()` autour de `miniSearch.search()` |
| **Time to first result** | < 300ms depuis keystroke | UX perçue (debounce 250ms + search 50ms) |
| **Modal open animation** | 150-200ms | CSS transition duration |
| **Debounce timing** | 250ms (ajustable) | Configurable dans `useSearch()` |
| **Bundle MiniSearch** | ~7KB gzipped | `npx nuxi analyze` |

**Seuils d'alerte :**

| Condition | Action |
|-----------|--------|
| Index > 10MB | Activer `truncateContent(2000)` |
| Load time > 1s | Implémenter parallel load pattern |
| Search latency > 100ms | Réduire `maxResults` ou simplifier fuzzy |
| > 500 articles | Évaluer Web Worker ou Pagefind |

**Outils de mesure :**

```typescript
// Mesurer la latence de recherche
const start = performance.now()
const results = miniSearch.search(query)
console.log(`Search latency: ${(performance.now() - start).toFixed(2)}ms`)
```

---
