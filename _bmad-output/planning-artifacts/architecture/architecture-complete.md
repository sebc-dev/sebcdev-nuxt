# Architecture Decision Document - sebc.dev

> Document consolidé - Version complète (corrigé)

---

## Table des Matières

1. [Project Context Analysis](#1-project-context-analysis)
2. [Starter Template Evaluation](#2-starter-template-evaluation)
3. [Core Architectural Decisions](#3-core-architectural-decisions)
4. [Implementation Patterns & Consistency Rules](#4-implementation-patterns--consistency-rules)
5. [Project Structure & Boundaries](#5-project-structure--boundaries)

---

# 1. Project Context Analysis

## Requirements Overview

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

## Technical Stack Decisions

| Technologie | Version | Rôle |
|-------------|---------|------|
| **Nuxt** | 4.2.2 | Framework full-stack, SSG mode |
| **Nuxt Content** | 3.10.0+ | CMS fichiers Markdown/MDC |
| **Search Engine** | Pagefind 1.4.0+ | Recherche full-text post-build (~8KB core entry + 36KB API client + chunks) |
| **TailwindCSS** | 4.1.17 | Styling utility-first via @tailwindcss/vite |
| **shadcn-vue** | 2.4.3+ | Composants UI accessibles (Reka UI) |
| **Validation** | Zod 4 | Validation schéma (~5KB gzip / ~10KB min) ou Valibot (~1KB gzip / ~2KB min) + Standard Schema |
| **Hébergement** | Cloudflare Pages | Edge deployment, bande passante illimitée, Early Hints +30% LCP |
| **Runtime** | Node.js 22 LTS | Active LTS "Jod", version stable Cloudflare Pages |
| **Package Manager** | pnpm 10+ | Lifecycle scripts désactivés par défaut (.npmrc requis) |

## Implications Architecturales du Stack

**Nuxt Content 3 + Pagefind :**
- Génération statique complète (SSG) - pas de server functions pour le contenu
- Pagefind indexe le HTML post-build (script `package.json`: `nuxt build && npx pagefind...`)
- Bundle core ~8KB + index chargé par chunks (~40KB/chunk), <300KB total pour 10k pages
- Stemming natif français/anglais via détection automatique attribut HTML `lang`
- Filtrage multi-critères entièrement côté client
- Zéro coût serveur (aligne parfaitement avec budget 0€)
- Mise à jour de l'index = rebuild + redéploiement

**Nuxt 4+ Structure :**
- Nouvelle arborescence `app/` (srcDir) pour composants/pages/layouts
- `server/` et `content/` résolus depuis rootDir
- Meilleure séparation des préoccupations
- TypeScript-first avec inférence améliorée
- Hydratation lazy native (hydrate-on-visible, hydrate-on-idle, hydrate-on-interaction)
- Plus besoin de `future.compatibilityVersion: 4` (défaut)

**TailwindCSS 4 + shadcn-vue (Reka UI) :**
- Intégration via @tailwindcss/vite (module @nuxtjs/tailwindcss@6.14 supporte TW4 mais Vite recommandé)
- Configuration CSS-native avec @theme (remplace tailwind.config.js)
- Mode sombre unique = tokens oklch définis directement sans variantes dark:
- Composants shadcn copiés dans `components/ui/` - ownership total
- shadcn-vue path aliases: `@/` résout vers `app/` en Nuxt 4
- Accessibilité via Reka UI (rebrand Radix Vue février 2025) - validation manuelle WCAG requise
- Command component idéal pour la recherche (⌘K palette)
- Tests a11y en CI avec @axe-core/playwright (tooltips WCAG SC 1.4.13 non couverts)

## Technical Constraints & Dependencies

| Contrainte | Impact Architectural |
|------------|---------------------|
| Budget 0€ | Architecture 100% statique, services gratuits |
| Solo dev | shadcn-vue = composants prêts, moins de code custom |
| Nuxt 4.2 + Content 3 | Stack moderne, SSG, Pagefind post-build, `asSitemapCollection()` |
| TailwindCSS 4 | @tailwindcss/vite recommandé, config CSS-native @theme |
| shadcn-vue + Reka UI | Composants accessibles, path aliases `@/` → `app/` |
| Zod 4 / Valibot | Validation schéma Content 3, Standard Schema natif |
| Cloudflare Pages | Edge global (330+ DC), bande passante illimitée, HTTP/3 manuel |
| Mode sombre unique | Tokens oklch définis directement, pas de toggle |
| Bilingue natif | @nuxtjs/i18n v10.2.1+, strategy prefix_except_default, strictSeo experimental |

## Cross-Cutting Concerns Identified

1. **Internationalization (i18n)** - @nuxtjs/i18n v10.2.1+, prefix_except_default, hreflang auto via useLocaleHead(), strictSeo experimental
2. **Performance Optimization** - SSG, hydratation lazy native Nuxt 4 (hydrate-on-*), nuxt-vitalizer 2.0+, features.inlineStyles, @nuxt/image v2
3. **Accessibility (a11y)** - Reka UI primitives, @axe-core/playwright 4.11+ CI, validation manuelle NVDA/VoiceOver
4. **SEO/GEO Dual Optimization** - llms.txt (adoption précoce, zéro visite AI crawler vérifié), FAQPage Schema, TechArticle, citations +40% visibilité
5. **Content Architecture** - MDC parsing, Zod/Valibot validation (⚠️ @zod/mini déprécié), taxonomy (piliers/catégories/tags)
6. **Client-Side Search** - Pagefind 1.4+ post-build (~8KB + chunks), script package.json, stemming FR/EN natif
7. **Design System** - Tailwind 4 @theme tokens oklch, shadcn-vue Reka UI, dark-first
8. **Build & Runtime** - Node.js 22 LTS, pnpm 10+ (config via package.json ou pnpm-workspace.yaml), nitro preset cloudflare_pages
9. **Sitemap Integration** - @nuxtjs/sitemap v7.5+ avec `asSitemapCollection()` wrapper pour Content v3

---

# 2. Starter Template Evaluation

## Primary Technology Domain

Full-stack Content Platform (Nuxt 4 SSG) - génération statique complète pour Cloudflare Pages.

## Approche Sélectionnée : Initialisation Modulaire

**Rationale :** Étant donné le stack spécifique (Nuxt 4 + Content 3 + TailwindCSS 4 + shadcn-vue), une approche modulaire garantit les versions les plus récentes et un contrôle total sur chaque composant. Aucun template existant ne combine exactement ces technologies dans leurs dernières versions.

## Commande d'Initialisation

```bash
# Créer projet Nuxt 4
pnpm dlx nuxi@latest init sebc-dev
cd sebc-dev && pnpm install

# Modules essentiels
pnpm add @nuxt/content @nuxt/image
pnpm add -D @tailwindcss/vite tailwindcss
pnpm dlx nuxi@latest module add shadcn-nuxt
pnpm dlx nuxi@latest module add i18n

# Validation de schéma
pnpm add zod

# Recherche post-build
pnpm add -D pagefind

# Modules performance Lighthouse
pnpm add -D nuxt-vitalizer

# Génération automatique llms.txt
pnpm add nuxt-llms

# Tests a11y CI
pnpm add -D @axe-core/playwright

# Initialiser shadcn-vue (utilise Reka UI)
pnpm dlx shadcn-vue@latest init
```

## Décisions Architecturales du Starter

**Language & Runtime:**
- TypeScript strict par défaut
- Node.js 22 LTS "Jod" (version stable recommandée par Cloudflare Pages)
- pnpm 10+ comme package manager (⚠️ lifecycle scripts désactivés par défaut - config via `package.json` champ `pnpm` ou `pnpm-workspace.yaml`)

**Styling Solution:**
- TailwindCSS 4.1.17+ via @tailwindcss/vite (⚠️ @nuxtjs/tailwindcss incompatible avec TW4 - v7.0.0-beta.0 en pré-release uniquement)
- Configuration CSS-native avec @theme (remplace tailwind.config.js)
- Tokens sémantiques oklch pour dark-first design

**Build Tooling:**
- Vite comme bundler (intégré Nuxt)
- SSG mode pour génération statique
- Pagefind indexation post-build (hook `nitro:build:public-assets`)
- Cloudflare Pages compatible (preset cloudflare_pages)

**UI Components:**
- shadcn-vue 2.4.3+ avec Reka UI primitives (rebrand de Radix Vue depuis février 2025)
- Composants dans app/components/ui/
- Tests a11y @axe-core/playwright 4.11.0 en CI (couverture WCAG incomplète native)

**Content Management:**
- Nuxt Content 3.10.0+ avec collections
- Fichiers MDC dans content/
- Validation Zod 4 (~5KB gzip / ~10KB min) ou `zod/mini` (~2KB gzip / ~5KB min) + Standard Schema natif
- ⚠️ `@zod/mini` npm package **déprécié** → utiliser `import { z } from 'zod'`

**Search:**
- Pagefind 1.4.0+ post-build (~8KB core entry + 36KB API client vs 500KB+ SQLite WASM)
- Stemming natif FR/EN sans configuration
- Chunks chargés uniquement pour termes recherchés

**Internationalization:**
- @nuxtjs/i18n v10.2.1+
- Strategy: prefix_except_default (/about en, /fr/about fr)
- Propriété `language` obligatoire pour hreflang auto
- Lazy-loading des traductions

**Performance Optimization:**
- Hydratation lazy native Nuxt 4 (hydrate-on-visible, hydrate-on-idle)
- nuxt-vitalizer 2.0.0+ pour optimisation LCP
- features.inlineStyles pour réduction CLS

## Configuration nuxt.config.ts de Base

```typescript
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  // Nuxt 4 par défaut, plus besoin de future.compatibilityVersion: 4
  compatibilityDate: '2025-07-15',

  modules: [
    '@nuxt/content',
    '@nuxt/image',
    '@nuxtjs/i18n',
    '@nuxtjs/sitemap',
    'nuxt-schema-org',
    'nuxt-llms',        // Génération automatique /llms.txt
    'nuxt-vitalizer',
    'shadcn-nuxt',
  ],

  css: ['~/assets/css/main.css'],

  vite: {
    plugins: [tailwindcss()],
  },

  // Optimisations performances Lighthouse
  features: {
    inlineStyles: true, // Remplace experimental.inlineSSRStyles (réduit CLS 0.77 → 0.00)
  },

  i18n: {
    locales: [
      { code: 'en', language: 'en-US', name: 'English' },
      { code: 'fr', language: 'fr-FR', name: 'Français' },
    ],
    defaultLocale: 'en',
    strategy: 'prefix_except_default', // /about (en), /fr/about (fr)
    baseUrl: 'https://sebc.dev',
  },

  shadcn: {
    prefix: '',
    componentDir: './app/components/ui',
  },

  nitro: {
    preset: 'cloudflare_pages',
    autoSubfolderIndex: false,
    prerender: {
      routes: ['/rss.xml'], // Pre-render RSS feed
    },
  },

  // Pagefind: utiliser script package.json plutôt que hook Nitro (plus robuste pour SSG)
  // "build": "nuxt build && npx pagefind --site .output/public"
})
```

## Configuration TailwindCSS 4

```css
/* app/assets/css/main.css */
@import "tailwindcss";

@theme {
  /* Typographie */
  --font-display: "Satoshi", "sans-serif";
  --font-mono: "JetBrains Mono", "monospace";

  /* Palette oklch dark-first */
  --color-primary: oklch(0.92 0.19 114.08);
  --color-secondary: oklch(0.85 0.15 242.32);
  --color-background: oklch(0.13 0.01 264.05);
  --color-foreground: oklch(0.98 0.01 264.05);

  /* Breakpoints personnalisés */
  --breakpoint-3xl: 1920px;
}
```

## Configuration pnpm 10

**Option 1 - package.json (Recommandée) :**
```json
{
  "pnpm": {
    "onlyBuiltDependencies": ["esbuild", "sharp", "pagefind"]
  }
}
```

**Option 2 - pnpm-workspace.yaml :**
```yaml
# pnpm-workspace.yaml - Configuration pnpm 10
packages:
  - '.'

onlyBuiltDependencies:
  - esbuild
  - sharp
  - pagefind
```

**Note importante :** La syntaxe `.npmrc` avec `pnpm.onlyBuiltDependencies[]` est **incorrecte pour pnpm 10**. Utiliser `package.json` (champ `pnpm`) ou `pnpm-workspace.yaml` à la place.

## Arborescence Projet Nuxt 4

```
sebc-dev/
├── app/                          # srcDir (nouveau défaut Nuxt 4)
│   ├── assets/
│   │   └── css/
│   │       └── main.css          # TailwindCSS entry point
│   ├── components/
│   │   └── ui/                   # shadcn-vue (Reka UI)
│   ├── composables/
│   ├── layouts/
│   ├── pages/
│   └── plugins/
│       └── ssr-width.ts
├── content/                      # Résolu depuis rootDir
│   ├── fr/
│   └── en/
├── server/                       # API routes (résolu depuis rootDir)
├── public/                       # Assets statiques
├── nuxt.config.ts
├── pnpm-workspace.yaml           # Configuration pnpm 10 (alternative au champ pnpm dans package.json)
└── package.json
```

## Hydratation Lazy Native (remplace nuxt-delay-hydration)

```vue
<template>
  <!-- Hydrate quand visible dans le viewport -->
  <LazyExpensiveComponent hydrate-on-visible />

  <!-- Hydrate pendant l'idle time du navigateur -->
  <LazyHeavyComponent hydrate-on-idle />

  <!-- Hydrate après un délai fixe (2s) -->
  <LazyDelayedComponent :hydrate-after="2000" />

  <!-- Hydrate sur interaction (hover, click, focus) -->
  <LazyInteractiveComponent hydrate-on-interaction />
</template>
```

## Migration Reka UI

```typescript
// Ancien import (legacy)
import { TooltipRoot, TooltipTrigger } from 'radix-vue'

// Nouveau import (recommandé depuis février 2025)
import { TooltipRoot, TooltipTrigger } from 'reka-ui'
```

```css
/* Variables CSS également renommées */
/* Ancien */
.element { color: var(--radix-tooltip-trigger-color); }

/* Nouveau */
.element { color: var(--reka-tooltip-trigger-color); }
```

## Paramètres Cloudflare Pages

**À désactiver dans le dashboard Cloudflare** (causent problèmes d'hydratation) :
- ❌ Rocket Loader™
- ❌ Mirage (Image Optimization)
- ❌ Email Address Obfuscation
- ❌ Auto-minification (déprécié août 2024 → minifier au build)

**À activer manuellement pour domaines custom** :
- ✅ HTTP/3 (activé par défaut sur `*.pages.dev` uniquement)

**Configuration Build** :
- Framework preset: `Nuxt.js`
- Build command: `pnpm run build`
- Build output directory: `.output/public`
- Node version: `22.16.0` (version stable recommandée)

**Avantages vérifiés (tier gratuit)** :
- Bande passante illimitée (pas de frais d'egress)
- Early Hints activé par défaut → **+30% LCP** pour nouveaux visiteurs

## Versions des Packages (Décembre 2025)

| Package | Version recommandée | Notes |
|---------|-------------------|-------|
| nuxt | 4.2.2 | Version stable actuelle (9 déc. 2025) |
| @nuxt/content | 3.10.0+ | Pleinement compatible Nuxt 4; `asSitemapCollection()` requis |
| tailwindcss | 4.1.17 | Version stable actuelle (6 nov. 2025) |
| @tailwindcss/vite | 4.1.17 | Intégration directe requise (module Nuxt supporte TW4 mais Vite recommandé) |
| shadcn-vue | 2.4.3+ | Utilise Reka UI; ⚠️ path aliases `@/` → `app/` à ajuster |
| reka-ui | Latest | Rebrand de Radix Vue |
| @nuxtjs/i18n | 10.2.1+ | Compatible Nuxt 4; experimental `strictSeo` mode |
| zod | 4.x | ~5KB gzip / ~10KB min; `zod/mini` (~2KB gzip / ~5KB min) disponible |
| pagefind | 1.4.0+ | ~8KB core + chunks (~40KB/chunk); <300KB total pour 10k pages |
| nuxt-vitalizer | 2.0.0+ | DelayHydration component supprimé → macros natives |
| @axe-core/playwright | 4.11.0+ | Tests a11y standard |
| @nuxtjs/sitemap | 7.5.0+ | Utiliser `asSitemapCollection()` pour Content v3 |
| nuxt-llms | latest | Génération automatique /llms.txt avec @nuxt/content ^3.2.0 |
| pnpm | 10.26.2+ | Lifecycle scripts désactivés par défaut (config via package.json ou pnpm-workspace.yaml) |
| node | 22 LTS | Version stable Cloudflare Pages (Node 24 non recommandé pour CF) |

## Alternatives à Considérer

**Recherche** :
- Orama (8.2k+ stars) : Plus de fonctionnalités, API TypeScript-first
- MiniSearch : Plus léger, API simple

**Validation (Standard Schema compatible)** :

| Library | Taille (gzip / min) | Tree-shaking | Meilleur pour |
|---------|---------------------|--------------|---------------|
| **Valibot v1.0** | ~1KB / ~2KB | Excellent | Client-side, builds minimaux |
| Zod 4 (`zod/mini`) | ~2KB / ~5KB | Bon | Équilibre taille/écosystème |
| Zod 4 (complet) | ~5KB / ~10KB | Modéré | Schémas complexes, inférence TS |

**SEO All-in-One** :
- `@nuxtjs/seo` : Bundle sitemap, robots, schema.org, OG images, link checking

**Bundle Analysis** :
- `npx nuxi analyze` : Intégré (vite-bundle-visualizer)
- nuxt-bundle-analysis : GitHub Action pour CI/CD

## Notes Importantes

1. **TailwindCSS 4 intégration** : `@nuxtjs/tailwindcss@6.14.0` supporte TW4, mais `@tailwindcss/vite` est recommandé pour les nouveaux projets TW4 (Config Viewer moins utile avec CSS-first).

2. **`@zod/mini` npm package déprécié** : Utiliser `import { z } from 'zod'` ou `import * as z from 'zod/mini'` (import path, pas package).

3. **nuxt-delay-hydration obsolète** : Hydratation lazy native depuis Nuxt 3.16+ (`hydrate-on-visible`, `hydrate-on-idle`, `hydrate-on-interaction`, `hydrate-on-media-query`, `hydrate-when`, `hydrate-never`).

4. **Reka UI est le nouveau standard** : Rebrand de Radix Vue (février 2025). shadcn-vue utilise Reka UI par défaut.

5. **Node.js 22 LTS recommandé** : Version stable pour Cloudflare Pages. Node 24 existe mais non recommandé pour CF (support souvent en retard).

6. **pnpm 10 sécurité** : Lifecycle scripts désactivés par défaut. Configuration via `package.json` (champ `pnpm`) ou `pnpm-workspace.yaml` (pas `.npmrc`).

7. **llms.txt** : Module `nuxt-llms` génère automatiquement `/llms.txt` avec @nuxt/content ^3.2.0. Remplace le server route personnalisé.

8. **@nuxt/content v3 API** : Changements majeurs par rapport à v2 :
   - ❌ Composants supprimés : `<ContentDoc>`, `<ContentList>`, `<ContentQuery>`
   - ✅ Utiliser `<ContentRenderer>` pour tout le rendu
   - ❌ `queryContent()` (v2) → ✅ `queryCollection()` (v3)
   - Mode document-driven supprimé - créer les pages manuellement
   - Composants prose personnalisés dans `components/prose/`

9. **Reading time** : 200 wpm standard, considérer 180 wpm pour contenu technique avec code.

---

# 3. Core Architectural Decisions

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

---

# 4. Implementation Patterns & Consistency Rules

## Pattern Categories Defined

**15 points de conflit potentiels** identifiés et résolus pour garantir la cohérence entre agents IA.

## Naming Patterns

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

## Structure Patterns

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

## Format Patterns

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

## Communication Patterns

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

**Hydratation Lazy (Nuxt 4 natif):**
```vue
<template>
  <!-- Hydrate quand visible dans le viewport -->
  <LazyTableOfContents hydrate-on-visible />

  <!-- Hydrate pendant l'idle time du navigateur -->
  <LazySearchCommand hydrate-on-idle />

  <!-- Hydrate sur interaction (hover, click, focus) -->
  <LazyArticleComments hydrate-on-interaction />

  <!-- Hydrate après un délai fixe -->
  <LazyNewsletterForm :hydrate-after="2000" />
</template>
```

**Migration Reka UI (depuis Radix Vue):**
```typescript
// ❌ Ancien import (legacy, ne plus utiliser)
import { TooltipRoot, TooltipTrigger } from 'radix-vue'

// ✅ Nouveau import (recommandé depuis février 2025)
import { TooltipRoot, TooltipTrigger } from 'reka-ui'
```

```css
/* Variables CSS également renommées */
/* ❌ Ancien */ .element { color: var(--radix-tooltip-trigger-color); }
/* ✅ Nouveau */ .element { color: var(--reka-tooltip-trigger-color); }
```

## Process Patterns

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
- Validateur: Zod 4 (~5KB gzip / ~10KB min) ou Valibot v1.0 (~1KB gzip / ~2KB min tree-shaken)
- Import direct depuis `zod` (⚠️ `@zod/mini` package **déprécié**, `import { z } from '@nuxt/content'` **à supprimer**)
- Validation au build time via Content 3 collections

```typescript
// content.config.ts
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { asSitemapCollection } from '@nuxtjs/sitemap/content'
import { z } from 'zod'  // ✅ Import direct depuis zod (pas @nuxt/content)

export default defineContentConfig({
  collections: {
    articles: defineCollection(
      asSitemapCollection({  // ✅ Intégration sitemap Content v3
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
    )
  }
})
```

**Intégration Schema.org + Sitemap avec Content 3:**
```typescript
// content.config.ts - Configuration complète
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { asSchemaOrgCollection } from 'nuxt-schema-org/content'
import { asSitemapCollection } from '@nuxtjs/sitemap/content'
import { z } from 'zod'  // ✅ Import direct

export default defineContentConfig({
  collections: {
    articles: defineCollection(
      asSitemapCollection(  // Ordre: sitemap wraps schemaOrg
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
    )
  }
})
```

## Enforcement Guidelines

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

// ❌ Package @zod/mini (déprécié)
import { z } from '@zod/mini'  // Utiliser 'zod' ou 'zod/mini' (import path)

// ❌ Import zod depuis @nuxt/content (sera supprimé)
import { z } from '@nuxt/content'

// ❌ nuxt-delay-hydration (obsolète depuis Nuxt 3.16+)
modules: ['nuxt-delay-hydration']

// ❌ experimental.inlineSSRStyles (renommé)
experimental: { inlineSSRStyles: true }  // Utiliser features.inlineStyles

// ❌ Import depuis radix-vue (rebrandé)
import { ... } from 'radix-vue'  // Utiliser reka-ui

// ❌ Hook Nitro pour Pagefind (timing incertain en SSG)
hooks: { 'nitro:build:public-assets': ... }  // Utiliser script package.json: "nuxt build && npx pagefind..."

// ❌ Composants Content v2 (supprimés dans v3)
<ContentDoc />
<ContentList />
<ContentQuery />
// ✅ Utiliser <ContentRenderer> + queryCollection()

// ❌ queryContent() (Content v2)
const posts = await queryContent('blog').find()
// ✅ queryCollection() (Content v3)
const posts = await queryCollection('blog').all()

// ❌ Répertoire Pagefind ancien
public/_pagefind/  // Utiliser pagefind/ (changé depuis v1.0.0)

// ❌ Configuration pnpm dans .npmrc
pnpm.onlyBuiltDependencies[]=sharp  // Utiliser package.json (champ pnpm) ou pnpm-workspace.yaml
```

---

# 5. Project Structure & Boundaries

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
├── pnpm-workspace.yaml                  # Config pnpm 10 (optionnel, alternative au champ pnpm dans package.json)
├── components.json                      # Config shadcn-vue
├── content.config.ts                    # Collections Content 3
├── nuxt.config.ts                       # Config Nuxt principale
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
└── wrangler.toml                        # Config Cloudflare Pages (optionnel)
```

**Configuration pnpm 10 (dans package.json) :**
```json
{
  "pnpm": {
    "onlyBuiltDependencies": ["esbuild", "sharp", "pagefind"]
  }
}
```

> **Note :** La syntaxe `.npmrc` avec `pnpm.onlyBuiltDependencies[]` est incorrecte pour pnpm 10.

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
           (36KB API client)  (FR/EN stemming)
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
| **Pagefind** | Script post-build | `package.json` (script build: "nuxt build && npx pagefind...") |

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
| `hooks.build:done` Pagefind | ❌ Timing incorrect | Script `package.json` post-build (plus robuste pour SSG) |

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
