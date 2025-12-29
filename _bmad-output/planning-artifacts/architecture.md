---
stepsCompleted: [1, 2, 3, 4, 5, 6]
inputDocuments:
  - product-brief-sebcdev-nuxt.md
  - prd/index.md
  - prd/1-executive-summary.md
  - prd/2-personas-dtaills.md
  - prd/3-analyse-concurrentielle.md
  - prd/5-stratgie-bilingue.md
  - prd/7-structure-contenu-answer-first.md
  - prd/10-roadmap-dtaille.md
  - prd/13-functional-requirements-mvp.md
  - prd/14-non-functional-requirements-mvp.md
workflowType: 'architecture'
project_name: 'sebc.dev'
user_name: 'Negus'
date: '2025-12-29'
---

# Architecture Decision Document

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
50 FRs couvrant un blog technique bilingue avec système de navigation avancé, filtrage multi-critères, expérience de lecture enrichie (ToC, progression, code blocks), et optimisation GEO pour visibilité dans les moteurs IA.

**Non-Functional Requirements:**
- Performance: Lighthouse 100/100/100/100, LCP < 2.5s, CLS < 0.1
- Accessibilité: WCAG 2.1 AA, navigation clavier 100%
- SEO: Score 100, Schema.org validé, crawl < 500ms
- Sécurité: HTTPS obligatoire, headers CSP, Mozilla Observatory B+
- Disponibilité: 99.5% uptime mensuel

**Scale & Complexity:**

- Primary domain: Full-stack Content Platform (Nuxt 4 Static)
- Complexity level: Medium-High
- Estimated architectural components: ~8-12 modules principaux

### Technical Stack Decisions

| Technologie | Version | Rôle |
|-------------|---------|------|
| **Nuxt** | 4+ | Framework full-stack, SSG mode |
| **Nuxt Content** | 3 | CMS fichiers Markdown/MDC |
| **Content Dump** | Compressé | Bundle statique téléchargé client |
| **Search Engine** | WebAssembly | SQLite WASM pour recherche full-text client-side |
| **TailwindCSS** | 4 | Styling utility-first, config CSS-native |
| **shadcn-vue** | Latest | Composants UI accessibles (Radix Vue) |
| **Hébergement** | Cloudflare Pages | Edge deployment, 100% statique |

### Implications Architecturales du Stack

**Nuxt Content 3 + WASM Search :**
- Génération statique complète (SSG) - pas de server functions pour le contenu
- Le client télécharge le dump compressé au premier chargement
- Recherche instantanée sans latence réseau après chargement initial
- Filtrage multi-critères entièrement côté client
- Zéro coût serveur (aligne parfaitement avec budget 0€)
- Mise à jour du dump = rebuild + redéploiement

**Nuxt 4+ Structure :**
- Nouvelle arborescence `app/` pour composants/pages/layouts
- `server/` pour API routes (minimal dans notre cas)
- Meilleure séparation des préoccupations
- TypeScript-first avec inférence améliorée

**TailwindCSS 4 + shadcn-vue :**
- Design system cohérent via CSS variables Tailwind
- Mode sombre unique = configuration simplifiée (pas de toggle)
- Composants shadcn copiés dans `components/ui/` - ownership total
- Accessibilité WCAG AA incluse via Radix Vue primitives
- Command component idéal pour la recherche (⌘K palette)
- Dropdown menus pour navigation header (thèmes, catégories, niveaux)

### Technical Constraints & Dependencies

| Contrainte | Impact Architectural |
|------------|---------------------|
| Budget 0€ | Architecture 100% statique, services gratuits |
| Solo dev | shadcn-vue = composants prêts, moins de code custom |
| Nuxt 4 + Content 3 | Stack moderne, SSG, client-side search |
| TailwindCSS 4 | CSS-first config, design tokens via variables |
| shadcn-vue | Composants accessibles, copy-paste ownership |
| Zod v4 | Validation schéma Content 3, Standard Schema |
| Cloudflare Pages | Edge deployment, pas de server compute |
| Mode sombre unique | Palette dark fixe, pas de theme switching |
| Bilingue natif | i18n intégré, contenu FR/EN dans Content |

### Cross-Cutting Concerns Identified

1. **Internationalization (i18n)** - Routes, contenu dual-language, UI labels, SEO hreflang
2. **Performance Optimization** - SSG, preloading dump, lazy hydration, image optimization
3. **Accessibility (a11y)** - shadcn/Radix primitives, keyboard nav, ARIA, contrasts
4. **SEO/GEO Dual Optimization** - Pre-rendered HTML, Schema, llms.txt, Answer-First
5. **Content Architecture** - MDC parsing, frontmatter schema, taxonomy (piliers/catégories/tags)
6. **Client-Side Search** - Dump size management, WASM loading, Command palette UX
7. **Design System** - Tailwind 4 tokens, shadcn components, cohérence visuelle

## Starter Template Evaluation

### Primary Technology Domain

Full-stack Content Platform (Nuxt 4 SSG) - génération statique complète pour Cloudflare Pages.

### Approche Sélectionnée : Initialisation Modulaire

**Rationale :** Étant donné le stack spécifique (Nuxt 4 + Content 3 + TailwindCSS 4 + shadcn-vue), une approche modulaire garantit les versions les plus récentes et un contrôle total sur chaque composant. Aucun template existant ne combine exactement ces technologies dans leurs dernières versions.

### Commande d'Initialisation

```bash
# Créer projet Nuxt 4
pnpm dlx nuxi@latest init sebc-dev
cd sebc-dev && pnpm install

# Modules essentiels
pnpm add @nuxt/content
pnpm add -D @tailwindcss/vite tailwindcss shadcn-nuxt
pnpm dlx nuxi@latest module add i18n

# Validation de schéma
pnpm add zod

# Initialiser shadcn-vue
pnpm dlx shadcn-vue@latest init
```

### Décisions Architecturales du Starter

**Language & Runtime:**
- TypeScript strict par défaut
- Node.js 22+ LTS (ou 24+ pour nouvelles installations)
- pnpm comme package manager

**Styling Solution:**
- TailwindCSS 4 via Vite plugin (@tailwindcss/vite)
- Configuration CSS-first (pas de tailwind.config.js)
- CSS variables pour design tokens

**Build Tooling:**
- Vite comme bundler (intégré Nuxt)
- SSG mode pour génération statique
- Cloudflare Pages compatible

**UI Components:**
- shadcn-vue avec Radix Vue primitives
- Composants dans app/components/ui/
- Accessibilité WCAG AA native

**Content Management:**
- Nuxt Content 3 avec collections
- Fichiers MDC dans content/
- Dump compressé + WASM search

**Internationalization:**
- @nuxtjs/i18n v10+
- Strategy: prefix (/fr/, /en/)
- Lazy-loading des traductions

### Configuration nuxt.config.ts de Base

```typescript
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  future: { compatibilityVersion: 4 },
  compatibilityDate: '2025-01-01',

  modules: [
    '@nuxt/content',
    '@nuxtjs/i18n',
    'shadcn-nuxt',
  ],

  css: ['~/assets/css/main.css'],

  vite: {
    plugins: [tailwindcss()],
  },

  i18n: {
    locales: [
      { code: 'fr', language: 'fr-FR', name: 'Français' },
      { code: 'en', language: 'en-US', name: 'English' },
    ],
    defaultLocale: 'fr',
    strategy: 'prefix',
  },

  shadcn: {
    prefix: '',
    componentDir: './app/components/ui',
  },
})
```

### Arborescence Projet Nuxt 4

```
sebc-dev/
├── app/
│   ├── assets/
│   │   └── css/
│   │       └── main.css
│   ├── components/
│   │   └── ui/                   # shadcn-vue
│   ├── composables/
│   ├── layouts/
│   ├── pages/
│   └── plugins/
│       └── ssr-width.ts
├── content/
│   ├── fr/
│   └── en/
├── server/
├── public/
├── nuxt.config.ts
└── package.json
```

**Note:** L'initialisation du projet avec ces commandes sera la première story d'implémentation.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Content structure: By language with single collection
- Frontmatter schema: English constants, auto-calculated readingTime
- Component architecture: Structured folders with custom Prose components
- SEO/GEO: Auto-generated llms.txt, Schema.org via @unhead

**Important Decisions (Shape Architecture):**
- State management: Vue composables only (no Pinia)
- Image optimization: @nuxt/image + Cloudflare provider
- CI/CD: Cloudflare Pages direct Git integration

**Deferred Decisions (Post-MVP):**
- Newsletter integration
- Comments system
- Advanced analytics (heatmaps)

### Content Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **File Organization** | By language (`content/fr/`, `content/en/`) | Natural fit with @nuxtjs/i18n, clear separation |
| **Collections** | Single `articles` collection | Simpler queries, filter by pillar |
| **Frontmatter Constants** | English (`ai`, `tutorial`, `beginner`) | Consistency, i18n-agnostic |
| **Reading Time** | Auto-calculated (200 words/min) | Less maintenance, always accurate |

**Frontmatter Schema:**

```yaml
title: string
description: string
slug: string
pillar: 'ai' | 'engineering' | 'ux'
category: 'news' | 'tutorial' | 'deep-dive' | 'case-study' | 'retrospective'
level: 'all' | 'beginner' | 'intermediate' | 'advanced'
tags: string[]
publishedAt: date
updatedAt: date
image: string
draft: boolean
```

### Frontend Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Component Structure** | `ui/`, `content/`, `layout/`, `search/` | Clear separation of concerns |
| **State Management** | Vue composables only | SSG blog doesn't need global store |
| **Prose Components** | Custom (ProseCode, ProseH2, ProseDetails) | Required for FR11-13 (code blocks), FR8 (ToC) |

**Component Organization:**

```
app/components/
├── ui/                    # shadcn-vue
├── content/               # ArticleCard, TableOfContents, ReadingProgress, CodeBlock
├── layout/                # TheHeader, TheFooter, LanguageSwitcher
└── search/                # SearchCommand, SearchFilters
```

### SEO & GEO Implementation

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **llms.txt** | Auto-generated at build | Always current, zero maintenance |
| **Schema Markup** | nuxt-schema-org | Type-safe, TechArticle + FAQ support, Content 3 integration |
| **Sitemap** | @nuxtjs/sitemap | Official module, hreflang support |
| **RSS** | Server route generation | Native Content 3 integration |

### Infrastructure & Deployment

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Image Optimization** | @nuxt/image + Cloudflare provider | Auto WebP, lazy loading, CDN |
| **CI/CD** | Cloudflare Pages direct | Zero config, preview branches, fast |
| **Environment Config** | .env + runtimeConfig + CF fallback | Local dev + prod parity |

**Deployment Flow:**

```
Git Push → Cloudflare Pages Build → SSG Output → Edge Deployment
                ↓
         Preview URL (branches)
```

### Modules Nuxt Finaux

```typescript
modules: [
  '@nuxt/content',
  '@nuxtjs/i18n',
  '@nuxt/image',
  '@nuxtjs/sitemap',
  'nuxt-schema-org',
  'shadcn-nuxt',
]
```

### Validation Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Schema Validator** | Zod v4 | Standard Schema, JSON Schema natif sans dépendance extra |
| **Import Source** | `import { z } from 'zod'` | Re-export `@nuxt/content` déprécié, sera supprimé |
| **Validation Scope** | Build time | Erreurs détectées au build, pas en production |
| **Type Inference** | `z.infer<typeof schema>` | Types TypeScript générés automatiquement |

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

**15 points de conflit potentiels** identifiés et résolus pour garantir la cohérence entre agents IA.

### Naming Patterns

**File Naming Conventions:**

| Type | Convention | Exemple |
|------|------------|---------|
| Composants Vue | PascalCase | `ArticleCard.vue` |
| Pages Nuxt | kebab-case | `[slug].vue` |
| Composables | camelCase + `use` | `useReadingTime.ts` |
| Utilitaires | camelCase | `formatDate.ts` |
| Types | PascalCase | `Article.ts` |
| Contenu MDC | kebab-case | `mon-premier-article.md` |

**i18n Key Naming:**
- Nested objects structure
- Groupé par feature/section

```typescript
// ✅ Correct
{
  header: {
    nav: { home: "Accueil", articles: "Articles" }
  },
  article: {
    readingTime: "{min} min de lecture"
  }
}
```

**CSS/Tailwind Naming:**
- Classes custom: kebab-case (`article-card`)
- CSS variables: `--color-primary`, `--spacing-lg`
- Animations: kebab-case (`fade-in`, `slide-up`)

### Structure Patterns

**Test Organization:**
- Co-located avec le code source
- `Component.vue` → `Component.test.ts` (même dossier)

**Composables Organization:**
- Un fichier par composable
- Flat structure dans `app/composables/`

```
app/composables/
├── useReadingTime.ts
├── useArticleFilters.ts
├── useTableOfContents.ts
└── useSeoMeta.ts
```

**TypeScript Types:**
- Groupés par domaine dans `types/`

```
types/
├── article.ts      # Article, ArticleMeta, Pillar, Category
├── search.ts       # SearchFilters, SearchResult
└── navigation.ts   # NavItem, Breadcrumb
```

### Format Patterns

**Date Formatting:**
- Frontmatter: ISO 8601 (`2025-01-15`)
- Display: `Intl.DateTimeFormat` natif (zéro dépendance)

```typescript
// ✅ Correct
const formatDate = (date: Date, locale: string) =>
  new Intl.DateTimeFormat(locale, {
    day: 'numeric',
    month: 'long',
    year: 'numeric'
  }).format(date)
```

**Error Format (server routes):**

```typescript
// Format simple
{ error: string, statusCode: number }

// Exemple
{ error: "Article not found", statusCode: 404 }
```

**JSON Conventions:**
- camelCase pour toutes les propriétés
- Cohérent avec TypeScript/JavaScript

### Communication Patterns

**Vue Events:**
- kebab-case: `@article-selected`, `@filter-changed`

```vue
<!-- ✅ Correct -->
<ArticleCard @article-selected="handleSelect" />

<!-- ❌ Incorrect -->
<ArticleCard @articleSelected="handleSelect" />
```

**Component Props:**
- Inline `defineProps<{}>` pour composants simples
- Interface exportée si props réutilisées ailleurs

```typescript
// Simple component
defineProps<{
  article: Article
  showExcerpt?: boolean
}>()

// Reusable props
export interface ArticleCardProps {
  article: Article
  variant: 'card' | 'hero' | 'list'
}
```

**Loading States:**
- Naming: `isLoading`, `isPending`
- UI: `<Skeleton />` de shadcn-vue
- Pattern: destructure depuis `useAsyncData`

### Process Patterns

**Error Handling Hierarchy:**

| Niveau | Approche | Exemple |
|--------|----------|---------|
| Composant | `try/catch` + état local | Form validation |
| Page | `useAsyncData` error | Article fetch |
| Global | `error.vue` | 404, 500 |

**Logging:**
- Console native (`console.log`, `console.error`)
- Pas de wrapper custom pour MVP

**Data Validation:**
- Validateur: Zod v4 (Standard Schema compatible)
- Import direct depuis `zod` (re-export `@nuxt/content` déprécié)
- Validation au build time via Content 3 collections

```typescript
// content.config.ts
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { z } from 'zod'  // ✅ Import direct - ne pas utiliser @nuxt/content

export default defineContentConfig({
  collections: {
    articles: defineCollection({
      type: 'page',
      source: '**/*.md',
      schema: z.object({
        title: z.string(),
        description: z.string(),
        slug: z.string(),
        pillar: z.enum(['ai', 'engineering', 'ux']),
        category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
        level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),
        tags: z.array(z.string()),
        publishedAt: z.coerce.date(),
        updatedAt: z.coerce.date().optional(),
        image: z.string().optional(),
        draft: z.boolean().default(false),
      })
    })
  }
})
```

**Intégration Schema.org avec Content 3:**
```typescript
// content.config.ts
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { asSchemaOrgCollection } from 'nuxt-schema-org/content'
import { z } from 'zod'

export default defineContentConfig({
  collections: {
    articles: defineCollection(
      asSchemaOrgCollection({
        type: 'page',
        source: '**/*.md',
        schema: z.object({
          title: z.string(),
          description: z.string(),
          pillar: z.enum(['ai', 'engineering', 'ux']),
          category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
          level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),
          tags: z.array(z.string()),
          publishedAt: z.date(),
          updatedAt: z.date().optional(),
          image: z.string().optional(),
          draft: z.boolean().default(false),
        })
      })
    )
  }
})
```

### Enforcement Guidelines

**All AI Agents MUST:**

1. Suivre les conventions de nommage définies ci-dessus
2. Placer les tests à côté des fichiers sources
3. Utiliser `Intl.DateTimeFormat` pour le formatage de dates
4. Émettre les events Vue en kebab-case
5. Valider le contenu via Content 3 schema

**Anti-Patterns à Éviter:**
```typescript
// ❌ snake_case dans JSON
{ published_at: "2025-01-15" }

// ❌ Tests dans dossier séparé
tests/components/ArticleCard.test.ts

// ❌ Date formatting avec lib externe
import { format } from 'date-fns'

// ❌ Events en camelCase
emit('articleSelected')

// ❌ Import zod depuis @nuxt/content (déprécié)
import { z } from '@nuxt/content'
```

## Project Structure & Boundaries

### Complete Project Directory Structure

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
│   │   └── useSearch.ts                 # Intégration WASM search
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
├── components.json                      # Config shadcn-vue
├── content.config.ts                    # Collections Content 3
├── nuxt.config.ts                       # Config Nuxt principale
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
└── wrangler.toml                        # Config Cloudflare Pages
```

### Architectural Boundaries

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
Compressed Dump (client download)
      ↓
WASM SQLite (client-side queries)
      ↓
Vue Components (display)
```

### Requirements to Structure Mapping

| FR/Feature | Fichiers Concernés |
|------------|-------------------|
| **FR1-7 Lecture Contenu** | `pages/articles/[slug].vue`, `components/content/*`, `layouts/article.vue` |
| **FR8 ToC** | `components/content/TableOfContents.vue`, `composables/useTableOfContents.ts` |
| **FR9 Progression** | `components/content/ReadingProgress.vue` |
| **FR11-13 Code Blocks** | `components/prose/ProseCode.vue` |
| **FR14-20 Navigation** | `components/layout/*`, `pages/piliers/[pillar].vue` |
| **FR21-24 Badges** | `components/ui/badge/`, `components/content/ArticleMeta.vue` |
| **FR27-36 Recherche/Filtres** | `components/search/*`, `composables/useArticleFilters.ts` |
| **FR37-40 Bilingue** | `i18n/*`, `components/layout/LanguageSwitcher.vue` |
| **FR41-45 SEO/GEO** | `composables/useSeoMeta.ts`, `server/routes/llms.txt.ts` |

### External Integration Points

| Service | Intégration | Fichiers |
|---------|-------------|----------|
| **Plausible Analytics** | Script + runtimeConfig | `nuxt.config.ts`, `app.vue` |
| **Cloudflare Pages** | Build output | `wrangler.toml`, `nuxt.config.ts` |
| **GitHub** | Git push → auto-deploy | `.github/CODEOWNERS` |

