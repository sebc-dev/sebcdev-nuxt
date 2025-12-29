# Core Architectural Decisions

## Decision Priority Analysis

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

## Content Architecture

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

## Frontend Architecture

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

## SEO & GEO Implementation

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **llms.txt** | Module nuxt-llms (auto) | Navigation IA efficace, génération automatique avec @nuxt/content ^3.2.0 |
| **Schema Markup** | nuxt-schema-org | TechArticle + FAQPage + BreadcrumbList, Content 3 intégré |
| **Sitemap** | @nuxtjs/sitemap | Official module, hreflang auto |
| **RSS** | Server route generation | Native Content 3 integration |
| **i18n SEO** | useLocaleHead() | Injection auto hreflang avec `language` obligatoire |

**Optimisations GEO (Princeton/Georgia Tech):**
- Citations d'experts : +40% visibilité
- Statistiques incluses : +35-40%
- Sources citées : +30-40%
- Sections FAQ : potentiel citation IA élevé

## Infrastructure & Deployment

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Image Optimization** | @nuxt/image v2 + Cloudflare | AVIF/WebP auto, Early Hints LCP +30% |
| **CI/CD** | Cloudflare Pages direct | Zero config, preview branches, fast |
| **Environment Config** | .env + runtimeConfig + CF fallback | Local dev + prod parity |
| **Tests a11y** | @axe-core/playwright CI | Couverture WCAG shadcn-vue incomplète |

**Cloudflare Pages Avantages (budget 0€):**

| Avantage | Valeur |
|----------|--------|
| Bande passante | **Illimitée** (vs 100GB Netlify/Vercel) |
| Builds/mois | 500 |
| Réseau edge | 330+ datacenters, 95% population <50ms |
| HTTP/3 + Early Hints | Activés automatiquement |

**Deployment Flow:**

```
Git Push → Cloudflare Pages Build → SSG + Pagefind → Edge Deployment
                ↓
         Preview URL (branches)
```

## Modules Nuxt Finaux

```typescript
// ORDRE CRITIQUE: modules SEO AVANT @nuxt/content
modules: [
  '@nuxt/image',
  '@nuxtjs/i18n',
  '@nuxtjs/sitemap',     // SEO - avant @nuxt/content
  'nuxt-schema-org',     // SEO - avant @nuxt/content
  '@nuxt/content',       // ← APRÈS les modules SEO
  'nuxt-llms',           // Génération automatique /llms.txt
  'nuxt-vitalizer',      // Optimisation LCP
  'shadcn-nuxt',
]

// Performance (remplace experimental.inlineSSRStyles)
features: {
  inlineStyles: true,   // CLS 0.77 → 0.00
}

// Cloudflare Pages SSG
nitro: {
  preset: 'cloudflare_pages',
  autoSubfolderIndex: false,
  prerender: {
    routes: ['/rss.xml'],  // Pre-render RSS feed
  },
}

// Route Rules - Cache Cloudflare optimisé
routeRules: {
  // Cache statique agressif pour assets buildés (1 an, immutable)
  '/assets/**': { headers: { 'cache-control': 'public, max-age=31536000, immutable' } },
  // Cache court pour HTML SSG (permet rollbacks rapides)
  '/**': { headers: { 'cache-control': 'public, max-age=60, s-maxage=60' } },
}
```

**Notes importantes:**
- `@nuxtjs/tailwindcss@6.14.0` supporte TW4, mais `@tailwindcss/vite` recommandé pour nouveaux projets
- `nuxt-delay-hydration` obsolète → hydratation lazy native Nuxt 4 (hydrate-on-visible, etc.)
- `experimental.inlineSSRStyles` renommé en `features.inlineStyles`
- `nuxt-vitalizer` 2.0: DelayHydration component supprimé → macros natives
- **Pagefind** : Exécuter via script `package.json` plutôt que hook Nitro (plus robuste pour SSG) :
  ```json
  "scripts": {
    "build": "nuxt build && npx pagefind --site .output/public"
  }
  ```

## Validation Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Schema Validator** | Zod 4 ou Valibot v1 | Standard Schema natif; Valibot ~1KB gzip / ~2KB min, Zod ~5KB gzip / ~10KB min, zod/mini ~2KB gzip / ~5KB min |
| **Import Source** | `import { z } from 'zod'` ou `import * as z from 'zod/mini'` | ⚠️ `import { z } from '@nuxt/content'` est **déprécié** et sera supprimé |
| **Validation Scope** | Build time | Erreurs détectées au build via Content 3 collections |
| **Type Inference** | `z.infer<typeof schema>` | Types TypeScript générés automatiquement |
| **Sitemap Integration** | `asSitemapCollection()` | Wrapper requis pour @nuxtjs/sitemap v7.5+ avec Content v3 |
| **SEO Unified** | `asSeoCollection()` (optionnel) | Combine sitemap + schema.org + OG Image + Robots via `@nuxtjs/seo/content` |

## Search Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Search Engine** | Pagefind 1.4.0+ | ~100KB total (JS+UI+CSS), core ~8KB + chunks (~40KB/chunk); <300KB pour 10k pages |
| **Indexation** | Script `package.json` post-build | Plus robuste que hook Nitro pour SSG |
| **Loading** | Chunks à la demande | Télécharge uniquement chunks contenant termes recherchés |
| **UX** | Command palette | Intégré avec shadcn-vue Command component (⌘K) |

## @nuxt/content v3 API Changes

**Composants supprimés (migration v2 → v3) :**
- ❌ `<ContentDoc />` - supprimé
- ❌ `<ContentList />` - supprimé
- ❌ `<ContentQuery />` - supprimé
- ✅ Utiliser `<ContentRenderer>` pour tout le rendu

**API de requête :**
```typescript
// ❌ ANCIEN (Content v2)
const posts = await queryContent('blog').find()

// ✅ NOUVEAU (Content v3)
const posts = await queryCollection('blog').all()
```

**Autres changements :**
- Mode document-driven supprimé - créer les pages manuellement
- Composants prose personnalisés dans `components/prose/` (non plus `components/content/`)
- Répertoire de sortie Pagefind : `pagefind/` (non plus `_pagefind/`)

## Hydratation Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Méthode** | Hydratation lazy native Nuxt 4 | Remplace nuxt-delay-hydration (obsolète) |
| **Composants lourds** | `hydrate-on-visible` | Hydrate quand visible dans viewport |
| **Composants interactifs** | `hydrate-on-interaction` | Hydrate sur hover/click/focus |
| **Composants différés** | `hydrate-on-idle` | Hydrate pendant idle time navigateur |
