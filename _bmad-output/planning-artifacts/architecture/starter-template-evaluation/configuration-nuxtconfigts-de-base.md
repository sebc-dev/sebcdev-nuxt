# Configuration nuxt.config.ts de Base

```typescript
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  // Nuxt 4 par défaut, plus besoin de future.compatibilityVersion: 4
  compatibilityDate: '2025-07-15',

  // ORDRE CRITIQUE: @nuxtjs/seo AVANT @nuxt/content
  modules: [
    '@nuxt/image',
    '@nuxtjs/i18n',
    '@nuxtjs/seo',         // Suite SEO complète - AVANT @nuxt/content
    '@nuxt/content',       // APRÈS @nuxtjs/seo
    'nuxt-llms',           // Génération automatique /llms.txt
    'nuxt-vitalizer',
    'shadcn-nuxt',
  ],

  css: ['~/assets/css/main.css'],

  vite: {
    plugins: [tailwindcss()],
  },

  // Configuration site (requise par @nuxtjs/seo)
  site: {
    url: 'https://sebc.dev',
    name: 'sebc.dev',
    description: 'Blog technique sur le développement web',
    defaultLocale: 'en',
  },

  // Configuration OG Image (inclus dans @nuxtjs/seo)
  ogImage: {
    zeroRuntime: true,  // ESSENTIEL pour SSG pur
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
    prerender: {
      autoSubfolderIndex: false, // Évite les doubles redirects Cloudflare
      crawlLinks: true,
      routes: ['/rss.xml'], // Pre-render RSS feed
    },
  },

  routeRules: {
    // Cache statique agressif pour assets buildés par Nuxt (1 an, immutable)
    '/_nuxt/**': { headers: { 'cache-control': 'public, max-age=31536000, immutable' } },
    // Cache court pour HTML SSG (permet rollbacks rapides)
    '/**': { headers: { 'cache-control': 'public, max-age=0, must-revalidate' } },
  },

  // MiniSearch: index généré via script postgenerate
  // "scripts": { "generate": "nuxt generate", "postgenerate": "node scripts/generate-search-index.mjs" }
})
```
