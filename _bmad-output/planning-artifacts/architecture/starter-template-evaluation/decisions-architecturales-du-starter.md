# Décisions Architecturales du Starter

**Language & Runtime:**
- TypeScript strict par défaut
- Node.js 22 LTS "Jod" (version stable recommandée par Cloudflare Pages)
- pnpm 10+ comme package manager (⚠️ lifecycle scripts désactivés par défaut - config via `package.json` champ `pnpm` ou `pnpm-workspace.yaml`)

**Styling Solution:**
- TailwindCSS 4.1.17+ via @tailwindcss/vite (⚠️ @nuxtjs/tailwindcss@6.14.0 supporte TW4, mais @tailwindcss/vite recommandé pour nouveaux projets CSS-first)
- Configuration CSS-native avec @theme (remplace tailwind.config.js)
- Tokens sémantiques oklch pour dark-first design

**Build Tooling:**
- Vite comme bundler (intégré Nuxt)
- SSG mode pour génération statique
- MiniSearch index généré post-build (script `scripts/generate-search-index.mjs`)
- Cloudflare Pages compatible (preset cloudflare_pages)

**UI Components:**
- shadcn-vue 2.4.3+ avec Reka UI primitives (rebrand de Radix Vue depuis février 2025)
- Composants dans app/components/ui/
- Tests a11y @axe-core/playwright 4.11.0 en CI (couverture WCAG incomplète native)
- Tests a11y unitaires vitest-axe pour validation composants isolés

**Content Management:**
- Nuxt Content 3.10.0+ avec collections
- Fichiers MDC dans content/
- Validation Zod 4 (~5KB gzip / ~10KB min) ou `zod/mini` (~2KB gzip / ~5KB min) + Standard Schema natif
- ⚠️ Import Zod 4 : `import { z } from 'zod/v4'` (API moderne avec top-level validators)
- ⚠️ `@zod/mini` npm package **déprécié** → utiliser `import * as z from 'zod/mini'` (import path)
- Migration automatique : `npx zod-v3-to-v4 path/to/tsconfig.json`

**Search:**
- MiniSearch 7.x (~7KB minified, zéro dépendance)
- Index JSON pré-généré au build time
- Boosting par champ, fuzzy search, prefix search natifs
- Stemming FR via option `stemmer` (snowball-stemmers)

**Internationalization:**
- @nuxtjs/i18n v10.2.1+
- Strategy: prefix_except_default (/about en, /fr/about fr)
- Propriété `language` obligatoire pour hreflang auto
- Lazy-loading des traductions

**Performance Optimization:**
- Hydratation lazy native Nuxt 4 (hydrate-on-visible, hydrate-on-idle)
- nuxt-vitalizer 2.0.0+ pour optimisation LCP
- features.inlineStyles pour réduction CLS
