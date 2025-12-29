# Starter Template Evaluation

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
- Node.js 22.16.0 (version stable recommandée par Cloudflare Pages)
- pnpm 10+ comme package manager (⚠️ lifecycle scripts désactivés par défaut - config via pnpm-workspace.yaml)

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
- Validation Zod 4 (8-10KB) ou `zod/mini` (3-5KB) + Standard Schema natif
- ⚠️ `@zod/mini` npm package **déprécié** → utiliser `import { z } from 'zod'`

**Search:**
- Pagefind 1.4.0+ post-build (36KB core vs 500KB+ SQLite WASM)
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

  // Hook Pagefind post-build (après génération des assets publics)
  hooks: {
    'nitro:build:public-assets': async () => {
      const { exec } = await import('child_process')
      exec('npx pagefind --site .output/public')
    },
  },
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

```yaml
# pnpm-workspace.yaml - Configuration pnpm 10
# Lifecycle scripts désactivés par défaut pour sécurité
packages:
  - '.'

onlyBuiltDependencies:
  - esbuild
  - sharp
  - pagefind
```

**Note importante :** La syntaxe `.npmrc` avec `pnpm.onlyBuiltDependencies[]` est **incorrecte pour pnpm 10**. Utiliser `pnpm-workspace.yaml` à la place.

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
├── .npmrc                        # Configuration pnpm 10
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
| zod | 4.x | 8-10KB gzippé; `zod/mini` (3-5KB) disponible |
| pagefind | 1.4.0+ | ~8KB core + chunks (~40KB/chunk); <300KB total pour 10k pages |
| nuxt-vitalizer | 2.0.0+ | DelayHydration component supprimé → macros natives |
| @axe-core/playwright | 4.11.0+ | Tests a11y standard |
| @nuxtjs/sitemap | 7.5.0+ | Utiliser `asSitemapCollection()` pour Content v3 |
| nuxt-llms | latest | Génération automatique /llms.txt avec @nuxt/content ^3.2.0 |
| pnpm | 10.26.2+ | Lifecycle scripts désactivés par défaut (config via pnpm-workspace.yaml) |
| node | 22.16.0 | Version stable Cloudflare Pages (Node 24 LTS non recommandé pour CF) |

## Alternatives à Considérer

**Recherche** :
- Orama (8.2k+ stars) : Plus de fonctionnalités, API TypeScript-first
- MiniSearch : Plus léger, API simple

**Validation (Standard Schema compatible)** :

| Library | Taille gzippée | Tree-shaking | Meilleur pour |
|---------|----------------|--------------|---------------|
| **Valibot v1.0** | 0.7-2KB | Excellent | Client-side, builds minimaux |
| Zod 4 (`zod/mini`) | 3-5KB | Bon | Équilibre taille/écosystème |
| Zod 4 (complet) | 8-10KB | Modéré | Schémas complexes, inférence TS |

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

5. **Node.js 22.16.0 recommandé** : Version stable pour Cloudflare Pages. Node 24 LTS existe mais non recommandé pour CF.

6. **pnpm 10 sécurité** : Lifecycle scripts désactivés par défaut. Configuration `pnpm-workspace.yaml` requise (pas `.npmrc`).

7. **llms.txt** : Module `nuxt-llms` génère automatiquement `/llms.txt` avec @nuxt/content ^3.2.0. Remplace le server route personnalisé.

8. **@nuxt/content v3 API** : Changements majeurs par rapport à v2 :
   - ❌ Composants supprimés : `<ContentDoc>`, `<ContentList>`, `<ContentQuery>`
   - ✅ Utiliser `<ContentRenderer>` pour tout le rendu
   - ❌ `queryContent()` (v2) → ✅ `queryCollection()` (v3)
   - Mode document-driven supprimé - créer les pages manuellement
   - Composants prose personnalisés dans `components/mdc/`

9. **Reading time** : 200 wpm standard, considérer 180 wpm pour contenu technique avec code.

**Note:** L'initialisation du projet avec ces commandes sera la première story d'implémentation.