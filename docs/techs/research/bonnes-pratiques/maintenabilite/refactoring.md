# Vue 3 / Nuxt 4 refactoring patterns for maintainable code

**For a multilingual SSG blog on Cloudflare Pages using Nuxt 4.2.x, three refactoring patterns dramatically improve maintainability: extracting composables, choosing the right composition strategy, and optimizing lazy loading.** This guide provides decision frameworks and TypeScript patterns specifically tuned for the `<script setup lang="ts">` workflow with shadcn-vue/Reka UI components. The patterns below reflect December 2025 best practices from official Vue 3 and Nuxt 4 documentation, accounting for breaking changes in Nuxt 4's new `app/` directory structure.

---

## Extract composable pattern: when and how to refactor

A composable is warranted when logic involves **stateful reactivity plus lifecycle hooks** and appears in three or more components. The canonical example is tracking mouse position—combining `ref()` state, event listeners, and cleanup in `onUnmounted()`. Stateless utility functions (date formatting, string manipulation) should remain plain functions in `app/utils/`, not composables.

**Extraction criteria checklist:**
- Logic manages reactive state that changes over time
- Lifecycle hooks are required (onMounted, onUnmounted, watch)
- Same pattern appears in 3+ components
- Async data fetching with loading/error states
- DOM event listener setup/cleanup patterns

**Anti-pattern to avoid:** Creating composables for trivial one-liners or simple computed values. Over-abstraction fragments code without adding value. A `useFormatDate()` that just calls `toLocaleDateString()` adds indirection without benefit.

### Naming and file organization in Nuxt 4

Nuxt 4's default `srcDir` is now `app/`, changing the directory structure:

```
app/
├── composables/          # Auto-imported composables
│   ├── index.ts          # Re-exports for nested composables
│   ├── useAuth.ts        # Named export pattern (recommended)
│   └── blog/
│       └── useArticle.ts # Nested - requires re-export
├── utils/                # Auto-imported utilities (stateless)
└── components/
```

The `use*` prefix is mandatory for composable naming—`useCounter`, `useFetch`, `useI18n`. Nuxt auto-imports only **top-level files** in `composables/`. For nested directories, create an `index.ts` that re-exports:

```typescript
// app/composables/index.ts
export { useArticle } from './blog/useArticle'
export { useLocale } from './i18n/useLocale'
```

### TypeScript typing patterns

Return explicit interface types for composables to improve documentation and IDE support. Use **MaybeRefOrGetter** for flexible input parameters and **toValue()** to normalize them:

```typescript
// app/composables/useTranslatedContent.ts
import type { MaybeRefOrGetter, Ref } from 'vue'

interface UseTranslatedContentReturn {
  content: Ref<string | undefined>
  isLoading: Ref<boolean>
  error: Ref<Error | undefined>
}

export function useTranslatedContent(
  slug: MaybeRefOrGetter<string>,
  locale: MaybeRefOrGetter<string>
): UseTranslatedContentReturn {
  const content = shallowRef<string | undefined>()
  const isLoading = ref(false)
  const error = shallowRef<Error | undefined>()

  watchEffect(async () => {
    isLoading.value = true
    try {
      const result = await queryContent(toValue(slug))
        .locale(toValue(locale))
        .findOne()
      content.value = result?.body
    } catch (e) {
      error.value = e as Error
    } finally {
      isLoading.value = false
    }
  })

  return { content, isLoading, error }
}
```

**Critical reactivity rules:** Always return plain objects containing refs, never reactive objects. Destructuring `{ x, y } = useMouse()` preserves reactivity; wrapping in `reactive()` loses it. Use `shallowRef` for large data structures like blog content to avoid deep reactivity overhead.

### Composable lifecycle constraints

Composables must be called **synchronously** in `<script setup>` or `setup()`. They cannot be called in event handlers, setTimeout, or after await statements (except in `<script setup>` where the compiler restores context). Always clean up side effects:

```typescript
export function useEventListener(
  target: MaybeRefOrGetter<EventTarget | null>,
  event: string,
  handler: EventListener
) {
  watch(
    () => toValue(target),
    (el, _, onCleanup) => {
      el?.addEventListener(event, handler)
      onCleanup(() => el?.removeEventListener(event, handler))
    },
    { immediate: true }
  )
}
```

---

## Component composition: choosing the right data-sharing pattern

The choice between props, provide/inject, composables, and slots depends on component hierarchy depth and whether you're sharing data or behavior. Props remain correct for **direct parent-child communication** (1-2 levels). Beyond that, prop drilling becomes a maintenance burden.

### Type-safe provide/inject with InjectionKey

For compound components (like shadcn-vue accordions or dialogs), provide/inject shares state across deeply nested children. Always use **Symbol-based InjectionKey** for type safety:

```typescript
// app/keys/blog.ts
import type { InjectionKey, Ref } from 'vue'

export interface ArticleContext {
  article: Ref<Article | null>
  locale: Ref<string>
  setLocale: (locale: string) => void
}

export const ArticleKey: InjectionKey<ArticleContext> = Symbol('ArticleContext')
```

```typescript
// Provider component
<script setup lang="ts">
import { provide, readonly } from 'vue'
import { ArticleKey } from '@/keys/blog'

const article = ref<Article | null>(null)
const locale = ref('en')

provide(ArticleKey, {
  article: readonly(article),
  locale: readonly(locale),
  setLocale: (newLocale: string) => { locale.value = newLocale }
})
</script>
```

Create a strict injection helper to fail fast on missing context:

```typescript
// app/composables/useInjectStrict.ts
export function injectStrict<T>(key: InjectionKey<T>, fallback?: T): T {
  const resolved = inject(key, fallback)
  if (resolved === undefined) {
    throw new Error(`Missing injection for ${key.description}`)
  }
  return resolved
}
```

### Compound components pattern from Reka UI

shadcn-vue components follow Reka UI's Root-Context pattern. The root component manages state and provides context; child components inject and consume it. This enables flexible composition without prop drilling:

```vue
<!-- Usage matches Reka UI pattern -->
<Accordion type="single" collapsible>
  <AccordionItem value="item-1">
    <AccordionTrigger>Section 1</AccordionTrigger>
    <AccordionContent>Content for section 1</AccordionContent>
  </AccordionItem>
</Accordion>
```

The `asChild` prop (from Reka UI) merges primitive behavior onto custom elements, essential for composing multiple behaviors:

```vue
<TooltipTrigger asChild>
  <DialogTrigger asChild>
    <Button>Open with tooltip</Button>
  </DialogTrigger>
</TooltipTrigger>
```

### State management for SSG context

For SSG on Cloudflare Pages, **never use `ref()` outside setup context**—this causes cross-request state pollution and hydration mismatches. Use Nuxt's `useState()` for SSR-safe shared state:

```typescript
// app/composables/useBlogState.ts
export const useCurrentLocale = () => useState<string>('locale', () => 'en')
export const useReadingProgress = () => useState<number>('progress', () => 0)
```

For complex state with actions, Pinia remains viable but `useState` suffices for most blog scenarios. The critical rule: **state accessed during SSR must survive hydration**—`useState` handles this automatically.

### Decision framework for composition patterns

| Scenario | Pattern | Rationale |
|----------|---------|-----------|
| Parent → child (1-2 levels) | Props | Explicit API, validation support |
| Deep hierarchy (3+ levels) | Provide/inject | Avoids drilling, type-safe with InjectionKey |
| Compound component context | Provide/inject | Reka UI pattern, implicit state sharing |
| Reusable stateful logic | Composable | Encapsulates reactivity + lifecycle |
| Flexible content areas | Slots (scoped) | Parent controls rendering |
| SSR-safe shared state | `useState()` | Survives hydration, no cross-request pollution |

---

## Lazy loading and bundle optimization for Cloudflare Pages

Nuxt 4 auto-generates lazy versions of all components—prefix any component with `Lazy` to dynamically import it. This creates separate chunks loaded only when the component renders.

### When to lazy load in SSG context

**Never lazy load above-the-fold content**—this delays Largest Contentful Paint and creates layout shift. For a blog, hero sections, article headers, and navigation must load eagerly. Lazy loading benefits components that are conditionally rendered or below the fold.

| Component type | Lazy? | Hydration strategy |
|----------------|-------|-------------------|
| Article header/hero | No | Immediate |
| Navigation | No | Immediate |
| Article body | No | Immediate |
| Comments section | Yes | `hydrate-on-visible` |
| Share modal | Yes | `hydrate-on-interaction` |
| Related articles | Yes | `hydrate-on-visible` |
| Footer | Yes | `hydrate-on-idle` |

### Delayed hydration in Nuxt 4

Nuxt 4 (from v3.16+) supports **delayed hydration** for lazy components, crucial for SSG performance:

```vue
<template>
  <!-- Below fold: hydrate when scrolled into view -->
  <LazyCommentsSection hydrate-on-visible />
  
  <!-- Modal: hydrate only on user interaction -->
  <LazyShareDialog v-if="showShare" hydrate-on-interaction="click" />
  
  <!-- Footer: hydrate during browser idle time -->
  <LazyFooter hydrate-on-idle />
  
  <!-- Static content: never hydrate (no interactivity needed) -->
  <LazyTableOfContents hydrate-never />
</template>
```

Use `hydrate-never` only for purely decorative content with zero interactivity—event handlers won't work on non-hydrated components.

### Client-only components for browser APIs

For components using browser-only APIs (localStorage, window dimensions, third-party widgets), use the `.client.vue` suffix or `<ClientOnly>` wrapper:

```vue
<template>
  <ClientOnly>
    <InteractiveMap :coordinates="article.location" />
    <template #fallback>
      <div class="h-64 bg-muted animate-pulse" />
    </template>
  </ClientOnly>
</template>
```

### Cloudflare Pages deployment configuration

Configure Nitro for SSG output compatible with Cloudflare Pages:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      autoSubfolderIndex: false
    }
  }
})
```

**Critical Cloudflare dashboard settings to disable** (these interfere with Vue hydration):
- Rocket Loader™
- Mirage
- Email Address Obfuscation
- Auto Minify (Nuxt handles minification)

### Bundle optimization strategies

Run `npx nuxi analyze` regularly to identify bundle bloat. For a blog with Nuxt Content, shadcn-vue, and TailwindCSS 4, target **under 200KB gzipped** for the initial bundle.

Import third-party libraries selectively:

```typescript
// ❌ Imports entire library
import _ from 'lodash'

// ✅ Tree-shakeable import
import debounce from 'lodash-es/debounce'
```

For heavy dependencies like syntax highlighters or chart libraries, use dynamic imports inside `onMounted`:

```typescript
onMounted(async () => {
  const { default: Prism } = await import('prismjs')
  await import('prismjs/components/prism-typescript')
  Prism.highlightAll()
})
```

TailwindCSS 4.1.x automatically purges unused classes—no configuration needed. The output CSS contains only classes actually used in templates.

---

## Nuxt 4 breaking changes affecting these patterns

**Directory structure:** The default `srcDir` changed to `app/`. All client code (components, composables, pages) now lives in `app/`, while `server/` remains at root. The `~` alias points to `app/`. Run `npx codemod@latest nuxt/4/migration-recipe` for automated migration.

**Shallow reactivity by default:** `useAsyncData` and `useFetch` now return `shallowRef` data. For Nuxt Content queries where you need deep reactivity on nested article properties, explicitly opt in:

```typescript
const { data } = await useAsyncData(
  `article-${slug}`,
  () => queryContent(slug).findOne(),
  { deep: true }
)
```

**TypeScript configuration:** Nuxt 4 generates separate tsconfig files for app, server, shared, and build-time code. Update your root `tsconfig.json` to reference these:

```json
{
  "extends": "./.nuxt/tsconfig.json"
}
```

**Default value changes:** `data` and `error` from `useAsyncData`/`useFetch` now default to `undefined` (previously `null`). Update type guards accordingly.

---

## Actionable guidelines for AI code generation

When generating Vue 3/Nuxt 4 code for this stack, follow these rules:

**Composables:** Name with `use*` prefix. Place in `app/composables/`. Return objects containing refs (not reactive objects). Use `shallowRef` for content data. Accept `MaybeRefOrGetter` inputs with `toValue()` normalization. Always clean up side effects in watch cleanup or `onUnmounted`.

**Component composition:** Use props for direct parent-child (≤2 levels). Use provide/inject with `InjectionKey<T>` for compound components or deep hierarchies. Provide readonly refs with explicit mutation functions. Use `useState()` for SSR-safe shared state, never module-level `ref()`.

**Lazy loading:** Prefix with `Lazy` for conditional components. Never lazy load above-fold content. Add `hydrate-on-visible` for below-fold interactive content, `hydrate-on-idle` for non-critical components, `hydrate-on-interaction` for modals. Use `.client.vue` suffix for browser-only components.

**TypeScript:** Enable strict mode. Use explicit return types on composables. Use `InjectionKey<T>` for provide/inject. Prefer interface types over inline types. Run `nuxt prepare` to regenerate type definitions after adding composables.

**SSG optimization:** Configure `nitro.preset: 'cloudflare_pages'`. Disable Cloudflare's Rocket Loader and Auto Minify. Dynamic import heavy libraries in `onMounted`. Target <200KB gzipped initial bundle.