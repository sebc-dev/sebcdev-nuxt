# Indexes D1 pour Optimisation

Les indexes sont définis dans `content.config.ts` pour chaque collection (voir [Content Architecture](#content-architecture)).

**Stratégie d'indexation D1 :**

| Index | Colonnes | Usage | Type |
|-------|----------|-------|------|
| `path` | `path` | Lookup par URL | Unique |
| `pillar` | `pillar` | Filtrage par pilier | Simple |
| `publishedAt` | `publishedAt` | Tri chronologique | Simple |
| `draft` | `draft` | Filtrage publié/brouillon | Simple |
| `idx_published` | `(draft, publishedAt)` | Liste publiés triés par date | Composite |

**⚠️ Impact sur les coûts D1 :** Un `WHERE` sur colonne indexée = **1 row read** au lieu d'un table scan complet. L'index composite `idx_published` optimise la requête typique `WHERE draft = false ORDER BY publishedAt DESC` en une seule lecture d'index.

**Avantage collections séparées :** Chaque collection (`articles_fr`, `articles_en`) a ses propres indexes isolés, réduisant la taille des tables scannées.

**Notes importantes:**
- `@nuxtjs/tailwindcss@6.14.0` supporte TW4, mais `@tailwindcss/vite` recommandé pour nouveaux projets
- `nuxt-delay-hydration` obsolète → hydratation lazy native Nuxt 4 (hydrate-on-visible, etc.)
- `experimental.inlineSSRStyles` renommé en `features.inlineStyles`
- `nuxt-vitalizer` 2.0: DelayHydration component supprimé → macros natives
- **Fonts preload** : `crossorigin="anonymous"` obligatoire même en self-hosting. Utiliser `font-display: optional` pour CLS = 0 (voir `tailwindcss-patterns.md`)
- **MiniSearch** : Index généré via script `postgenerate` dans `package.json` :
  ```json
  "scripts": {
    "generate": "nuxt generate",
    "postgenerate": "node scripts/generate-search-index.mjs"
  }
  ```
- **Index MiniSearch** : Fichier `public/search-index.json` généré au build
