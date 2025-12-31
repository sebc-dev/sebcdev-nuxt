# Arborescence Projet Nuxt 4

```
sebc-dev/
├── app/                          # srcDir (nouveau défaut Nuxt 4)
│   ├── assets/
│   │   └── css/
│   │       └── main.css          # TailwindCSS entry point
│   ├── components/
│   │   ├── ui/                   # shadcn-vue (Reka UI)
│   │   ├── content/              # ArticleCard, TableOfContents
│   │   ├── layout/               # TheHeader, TheFooter
│   │   └── search/               # SearchCommand, SearchFilters
│   ├── composables/
│   │   └── index.ts              # Re-export si sous-dossiers
│   ├── layouts/
│   ├── pages/
│   ├── plugins/
│   │   └── ssr-width.ts
│   └── utils/
│       └── cn.ts                 # Utilitaire shadcn class merge
│
├── shared/                       # Code isomorphe partagé app/server
│   ├── types/                    # ✅ Auto-importé
│   │   └── article.ts
│   └── utils/                    # ✅ Auto-importé
│       └── validation.ts
│
├── content/                      # Nuxt Content 3 (RACINE obligatoire)
│   ├── fr/
│   └── en/
│
├── server/                       # Nitro (RACINE obligatoire)
│   ├── api/                      # Build-time uniquement en SSG
│   ├── routes/
│   │   └── rss.xml.ts
│   └── plugins/
│       └── llms-extend.ts
│
├── public/                       # Assets statiques (RACINE obligatoire)
│   ├── fonts/
│   ├── images/
│   ├── _headers                  # Headers Cloudflare
│   └── favicon.ico
│
├── nuxt.config.ts
├── content.config.ts             # Configuration Nuxt Content
├── wrangler.toml                 # Configuration D1 Cloudflare
├── pnpm-workspace.yaml           # Configuration pnpm 10
└── package.json
```

## Dossier `shared/` (Nuxt 3.14+)

Le dossier `shared/` permet de partager du code entre `app/` (Vue) et `server/` (Nitro). **Seuls `shared/utils/` et `shared/types/` sont auto-importés** — les autres fichiers nécessitent un import explicite via `#shared/path`.

```typescript
// shared/types/article.ts - Auto-importé, accessible partout
export interface Article {
  title: string
  slug: string
  pillar: 'ai' | 'engineering' | 'ux'
}

// shared/utils/formatReadingTime.ts - Auto-importé
export const formatReadingTime = (minutes: number, locale: string) =>
  locale === 'fr' ? `${minutes} min de lecture` : `${minutes} min read`
```

**⚠️ Restrictions importantes :**
- Le code dans `shared/` ne peut **PAS** importer de dépendances Vue ou Nitro spécifiques
- Uniquement du TypeScript isomorphe pur (types, fonctions utilitaires)
- Accès via alias `#shared` : `import { Article } from '#shared/types/article'`

| Dossier | Auto-importé | Alias |
|---------|--------------|-------|
| `shared/types/` | ✅ Oui | `#shared/types/*` |
| `shared/utils/` | ✅ Oui | `#shared/utils/*` |
| `shared/other/` | ❌ Non | `#shared/other/*` (import explicite) |

## Composables dans sous-dossiers (Piège courant)

**⚠️ Les composables dans des sous-dossiers de `composables/` ne sont PAS auto-scannés.**

```
app/composables/
├── useAuth.ts          # ✅ Auto-importé
├── useBlog.ts          # ✅ Auto-importé
└── search/
    └── useSearch.ts    # ❌ PAS auto-importé !
```

**Solutions :**

**Option 1 : Re-export dans `composables/index.ts`** (Recommandé)
```typescript
// app/composables/index.ts
export { useSearch } from './search/useSearch'
```

**Option 2 : Configuration `imports.dirs`**
```typescript
// nuxt.config.ts
imports: {
  dirs: ['composables/**']
}
```
