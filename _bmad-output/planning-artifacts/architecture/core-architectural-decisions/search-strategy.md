# Search Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Search Engine** | MiniSearch 7.x | ~7KB minified, zéro dépendance, API flexible |
| **Indexation** | Script build-time + JSON statique | Index pré-généré dans `public/search-index.json` |
| **Loading** | Index complet au premier accès | Fetch lazy sur ouverture search (⌘K) |
| **UX** | Command palette | Intégré avec shadcn-vue Command component (⌘K) |

**Avantages MiniSearch vs Pagefind :**
- Bundle plus léger (~7KB vs ~8KB + chunks externes)
- Pas de fichiers externes à charger dynamiquement
- Boosting configurable par champ (`title: 2`, `content: 1`)
- Fuzzy search et prefix search natifs
- Stemming FR via option `stemmer`
