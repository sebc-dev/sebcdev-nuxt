# Nuxt 4 and Vue 3 naming conventions and architecture guide

**The December 2025 Nuxt 4 ecosystem establishes clear conventions: PascalCase for components, `use` prefix for composables, kebab-case for page files, and the new `app/` srcDir structure that separates concerns for better performance and type safety.** This architecture change—moving source code from the root to `app/`—delivers significant startup improvements by reducing file system watchers scanning `node_modules/` and `.git/`. Combined with TailwindCSS 4's CSS-native configuration and shadcn-vue's component organization, these conventions create a cohesive development experience for SSG deployments on platforms like Cloudflare Pages.

## Component naming follows strict multi-word PascalCase rules

Vue 3's official style guide mandates **multi-word component names** to prevent collisions with current and future HTML elements. This rule applies to all user-defined components except the root `App` component and Vue built-ins like `<transition>`.

**File naming conventions** support either PascalCase or kebab-case, but PascalCase is preferred because it aligns with IDE autocompletion and JavaScript conventions:

```
app/components/
├── BaseButton.vue          # ✅ PascalCase (preferred)
├── base-button.vue         # ✅ kebab-case (acceptable)
├── basebutton.vue          # ❌ lowercase (avoid)
├── baseButton.vue          # ❌ camelCase (avoid)
```

**Template usage** differs by context. In Single-File Components and string templates, use PascalCase (`<MyComponent />`). In DOM templates, HTML's case insensitivity requires kebab-case (`<my-component></my-component>`) with explicit closing tags.

Nuxt 4 auto-imports components with **path-based naming by default**—a component at `components/base/Button.vue` becomes `<BaseButton />`. To disable this prefix behavior and use bare component names:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  components: [
    { path: '~/components', pathPrefix: false }
  ]
})
```

**Base component prefixes** should consistently use `Base`, `App`, or `V` for presentational components. Tightly coupled components include their parent's name: `TodoList.vue`, `TodoListItem.vue`, `TodoListItemButton.vue`. Single-instance components use `The` prefix: `TheHeader.vue`, `TheSidebar.vue`.

## Composables require the `use` prefix and camelCase naming

Vue 3 establishes a clear convention: **composable functions start with `use` and use camelCase**. This pattern signals that a function leverages Vue's Composition API and may contain reactive state:

```typescript
// app/composables/useMouse.ts
export function useMouse() {
  const x = ref(0)
  const y = ref(0)
  
  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))
  
  return { x, y }  // Return object with refs for destructuring
}
```

Nuxt 4 **auto-imports only top-level files** from `app/composables/` by default. Nested directories require explicit configuration:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  imports: {
    dirs: [
      '~/composables',
      '~/composables/**'  // Enable nested scanning
    ]
  }
})
```

**Server vs client composables** use file suffixes when environment-specific rendering is needed. Components support `.client.vue` and `.server.vue` suffixes, but composables typically use runtime checks with `process.client` or `process.server` guards instead.

Anthony Fu's VueUse guidelines recommend several patterns: use **options objects** for composable arguments (enabling future extension), prefer `shallowRef` over `ref` for large objects, and always use named exports for better tree-shaking. TypeScript naming follows `Use{Name}Options` for input types and `Use{Name}Return` for return types.

## Pages follow kebab-case with bracket syntax for dynamic routes

Nuxt 4's file-based routing uses **kebab-case for page files** with a bracket syntax for dynamic parameters that differs from Nuxt 3's colon-based approach:

| Pattern | URL | Route Name |
|---------|-----|------------|
| `pages/index.vue` | `/` | `index` |
| `pages/about.vue` | `/about` | `about` |
| `pages/blog/[slug].vue` | `/blog/:slug` | `blog-slug` |
| `pages/[[optional]].vue` | `/` or `/anything` | Optional param |
| `pages/[...slug].vue` | Any unmatched path | Catch-all |

**Dynamic route syntax** uses single brackets for required parameters (`[id]`), double brackets for optional parameters (`[[id]]`), and spread syntax for catch-all routes (`[...slug]`). Mixed parameters combine static text with dynamics: `pages/user-[name]-[id].vue` matches `/user-john-123`.

Route groups using parentheses—`pages/(marketing)/about.vue`—are ignored in URLs, enabling logical organization without affecting routing.

## i18n v10 integrates through definePageMeta for localized paths

The **@nuxtjs/i18n v10** module generates route names with locale suffixes: `about___en`, `about___fr`. The `prefix_except_default` strategy is recommended for most SSG sites, adding locale prefixes to all routes except the default language:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  i18n: {
    strategy: 'prefix_except_default',
    defaultLocale: 'en',
    locales: [
      { code: 'en', language: 'en-US' },
      { code: 'fr', language: 'fr-FR' }
    ]
  }
})
```

**Custom localized paths** use `definePageMeta` (replacing the deprecated `defineI18nRoute`):

```vue
<!-- pages/about.vue -->
<script setup>
definePageMeta({
  i18n: {
    paths: {
      en: '/about-us',
      fr: '/a-propos'
    }
  }
})
</script>
```

For dynamic route parameter translation, use the `useSetI18nParams` composable to map parameter values per locale, enabling URLs like `/products/red-mug` (English) and `/nl/products/rode-mok` (Dutch).

## Atomic Design integrates cleanly with Nuxt 4 auto-imports

The Atomic Design pattern maps naturally to Nuxt's structure: **atoms** become base components, **molecules** compose atoms, **organisms** are self-contained complex components, **templates** map to `layouts/`, and **pages** remain in `pages/`:

```
app/
├── components/
│   ├── atoms/
│   │   ├── BaseButton.vue
│   │   └── BaseInput.vue
│   ├── molecules/
│   │   ├── SearchForm.vue
│   │   └── UserCard.vue
│   ├── organisms/
│   │   ├── TheHeader.vue
│   │   └── ProductGrid.vue
│   └── ui/                    # shadcn-vue components
│       ├── button/
│       └── dialog/
├── layouts/
└── pages/
```

Configure auto-imports to work seamlessly with this structure by **disabling path prefixes** for atomic folders:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  components: [
    { path: '~/components/ui', pathPrefix: false },
    { path: '~/components/atoms', pathPrefix: false },
    { path: '~/components/molecules', pathPrefix: false },
    { path: '~/components/organisms', pathPrefix: false },
    '~/components'
  ]
})
```

**shadcn-vue 2.4+** (now powered by Reka UI) installs to `components/ui/` by default. The `shadcn-nuxt` module handles auto-imports automatically. Configure the prefix option based on your naming strategy—use `prefix: ''` for direct names like `<Button />` or `prefix: 'Ui'` to avoid conflicts with atomic components.

## The app/ directory structure defines Nuxt 4 architecture

Nuxt 4's most significant architectural change moves source code from the root to `app/` as the default `srcDir`. Daniel Roe explains this delivers **measurable performance improvements**: file system watchers no longer scan `node_modules/` and `.git/`, reducing startup delays especially on Windows and Linux.

The standard Nuxt 4 structure separates concerns by context:

```
project/
├── app/                       # srcDir - Vue application code
│   ├── assets/
│   ├── components/
│   ├── composables/
│   ├── layouts/
│   ├── middleware/
│   ├── pages/
│   ├── plugins/
│   ├── utils/
│   └── app.vue
├── content/                   # rootDir - Nuxt Content (v3)
├── server/                    # rootDir - Nitro server
│   ├── api/
│   ├── middleware/
│   └── utils/
├── shared/                    # rootDir - Shared app+server code
│   ├── types/
│   └── utils/
├── public/                    # rootDir - Static assets
└── nuxt.config.ts
```

**Path alias mappings** reflect this structure:

| Alias | Points To | Use Case |
|-------|-----------|----------|
| `~/` or `@/` | `app/` | Components, composables, pages |
| `~~/` or `@@/` | Project root | Access server/, content/ |
| `#shared` | `shared/` | Shared utilities and types |

The separation enables **distinct TypeScript configurations** per context (`.nuxt/tsconfig.app.json`, `.nuxt/tsconfig.server.json`), providing accurate IDE autocompletion for both environments.

## TailwindCSS 4 eliminates JavaScript configuration

TailwindCSS 4 introduces **CSS-native configuration** using the `@theme` directive, making `tailwind.config.js` optional:

```css
/* app/assets/css/main.css */
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.62 0.21 259);
  --color-secondary: oklch(0.70 0.17 162);
  --font-sans: "Inter", ui-sans-serif, system-ui, sans-serif;
  --breakpoint-3xl: 120rem;
}
```

Theme variables use **semantic namespaces**: `--color-*` for color utilities, `--font-*` for font families, `--spacing-*` for spacing, `--radius-*` for border radius, and `--breakpoint-*` for responsive variants.

Install with the Vite plugin for Nuxt 4:

```typescript
// nuxt.config.ts
import tailwindcss from "@tailwindcss/vite"

export default defineNuxtConfig({
  css: ['~/assets/css/main.css'],
  vite: {
    plugins: [tailwindcss()]
  }
})
```

**Dark mode** uses CSS custom properties with an inline theme reference:

```css
@layer base {
  :root { --background: hsl(0 0% 100%); }
  .dark { --background: hsl(0 0% 3.9%); }
}

@theme inline {
  --color-background: var(--background);
}
```

## Nuxt Content 3 requires collections and D1 for Cloudflare

Nuxt Content 3.10+ introduces a **SQL-based storage system** with typed collections, replacing the previous file-based approach. Configuration moves to `content.config.ts`:

```typescript
// content.config.ts
import { defineContentConfig, defineCollection, z } from '@nuxt/content'

export default defineContentConfig({
  collections: {
    docs: defineCollection({
      type: 'page',
      source: 'docs/**/*.md'
    }),
    blog: defineCollection({
      type: 'page',
      source: 'blog/*.md',
      schema: z.object({
        tags: z.array(z.string()),
        date: z.date()
      })
    })
  }
})
```

**File naming conventions** use numeric prefixes for ordering (`1.introduction.md`, `2.setup.md`) separated by dots. Navigation configuration uses `.navigation.yml` (replacing v2's `_dir.yml`).

For **Cloudflare Pages SSG deployment**, a D1 database binding is required:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/content'],
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      crawlLinks: true,
      routes: ['/']
    }
  },
  routeRules: {
    '/**': { prerender: true }
  }
})
```

Create a D1 database in Cloudflare Dashboard and bind it with the name `DB`. Disable Rocket Loader, Mirage, and Email Address Obfuscation in Cloudflare settings to prevent hydration issues.

## Conclusion

Nuxt 4's architecture establishes a mature, performance-oriented foundation where **conventions minimize configuration**. The `app/` directory separation delivers tangible startup improvements while enabling proper TypeScript context isolation between client and server code. Component naming follows Vue 3's battle-tested PascalCase multi-word rules, while composables consistently use the `use` prefix pattern established by VueUse.

The integration points work harmoniously: TailwindCSS 4's CSS-native `@theme` directive eliminates JavaScript configuration overhead, shadcn-vue's `components/ui/` structure coexists naturally with Atomic Design folders, and @nuxtjs/i18n v10's `definePageMeta` integration keeps localization logic close to components. For SSG deployments, Nuxt Content 3's collections system pairs with Cloudflare's D1 database to deliver type-safe content at the edge.

The key insight for project setup: **let Nuxt's auto-imports handle discovery while using explicit configuration only to disable path prefixes** for cleaner component names. This approach—embracing conventions rather than fighting them—unlocks the full benefits of the Nuxt 4 developer experience.