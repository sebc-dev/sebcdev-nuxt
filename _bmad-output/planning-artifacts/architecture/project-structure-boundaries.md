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
│   │   └── usePagefind.ts               # Intégration Pagefind search
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
│   ├── plugins/
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
│   ├── pagefind/                        # Généré post-build par Pagefind
│   │   ├── pagefind.js                  # API client (36KB)
│   │   ├── pagefind-ui.js               # UI optionnelle
│   │   └── index/                       # Chunks d'index
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
├── .npmrc                               # Config pnpm 10 (lifecycle scripts)
├── components.json                      # Config shadcn-vue
├── content.config.ts                    # Collections Content 3
├── nuxt.config.ts                       # Config Nuxt principale
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
└── wrangler.toml                        # Config Cloudflare Pages (optionnel)
```

**Configuration .npmrc (pnpm 10):**
```yaml
# Autoriser explicitement les lifecycle scripts pour packages spécifiques
pnpm.onlyBuiltDependencies[]=sharp
pnpm.onlyBuiltDependencies[]=esbuild
pnpm.onlyBuiltDependencies[]=pagefind
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
SSG HTML Output (.output/public)
      ↓
Pagefind Indexation (post-build hook)
      ↓
Index Chunks (chargés à la demande)
      ↓
Vue Components (display)
```

**Search Flow (Pagefind):**

```
User Input → Pagefind API → Index Chunks Download → Results → Command Palette
              (36KB core)    (FR/EN stemming)
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
| **FR27-36 Recherche/Filtres** | `components/search/*`, `composables/usePagefind.ts`, `composables/useArticleFilters.ts` |
| **FR37-40 Bilingue** | `i18n/*`, `components/layout/LanguageSwitcher.vue` |
| **FR41-45 SEO/GEO** | `composables/useSeoMeta.ts`, `server/routes/llms.txt.ts` |
| **Performance** | `nuxt.config.ts` (features.inlineStyles, nuxt-vitalizer, hydrate-on-*), `@nuxt/image` |
| **A11y Tests** | `tests/a11y/*`, `@axe-core/playwright 4.11+` |

## External Integration Points

| Service | Intégration | Fichiers |
|---------|-------------|----------|
| **Plausible Analytics** | Script + runtimeConfig | `nuxt.config.ts`, `app.vue` |
| **Cloudflare Pages** | Build output + Pagefind | `wrangler.toml`, `nuxt.config.ts` |
| **GitHub** | Git push → auto-deploy | `.github/CODEOWNERS` |
| **Pagefind** | Post-build hook | `nuxt.config.ts` (hooks.nitro:build:public-assets) |

## Incompatibilités Identifiées

| Composant | Problème | Solution |
|-----------|----------|----------|
| `@nuxtjs/tailwindcss` v6.14 | Supporte TW4, mais moins adapté CSS-first | `@tailwindcss/vite` recommandé |
| shadcn-vue tooltips | ❌ WCAG SC 1.4.13 | Tests manuels NVDA/VoiceOver |
| SQLite WASM | Surdimensionné (500KB+) | Pagefind 1.4+ (~8KB + chunks) |
| `nuxt-delay-hydration` | ❌ Obsolète depuis Nuxt 3.16+ | Hydratation lazy native (hydrate-on-*) |
| `experimental.inlineSSRStyles` | ❌ Renommé en Nuxt 4 | `features.inlineStyles` |
| `radix-vue` | ❌ Rebrandé (février 2025) | `reka-ui` |
| `@zod/mini` package | ❌ Déprécié | `import { z } from 'zod'` ou `'zod/mini'` |
| `hooks.build:done` Pagefind | ❌ Timing incorrect | `hooks.nitro:build:public-assets` |

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
- Node version: `24.12.0` ou plus récent

**Avantages vérifiés (tier gratuit)** :
- Bande passante illimitée (pas de frais d'egress)
- Early Hints activé par défaut → **+30% LCP** pour nouveaux visiteurs

