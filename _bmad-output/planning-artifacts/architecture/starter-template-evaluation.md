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
pnpm add @nuxt/content
pnpm add -D @tailwindcss/vite tailwindcss shadcn-nuxt
pnpm dlx nuxi@latest module add i18n

# Validation de schéma
pnpm add zod

# Initialiser shadcn-vue
pnpm dlx shadcn-vue@latest init
```

## Décisions Architecturales du Starter

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

## Configuration nuxt.config.ts de Base

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

## Arborescence Projet Nuxt 4

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
