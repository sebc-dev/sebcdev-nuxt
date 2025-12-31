# Nouveautés MiniSearch 7.x

| Version | Changement | Impact |
|---------|------------|--------|
| **7.0.0** | Target ES6+ (plus de IE11) | Vérifier compatibilité navigateurs |
| **7.0.0** | `SearchableMap` import séparé | `import { SearchableMap } from 'minisearch/SearchableMap'` |
| **7.1.0** | Option `boostTerm` | Boost personnalisé par terme de recherche |
| **7.2.0** | Option `stringifyField` | Contrôle de la sérialisation des champs |
| **7.2.0** | Target ES2018 | Unicode regex dans tokenizer (`/\p{P}/u`) |

**Compatibilité index** : Les index sérialisés avec MiniSearch 6.x sont compatibles avec 7.x — pas besoin de reconstruire lors de la migration.
