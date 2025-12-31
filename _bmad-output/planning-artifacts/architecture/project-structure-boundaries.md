# Project Structure & Boundaries

## Complete Project Directory Structure

```
sebc-dev/
├── .github/
│   └── CODEOWNERS
├── .vscode/
│   └── settings.json                    # Recommandations VS Code
├── app/
│   ├── assets/
│   │   └── css/
│   │       └── main.css                 # TailwindCSS imports + custom
│   ├── components/
│   │   ├── content/
│   │   │   ├── ArticleCard.vue          # Card article liste
│   │   │   ├── ArticleCard.test.ts
│   │   │   ├── ArticleHero.vue          # Hero page article
│   │   │   ├── ArticleMeta.vue          # Date, temps lecture, badges
│   │   │   ├── ReadingProgress.vue      # Barre progression lecture
│   │   │   ├── TableOfContents.vue      # ToC sticky sidebar
│   │   │   └── TableOfContents.test.ts
│   │   ├── layout/
│   │   │   ├── TheHeader.vue            # Header global
│   │   │   ├── TheFooter.vue            # Footer global
│   │   │   ├── LanguageSwitcher.vue     # Switch FR/EN
│   │   │   ├── TheBreadcrumb.vue        # Fil d'Ariane
│   │   │   └── NavigationMenu.vue       # Menu dropdowns piliers
│   │   ├── prose/
│   │   │   ├── ProseCode.vue            # Code blocks enrichis
│   │   │   ├── ProseH2.vue              # H2 avec ancres ToC
│   │   │   ├── ProseH3.vue              # H3 avec ancres ToC
│   │   │   └── ProseDetails.vue         # Sections dépliables
│   │   ├── search/
│   │   │   ├── SearchCommand.vue        # Command palette (⌘K)
│   │   │   ├── SearchCommand.test.ts
│   │   │   ├── SearchFilters.vue        # Filtres multi-critères
│   │   │   └── SearchResults.vue        # Liste résultats
│   │   └── ui/                          # shadcn-vue (auto-généré)
│   │       ├── badge/
│   │       ├── button/
│   │       ├── command/
│   │       ├── dropdown-menu/
│   │       ├── input/
│   │       ├── select/
│   │       └── skeleton/
│   ├── composables/
│   │   ├── useReadingTime.ts            # Calcul temps lecture
│   │   ├── useArticleFilters.ts         # Logique filtres
│   │   ├── useTableOfContents.ts        # Section active ToC
│   │   ├── useSeoMeta.ts                # Meta SEO/Schema
│   │   └── useSearch.ts                 # Intégration MiniSearch
│   ├── layouts/
│   │   ├── default.vue                  # Layout principal
│   │   └── article.vue                  # Layout page article
│   ├── pages/
│   │   ├── index.vue                    # Page d'accueil
│   │   ├── articles/
│   │   │   ├── index.vue                # Liste articles + filtres
│   │   │   └── [slug].vue               # Page article détail
│   │   ├── piliers/
│   │   │   └── [pillar].vue             # Articles par pilier
│   │   ├── a-propos.vue                 # Page À propos
│   │   └── [...slug].vue                # Catch-all 404
│   ├── middleware/
│   │   ├── 01.analytics.global.ts       # Global - tracking (ordre numérique)
│   │   ├── 02.redirects.global.ts       # Global - redirections
│   │   └── draft-protection.ts          # Named - protection drafts
│   ├── plugins/
│   │   ├── formatting.ts                # Universal - helpers formatage
│   │   ├── analytics.client.ts          # Client only - tracking
│   │   └── ssr-width.ts                 # Fix hydration shadcn
│   └── utils/
│       └── formatDate.ts                # Formatage dates Intl
├── content/
│   ├── fr/
│   │   └── articles/                    # Articles FR (MDC)
│   │       └── exemple-article.md
│   └── en/
│       └── articles/                    # Articles EN (MDC)
│           └── example-article.md
├── i18n/
│   ├── locales/
│   │   ├── fr.json                      # Traductions UI FR
│   │   └── en.json                      # Traductions UI EN
│   └── config.ts                        # Config i18n
├── public/
│   ├── favicon.ico
│   ├── robots.txt
│   ├── llms.txt                         # Généré au build
│   ├── search-index.json                # Généré post-build par script MiniSearch
│   └── images/
│       └── og/                          # Images OpenGraph
├── server/
│   ├── routes/
│   │   ├── rss.xml.ts                   # Feed RSS
│   │   └── llms.txt.ts                  # Génération llms.txt
│   └── utils/
│       └── content.ts                   # Helpers Content 3
├── types/
│   ├── article.ts                       # Article, Pillar, Category, Level
│   ├── search.ts                        # SearchFilters, SearchResult
│   └── navigation.ts                    # NavItem, Breadcrumb
├── .env.example                         # Variables environnement
├── .gitignore
├── pnpm-workspace.yaml                  # Config pnpm 10 (optionnel, alternative au champ pnpm dans package.json)
├── components.json                      # Config shadcn-vue
├── content.config.ts                    # Collections Content 3
├── nuxt.config.ts                       # Config Nuxt principale
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
└── wrangler.toml                        # Config Cloudflare D1 (OBLIGATOIRE)
```

**Configuration wrangler.toml (OBLIGATOIRE) :**

```toml
name = "sebc-dev"
compatibility_date = "2024-09-19"

[[d1_databases]]
binding = "DB"
database_name = "content-db"
database_id = "VOTRE_DATABASE_ID"  # Généré par: wrangler d1 create content-db
```

> **⚠️ CRITIQUE :** Sans cette configuration, le runtime Cloudflare Worker échouera avec une erreur 500 car il n'a pas accès au filesystem pour lire SQLite.

**Configuration pnpm 10 (dans package.json) :**
```json
{
  "pnpm": {
    "onlyBuiltDependencies": ["esbuild", "sharp"]
  }
}
```

> **Note :** La syntaxe `.npmrc` avec `pnpm.onlyBuiltDependencies[]` est incorrecte pour pnpm 10.

## Conventions de Nommage Fichiers

**Suffixes spéciaux Nuxt:**

| Suffixe | Contexte | Exemple |
|---------|----------|---------|
| `.client.ts` | Client uniquement | `analytics.client.ts` |
| `.server.ts` | Serveur uniquement | `database.server.ts` |
| `.server.vue` | Server Component (0 JS client) | `BlogContent.server.vue` |
| `.global.ts` | Middleware global | `01.analytics.global.ts` |

**Ordre d'exécution middleware global:**
- Numérotation préfixée: `01.setup.global.ts`, `02.redirects.global.ts`
- Exécutés dans l'ordre alphabétique

**Auto-import composables:**

```
app/composables/
├── index.ts              # Scanné ✅
├── useBlogPost.ts        # Scanné ✅
├── useBlogSearch.ts      # Scanné ✅
└── utils/
    └── helpers.ts        # NON scanné par défaut
```

Pour scanner les sous-dossiers:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  imports: {
    dirs: ['composables', 'composables/**']
  }
})
```

## Architectural Boundaries

**Component Boundaries:**

```
┌─────────────────────────────────────────────────────────────┐
│                        Layout (default/article)              │
├─────────────────────────────────────────────────────────────┤
│  TheHeader          │  Main Content        │  (Sidebar)     │
│  ├─ NavigationMenu  │  ├─ ArticleHero     │  TableOfContents│
│  ├─ SearchCommand   │  ├─ ArticleContent  │                 │
│  └─ LanguageSwitcher│  └─ ArticleMeta     │                 │
├─────────────────────────────────────────────────────────────┤
│                        TheFooter                             │
└─────────────────────────────────────────────────────────────┘
```

**Data Flow:**

```
Content (MDC files)
      ↓
Content 3 Collections (build time)
      ↓
┌─────────────────────────────────────┐
│ Génération simultanée               │
├─────────────────────────────────────┤
│ • SSG HTML Output (.output/public)  │
│ • Migrations D1 (schéma + données)  │
└─────────────────────────────────────┘
      ↓
MiniSearch Index Generation (post-build script)
      ↓
Cloudflare Deployment
      ↓
┌─────────────────────────────────────┐
│ Runtime Cloudflare Worker           │
├─────────────────────────────────────┤
│ • HTML statique (CDN Edge)          │
│ • API Content → D1 (Worker)         │
└─────────────────────────────────────┘
      ↓
Vue Components (display)
```

**⚠️ Sans D1 :** Le Worker n'a pas accès au filesystem → Erreur 500 sur toute requête API Content

**Search Flow (MiniSearch):**

```
Command Palette Open → Fetch search-index.json → Initialize MiniSearch (~7KB)
                              ↓
User Input → MiniSearch.search() → Fuzzy/Prefix Matching → Results Display
           (boosted: title 2x)    (stemming FR/EN)
```

## Requirements to Structure Mapping

| FR/Feature | Fichiers Concernés |
|------------|-------------------|
| **FR1-7 Lecture Contenu** | `pages/articles/[slug].vue`, `components/content/*`, `layouts/article.vue` |
| **FR8 ToC** | `components/content/TableOfContents.vue`, `composables/useTableOfContents.ts` |
| **FR9 Progression** | `components/content/ReadingProgress.vue` |
| **FR11-13 Code Blocks** | `components/prose/ProseCode.vue` |
| **FR14-20 Navigation** | `components/layout/*`, `pages/piliers/[pillar].vue` |
| **FR21-24 Badges** | `components/ui/badge/`, `components/content/ArticleMeta.vue` |
| **FR27-36 Recherche/Filtres** | `components/search/*`, `composables/useSearch.ts`, `composables/useArticleFilters.ts` |
| **FR37-40 Bilingue** | `i18n/*`, `components/layout/LanguageSwitcher.vue` |
| **FR41-45 SEO/GEO** | `composables/useSeoMeta.ts`, `server/routes/llms.txt.ts` |
| **Performance** | `nuxt.config.ts` (features.inlineStyles, nuxt-vitalizer, hydrate-on-*), `@nuxt/image` |
| **A11y Tests** | `tests/a11y/*`, `@axe-core/playwright 4.11+` |

## External Integration Points

| Service | Intégration | Fichiers |
|---------|-------------|----------|
| **Cloudflare D1** | Base de données Content (OBLIGATOIRE) | `wrangler.toml`, `nuxt.config.ts` |
| **Cloudflare Pages** | Build output + Worker | `wrangler.toml`, `nuxt.config.ts` |
| **Plausible Analytics** | Script + runtimeConfig | `nuxt.config.ts`, `app.vue` |
| **GitHub** | Git push → auto-deploy | `.github/CODEOWNERS` |
| **MiniSearch** | Script post-build | `package.json` (script: "postgenerate": "node scripts/generate-search-index.mjs") |

## Incompatibilités Identifiées

| Composant | Problème | Solution |
|-----------|----------|----------|
| `@nuxtjs/tailwindcss` v6.14 | Supporte TW4, mais moins adapté CSS-first | `@tailwindcss/vite` recommandé |
| shadcn-vue tooltips | ❌ WCAG SC 1.4.13 | Tests manuels NVDA/VoiceOver |
| SQLite WASM | Surdimensionné (500KB+) | MiniSearch (~7KB minified) |
| `nuxt-delay-hydration` | ❌ Obsolète depuis Nuxt 3.16+ | Hydratation lazy native (hydrate-on-*) |
| `experimental.inlineSSRStyles` | ❌ Renommé en Nuxt 4 | `features.inlineStyles` |
| `radix-vue` | ❌ Rebrandé (février 2025) | `reka-ui` |
| `@zod/mini` package | ❌ Déplacé (mai 2025) | `import { z } from 'zod'` ou `import * as z from 'zod/mini'` (path, pas package) |

## Paramètres Cloudflare Pages

**À désactiver dans le dashboard** (causent problèmes d'hydratation) :
- ❌ Rocket Loader™
- ❌ Mirage (Image Optimization)
- ❌ Email Address Obfuscation
- ❌ Auto-minification (déprécié août 2024 → minifier au build)

**À activer manuellement (domaines custom)** :
- ✅ HTTP/3 (activé par défaut sur `*.pages.dev` uniquement)

**Configuration Build** :
- Framework preset: `Nuxt.js`
- Build command: `pnpm run build`
- Build output directory: `.output/public`
- Node version: `22 LTS` (version stable recommandée)

**Avantages vérifiés (tier gratuit)** :
- Bande passante illimitée (pas de frais d'egress)
- Early Hints activé par défaut → **+30% LCP** pour nouveaux visiteurs

