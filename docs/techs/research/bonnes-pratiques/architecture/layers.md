# Nuxt 4 Layers Architecture: Complete Implementation Guide

**Nuxt 4's layers system enables powerful modular architecture for multilingual blogs with e-commerce expansion**, providing configuration inheritance, component sharing, and feature isolation without the complexity of full module development. The key shift in Nuxt 4 is the new `app/` directory as the default `srcDir`, affecting how layers organize their code. For your specific stack—Nuxt Content 3.x, i18n v10, TailwindCSS 4.x, and Cloudflare Pages SSG—layers offer an elegant path from blog to marketplace while maintaining zero-cost hosting.

---

## Nuxt 4 fundamentally changes layer directory structure

Nuxt 4 introduces `app/` as the default source directory, meaning components, pages, and composables now live inside `app/` rather than at the project root. Layers must mirror this structure:

```
project/
├── app/                      # Nuxt 4 srcDir (replaces root placement)
│   ├── components/
│   ├── composables/
│   ├── pages/
│   └── app.vue
├── layers/                   # Auto-scanned (lives at rootDir, not srcDir)
│   ├── shared/
│   │   ├── app/              # Layer's srcDir
│   │   │   ├── components/
│   │   │   └── composables/
│   │   └── nuxt.config.ts
│   └── blog/
│       ├── app/
│       │   ├── components/
│       │   └── pages/blog/
│       ├── content/
│       └── nuxt.config.ts
├── content/
├── server/                   # Server stays at rootDir
└── nuxt.config.ts
```

Layers in `~/layers/` are **auto-registered** since Nuxt 3.12, requiring no explicit `extends` configuration. Named aliases (`#layers/shared`, `#layers/blog`) are automatically created in v3.16+, enabling explicit cross-layer imports. The resolution priority flows from project files (highest) → auto-scanned layers (alphabetical, Z > A) → extended layers (first entry wins). Control ordering by prefixing directories: `1.shared/`, `2.blog/`, `3.marketplace/`.

**Critical Nuxt 4 change:** Module loading order was corrected—layer modules now load first (deeper layers first), then project modules. This means your project can properly override layer module behavior.

---

## Base layer design forms the foundation

A well-designed base layer contains UI primitives, utilities, and shared configuration with zero business logic:

```typescript
// layers/shared/nuxt.config.ts
import { fileURLToPath } from 'node:url'
import { dirname, join } from 'node:path'

const currentDir = dirname(fileURLToPath(import.meta.url))

export default defineNuxtConfig({
  $meta: { name: 'shared' },
  
  css: [join(currentDir, './app/assets/css/tailwind.css')],
  
  modules: ['@nuxtjs/i18n', 'shadcn-nuxt'],
  
  build: {
    transpile: ['reka-ui', 'vee-validate']  // Required for shadcn-vue in layers
  },
  
  shadcn: {
    prefix: '',
    componentDir: join(currentDir, './app/components/ui')
  }
})
```

**Always use absolute paths** with `join(currentDir, ...)` in layer configs—relative paths resolve incorrectly when consumed by other projects. Feature layers then extend shared functionality while maintaining isolation:

```typescript
// layers/blog/nuxt.config.ts
export default defineNuxtConfig({
  $meta: { name: 'blog' }
  // No need to explicitly extend shared—auto-registration handles it
})
```

Enforce architectural boundaries with `eslint-plugin-nuxt-layers`:

```javascript
// eslint.config.mjs
export default [{
  plugins: { 'nuxt-layers': nuxtLayers },
  rules: {
    'nuxt-layers/layer-boundaries': ['error', {
      layers: {
        shared: [],                    // Foundation: no dependencies
        blog: ['shared'],              // Can only import from shared
        marketplace: ['shared'],       // Isolated from blog
      }
    }]
  }
}]
```

---

## Nuxt Content 3.x requires centralized collection configuration

Content 3.x stores all content in a **single SQLite database** regardless of layer structure. Define all collections in the root `content.config.ts`, using absolute paths to reference layer content:

```typescript
// content.config.ts (root project)
import { defineCollection, defineContentConfig, z } from '@nuxt/content'
import path from 'path'

// Shared schemas for reuse
const seoSchema = z.object({
  ogImage: z.string().optional(),
  description: z.string()
})

export default defineContentConfig({
  collections: {
    // Root content
    pages: defineCollection({
      type: 'page',
      source: 'pages/**/*.md'
    }),
    
    // Blog layer content
    blog: defineCollection({
      type: 'page',
      source: {
        include: '**/*.md',
        prefix: '/blog',
        cwd: path.resolve(__dirname, 'layers/blog/content/blog')
      },
      schema: seoSchema.extend({
        date: z.date(),
        tags: z.array(z.string()),
        author: z.string()
      })
    }),
    
    // Future marketplace layer
    products: defineCollection({
      type: 'data',
      source: {
        include: '**/*.yml',
        cwd: path.resolve(__dirname, 'layers/marketplace/content/products')
      },
      schema: z.object({
        name: z.string(),
        price: z.number(),
        category: z.string()
      })
    })
  }
})
```

**MDC components** (Markdown Components) must be placed in `components/content/` within each layer. In Content 3.x, these are **no longer automatically global**—register them explicitly:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  components: [
    { path: '~/components/content', global: true },
    { path: '#layers/shared/app/components/content', global: true }
  ]
})
```

One critical limitation: **a document can only belong to one collection**. Avoid overlapping source patterns—use explicit `exclude` rules when needed.

---

## i18n v10 merges locale files across layers intelligently

Organize locale files in each layer's `i18n/locales/` directory. The merging follows layer priority—project files override layer files for matching keys:

```
layers/shared/i18n/locales/
├── en.json    →  {"common": {"submit": "Submit"}, "nav": {"home": "Home"}}
└── fr.json

layers/blog/i18n/locales/
├── en.json    →  {"blog": {"title": "Blog"}}
└── fr.json

i18n/locales/           # Project (highest priority)
├── en.json    →  {"nav": {"home": "Homepage"}}  # Overrides shared
└── fr.json
```

Configure i18n in the base layer, with locale additions in feature layers:

```typescript
// layers/shared/nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/i18n'],
  i18n: {
    defaultLocale: 'en',
    strategy: 'prefix_except_default',
    locales: [
      { code: 'en', file: 'en.json', name: 'English' },
      { code: 'fr', file: 'fr.json', name: 'Français' }
    ]
  }
})
```

For **language-separated content collections**, create per-locale collections and query dynamically:

```typescript
// content.config.ts
export default defineContentConfig({
  collections: {
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'en/**/*.md', prefix: '' }
    }),
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/**/*.md', prefix: '' }
    })
  }
})
```

```vue
<script setup>
const { locale } = useI18n()
const { data } = await useAsyncData('page', async () => {
  const collection = `blog_${locale.value}` as keyof Collections
  return await queryCollection(collection).path(route.path).first()
}, { watch: [locale] })
</script>
```

---

## TailwindCSS 4.x demands explicit layer source scanning

TailwindCSS 4's CSS-first configuration auto-detects classes via Vite's module graph, but **layer files may not be included automatically**. Use the `@source` directive to explicitly scan layer directories:

```css
/* app/assets/css/tailwind.css */
@import "tailwindcss";

/* Explicitly include layer content */
@source "../../../layers/shared/app/components";
@source "../../../layers/blog/app/components";

@theme {
  /* Design tokens available across all layers */
  --color-primary: oklch(0.72 0.11 178);
  --color-secondary: oklch(0.85 0.08 220);
  --font-display: "Inter", sans-serif;
  
  /* shadcn-vue semantic colors */
  --color-background: hsl(0 0% 100%);
  --color-foreground: hsl(222.2 84% 4.9%);
  --color-card: hsl(0 0% 100%);
  --color-border: hsl(214.3 31.8% 91.4%);
  --color-ring: hsl(222.2 84% 4.9%);
  --radius-lg: 0.5rem;
}

/* Dark mode using @theme inline for CSS variable references */
@theme inline {
  --color-background: var(--theme-bg);
  --color-foreground: var(--theme-fg);
}

@layer base {
  :root { --theme-bg: white; --theme-fg: black; }
  .dark { --theme-bg: hsl(222.2 84% 4.9%); --theme-fg: white; }
}
```

For **shadcn-vue in layers**, the critical configuration includes transpiling Reka UI and using absolute paths:

```typescript
// layers/shared/nuxt.config.ts
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  vite: {
    plugins: [tailwindcss()]
  },
  build: {
    transpile: ['reka-ui', 'vee-validate']  // Prevents hydration errors
  },
  shadcn: {
    componentDir: join(currentDir, './app/components/ui')
  }
})
```

---

## Cloudflare Pages deployment requires specific SSG configuration

For static generation with layers on Cloudflare Pages, configure Nitro's prerendering with Cloudflare-specific settings:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  compatibilityDate: '2024-09-19',
  
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      crawlLinks: true,
      autoSubfolderIndex: false,  // Required for Cloudflare route matching
      failOnError: false,
      routes: ['/', '/blog', '/fr', '/fr/blog']  // Explicitly include i18n routes
    },
    cloudflare: {
      deployConfig: true
    }
  },
  
  // Generate all content routes dynamically
  hooks: {
    'nitro:config'(nitroConfig) {
      // Add dynamic routes from content collections
    }
  }
})
```

**Build commands** for Cloudflare Pages:
- Build: `npx nuxi generate`
- Output directory: `dist/`
- Deploy: `npx wrangler pages deploy dist/`

Bundle optimization for layers:

```typescript
vite: {
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'content': ['@nuxt/content'],
          'i18n': ['@nuxtjs/i18n']
        }
      }
    }
  }
}
```

---

## Layers versus modules: a decision framework

| Scenario | Use Layers | Use Modules |
|----------|-----------|-------------|
| Sharing components/pages across apps | ✅ | |
| Domain-driven feature isolation | ✅ | |
| Theme/design system distribution | ✅ | |
| Build-time Vite/Nitro customization | | ✅ |
| Third-party service integration | | ✅ |
| Deep hook manipulation | | ✅ |

**For your multilingual blog → marketplace expansion**, layers are ideal. Start with:

- `layers/shared/`: shadcn-vue components, composables, i18n base config, Tailwind theme
- `layers/blog/`: Blog pages, MDC components, blog content directory
- `layers/marketplace/` (future): Product pages, cart composables, checkout flow

---

## Avoiding common pitfalls saves debugging time

The most frequent layer issues and their solutions:

**Silent component overrides**: Same-named components in multiple layers silently override based on priority. Use unique prefixes (`BlogCard.vue`, `MarketCard.vue`) or explicit naming conventions.

**Path resolution failures**: Using `~/` in layer code resolves to the consuming project, not the layer. Always use relative paths or `#layers/name` aliases:

```typescript
// ❌ Breaks when consumed
import { util } from '~/utils/helper'

// ✅ Works correctly
import { util } from '../utils/helper'
import { util } from '#layers/shared/utils/helper'
```

**Missing layer dependencies**: Packages imported by a layer must be in its `dependencies`, not `devDependencies`. When publishing layers as npm packages, consumers shouldn't need to install layer dependencies separately.

**Runtime config undefined**: A known bug causes `runtimeConfig` values without defaults to be undefined in production when using npm package layers. Always provide defaults:

```typescript
runtimeConfig: {
  apiKey: process.env.API_KEY || ''  // Provide fallback
}
```

---

## Conclusion

Nuxt 4 layers provide an elegant modular architecture for your multilingual blog with future marketplace expansion. The key patterns to implement are: centralized content configuration with per-layer source paths, shared TailwindCSS themes with explicit `@source` directives for layer scanning, i18n configuration inheritance from a base layer, and strict boundary enforcement via ESLint. By structuring layers as `shared` → `blog` → `marketplace`, you create isolated feature domains that share UI primitives without coupling business logic. For Cloudflare Pages deployment, ensure `autoSubfolderIndex: false` and explicit i18n route prerendering. The architecture scales cleanly—adding a marketplace layer requires only creating the new layer directory and extending `content.config.ts` with product collections, without modifying existing blog functionality.