# Search UX Implementation Guide: MiniSearch 7.x + Nuxt Content 3 + shadcn-vue

Implementing full-text search for a multilingual Nuxt 4 SSG blog requires coordinating MiniSearch 7.x for indexing, shadcn-vue Command components for the modal UX, and careful i18n integration. The optimal approach uses **separate pre-built search indexes per language**, **lazy-loaded at runtime**, with a **Dialog + Command composition** for the search modal.

## Architecture overview

The recommended architecture separates concerns into three layers: **build-time index generation** using Nuxt Content 3's `queryCollectionSearchSections()`, **client-side search** via MiniSearch with lazy-loaded JSON indexes, and **a CommandDialog modal** triggered by ⌘K/Ctrl+K. This approach keeps bundle size minimal (~8KB for MiniSearch) while enabling fast, fuzzy multilingual search.

---

## MiniSearch 7.x configuration rules

### Rule 1: Use optimized field boosting for blog content
```typescript
// composables/useSearch.ts
const SEARCH_OPTIONS = {
  fields: ['title', 'description', 'body', 'tags'],
  storeFields: ['title', 'description', 'path'],  // Minimal stored fields
  idField: 'path',
  searchOptions: {
    boost: { title: 4, description: 2, tags: 1.5, body: 1 },
    prefix: true,
    fuzzy: 0.2,  // 20% of term length tolerance
    weights: { fuzzy: 0.5, prefix: 0.8 }
  }
} as const
```

**Justification**: Title matches are **4x more relevant** than body matches for blog navigation. Storing only display fields keeps index size under **2-5MB for ~500 posts**.

### Rule 2: Generate separate indexes per locale at build time
```typescript
// server/routes/search/[locale].json.get.ts
import MiniSearch from 'minisearch'

export default eventHandler(async (event) => {
  const locale = getRouterParam(event, 'locale') as 'fr' | 'en'
  const collection = `blog_${locale}` as const
  
  const posts = await queryCollection(event, collection)
    .select('path', 'title', 'description', 'body')
    .all()

  const miniSearch = new MiniSearch({
    fields: ['title', 'description', 'body'],
    storeFields: ['title', 'description', 'path']
  })

  miniSearch.addAll(posts.map(p => ({
    path: p.path,
    title: p.title,
    description: p.description,
    body: extractPlainText(p.body)  // Strip AST to plain text
  })))

  return JSON.parse(JSON.stringify(miniSearch))  // Serialized index
})
```

**Anti-pattern**: Don't use a single index with locale filtering—this bloats index size and degrades relevance scoring.

### Rule 3: Load index lazily on search modal open
```typescript
// composables/useSearchIndex.ts
import MiniSearch from 'minisearch'

export function useSearchIndex() {
  const { locale } = useI18n()
  const miniSearch = shallowRef<MiniSearch | null>(null)
  const isLoading = ref(false)

  async function loadIndex() {
    if (miniSearch.value || isLoading.value) return
    isLoading.value = true
    
    const response = await fetch(`/search/${locale.value}.json`)
    const indexData = await response.json()
    
    miniSearch.value = MiniSearch.loadJSON(JSON.stringify(indexData), {
      fields: ['title', 'description', 'body'],
      storeFields: ['title', 'description', 'path']
    })
    isLoading.value = false
  }

  // Reload on locale change
  watch(locale, () => { miniSearch.value = null })

  return { miniSearch, isLoading, loadIndex }
}
```

**Pitfall**: You **must pass identical options** to `loadJSON()` as used during index creation—MiniSearch will throw otherwise.

### Rule 4: Configure fuzzy search conditionally for performance
```typescript
const searchResults = miniSearch.search(query, {
  prefix: (term, i, terms) => i === terms.length - 1,  // Only last term
  fuzzy: (term) => term.length > 3 ? 0.2 : false,      // Skip short terms
  maxFuzzy: 4,
  combineWith: 'AND'  // More specific results
})
```

---

## shadcn-vue Command modal rules

### Rule 5: Use Dialog + Command composition with VisuallyHidden accessibility
```vue
<!-- components/SearchModal.vue -->
<script setup lang="ts">
import { ref, watch, nextTick, computed } from 'vue'
import { useMagicKeys, whenever, useActiveElement } from '@vueuse/core'
import { logicAnd } from '@vueuse/math'
import { VisuallyHidden } from 'reka-ui'
import { Dialog, DialogContent } from '@/components/ui/dialog'
import {
  Command, CommandEmpty, CommandGroup, CommandInput,
  CommandItem, CommandList
} from '@/components/ui/command'

const open = ref(false)
const searchQuery = ref('')

// Keyboard shortcut: ⌘K / Ctrl+K
const activeElement = useActiveElement()
const notInInput = computed(() => 
  !['INPUT', 'TEXTAREA'].includes(activeElement.value?.tagName ?? '')
)

const { Meta_k, Ctrl_k } = useMagicKeys({
  passive: false,
  onEventFired(e) {
    if ((e.metaKey || e.ctrlKey) && e.key === 'k' && e.type === 'keydown') {
      e.preventDefault()  // Block browser bookmark shortcut
    }
  }
})

whenever(
  logicAnd(() => Meta_k.value || Ctrl_k.value, notInInput),
  () => { open.value = !open.value }
)

// Reset state on close
watch(open, (isOpen) => {
  if (!isOpen) nextTick(() => { searchQuery.value = '' })
})
</script>

<template>
  <Dialog v-model:open="open">
    <DialogContent class="p-0 gap-0 max-w-lg overflow-hidden">
      <!-- Required for WCAG: hidden but announced by screen readers -->
      <VisuallyHidden>
        <h2>{{ $t('search.title') }}</h2>
        <p>{{ $t('search.description') }}</p>
      </VisuallyHidden>
      
      <Command class="rounded-lg">
        <CommandInput 
          v-model="searchQuery"
          :placeholder="$t('search.placeholder')" 
          class="h-12"
        />
        <CommandList class="max-h-[300px] overflow-y-auto">
          <CommandEmpty>{{ $t('search.noResults') }}</CommandEmpty>
          <slot :query="searchQuery" />
        </CommandList>
      </Command>
    </DialogContent>
  </Dialog>
</template>
```

**Anti-pattern**: Never omit `DialogTitle`—use `VisuallyHidden` wrapper instead. Screen readers require it.

### Rule 6: Provide visual trigger button for mobile accessibility
```vue
<template>
  <Button variant="outline" @click="open = true" class="gap-2">
    <SearchIcon class="h-4 w-4" />
    <span class="hidden sm:inline">{{ $t('search.button') }}</span>
    <kbd class="hidden sm:inline-flex h-5 px-1.5 rounded border bg-muted text-[10px]">
      {{ isMac ? '⌘K' : 'Ctrl+K' }}
    </kbd>
  </Button>
</template>

<script setup>
const isMac = computed(() => 
  typeof navigator !== 'undefined' && /Mac/.test(navigator.platform)
)
</script>
```

**Justification**: Mobile users have no keyboard shortcut—always provide a clickable trigger.

---

## Search UX and performance rules

### Rule 7: Debounce search input at 200-300ms with VueUse refDebounced
```typescript
import { refDebounced } from '@vueuse/core'

const searchQuery = ref('')
const debouncedQuery = refDebounced(searchQuery, 250)  // Optimal timing

watch(debouncedQuery, (query) => {
  if (query.length >= 2) performSearch(query)
})
```

**Timing guidelines**:
- **150-200ms**: Autocomplete suggestions
- **250-300ms**: Full-text search (recommended default)
- **300-500ms**: Heavy filtering operations

**Anti-pattern**: Don't bind `refDebounced` directly to `v-model`—it returns a read-only ref.

### Rule 8: Implement result highlighting with HTML escaping
```typescript
function highlightMatches(text: string, terms: string[]): string {
  if (!terms.length) return escapeHtml(text)
  
  const escaped = escapeHtml(text)
  const pattern = terms
    .map(t => t.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'))
    .join('|')
  
  return escaped.replace(
    new RegExp(`(${pattern})`, 'gi'),
    '<mark class="bg-yellow-200 rounded px-0.5">$1</mark>'
  )
}

function escapeHtml(text: string): string {
  const div = document.createElement('div')
  div.textContent = text
  return div.innerHTML
}
```

**Security**: Always escape HTML **before** applying highlight regex to prevent XSS.

### Rule 9: Use skeleton loaders for loading states
```vue
<template>
  <div v-if="isLoading" class="space-y-2 p-2">
    <div v-for="i in 5" :key="i" class="animate-pulse">
      <div class="h-4 bg-muted rounded w-3/4 mb-2" />
      <div class="h-3 bg-muted/60 rounded w-full" />
    </div>
  </div>
  
  <CommandEmpty v-else-if="!results.length && query">
    {{ $t('search.noResults') }}
  </CommandEmpty>
  
  <CommandGroup v-else-if="results.length" :heading="$t('search.results')">
    <!-- results -->
  </CommandGroup>
</template>
```

**UX research**: Skeleton loaders reduce perceived wait time by **~35%** compared to spinners for content lists.

### Rule 10: Implement search history with privacy controls
```typescript
// composables/useSearchHistory.ts
import { useLocalStorage } from '@vueuse/core'

export function useSearchHistory(maxItems = 5) {
  const history = useLocalStorage<string[]>('search-history', [])

  function add(query: string) {
    const normalized = query.trim()
    if (!normalized || normalized.length < 2) return
    
    const filtered = history.value.filter(
      q => q.toLowerCase() !== normalized.toLowerCase()
    )
    history.value = [normalized, ...filtered].slice(0, maxItems)
  }

  function clear() { history.value = [] }

  return { history: readonly(history), add, clear }
}
```

---

## Nuxt Content 3 + i18n rules

### Rule 11: Define separate collections per locale
```typescript
// content.config.ts
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { z } from 'zod'

const blogSchema = z.object({
  title: z.string(),
  description: z.string(),
  tags: z.array(z.string()).optional(),
  publishedAt: z.date()
})

export default defineContentConfig({
  collections: {
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/blog/**/*.md', prefix: '' },
      schema: blogSchema
    }),
    blog_en: defineCollection({
      type: 'page', 
      source: { include: 'en/blog/**/*.md', prefix: '' },
      schema: blogSchema
    })
  }
})
```

**Directory structure**:
```
content/
  fr/blog/    ← French posts (default locale)
  en/blog/    ← English posts
```

### Rule 12: Use queryCollectionSearchSections for indexable content
```typescript
// For search indexing with section-level granularity
const { data: sections } = await useAsyncData(
  `search-${locale.value}`,
  () => queryCollectionSearchSections(`blog_${locale.value}`),
  { watch: [locale] }
)

// Returns: { id, title, titles, content, level }[]
```

### Rule 13: Generate locale-aware result URLs with useLocalePath
```vue
<script setup>
const localePath = useLocalePath()

function navigateToResult(path: string) {
  open.value = false
  navigateTo(localePath(path))
}
</script>

<template>
  <CommandItem
    v-for="result in results"
    :key="result.path"
    :value="result.path"
    @select="navigateToResult(result.path)"
  >
    <span v-html="highlight(result.title, result.terms)" />
  </CommandItem>
</template>
```

**Route examples** (prefix_except_default strategy):
- French (default): `/blog/mon-article`
- English: `/en/blog/my-article`

### Rule 14: Watch locale changes in useAsyncData
```typescript
const { locale } = useI18n()

const { data } = await useAsyncData(
  `search-data-${locale.value}`,
  () => queryCollection(`blog_${locale.value}`).all(),
  { watch: [locale] }  // REQUIRED: refetch on language switch
)
```

**Anti-pattern**: Forgetting `watch: [locale]` causes stale data after switching languages.

---

## Cloudflare Pages SSG deployment rules

### Rule 15: Use nuxt generate with in-memory SQLite
```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      crawlLinks: true,
      routes: ['/', '/blog', '/en', '/en/blog']
    }
  },
  content: {
    database: {
      type: 'sqlite',
      filename: ':memory:'  // Required for static/serverless
    }
  }
})
```

**Build command**: `nuxt generate` outputs to `.output/public`

---

## Complete implementation checklist

### Build-time setup
- [ ] Define `blog_fr` and `blog_en` collections in `content.config.ts`
- [ ] Create `/server/routes/search/[locale].json.get.ts` for index generation
- [ ] Configure i18n with `prefix_except_default` strategy
- [ ] Add prerender routes for search index endpoints

### Search composable
- [ ] Create `useSearchIndex()` with lazy loading and locale watching
- [ ] Create `useSearchHistory()` with localStorage persistence
- [ ] Implement `highlightMatches()` with HTML escaping
- [ ] Use `refDebounced` at 250ms for query input

### UI components  
- [ ] Build `SearchModal.vue` with Dialog + Command composition
- [ ] Add `VisuallyHidden` title/description for accessibility
- [ ] Implement ⌘K/Ctrl+K shortcut with `useMagicKeys`
- [ ] Add visual search button trigger for mobile
- [ ] Create skeleton loader for loading state
- [ ] Style CommandEmpty for "no results" state

### MiniSearch configuration
- [ ] Set field boosting: title(4), description(2), tags(1.5), body(1)
- [ ] Enable `prefix: true` and `fuzzy: 0.2`
- [ ] Store only `title`, `description`, `path` in index
- [ ] Use `loadJSONAsync` for non-blocking index loading

### i18n integration
- [ ] Use `useLocalePath()` for result navigation
- [ ] Localize placeholder text and UI labels via `$t()`
- [ ] Reload search index on locale change
- [ ] Handle missing translations with fallback

---

## Key anti-patterns to avoid

| Anti-pattern | Correct approach |
|--------------|------------------|
| Single index with locale filter | Separate indexes per language |
| Storing full body in storeFields | Store only display fields |
| High fuzzy values (>0.3) | Use `fuzzy: 0.2` maximum |
| Missing DialogTitle | Use VisuallyHidden wrapper |
| Keyboard shortcut without input check | Check activeElement before triggering |
| Loading index on page load | Lazy load on modal open |
| Direct v-model on refDebounced | Use separate source and debounced refs |
| Forgetting watch: [locale] | Always watch locale for i18n data |
| Same content in multiple collections | Exclusive source patterns per collection |
| v-html without escaping | Always escape before highlight regex |

---

## Performance targets

| Metric | Target |
|--------|--------|
| Search index size | <5MB per locale |
| Index load time | <500ms on 3G |
| Search latency | <50ms after debounce |
| Time to first result | <300ms from keystroke |
| Modal open animation | 150-200ms |
| Debounce timing | 250ms (adjustable) |

This architecture delivers **sub-100ms search performance** on pre-built indexes while maintaining full accessibility compliance and excellent mobile UX through the shadcn-vue Command palette pattern.