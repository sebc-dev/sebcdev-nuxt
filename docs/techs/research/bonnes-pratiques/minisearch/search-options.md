# Full-text search in Nuxt Content 3 with MiniSearch: A complete implementation guide

Nuxt Content 3 requires **manual MiniSearch integration** rather than using a built-in search composable—the `searchContent()` API from v2 no longer exists. For SSG deployment on Cloudflare Pages with zero-cost constraints, you'll use `queryCollectionSearchSections()` to generate searchable data at build time, then integrate MiniSearch 7.x on the client side. This architecture offers excellent control over search behavior but demands understanding the new SQL-based collections system and careful attention to index file sizes (Cloudflare's **25 MiB limit** per file).

## The architecture shift from Nuxt Content v2 to v3

Nuxt Content 3.10.0+ abandons the experimental `searchContent()` composable entirely. Instead, it provides `queryCollectionSearchSections()`, which generates section-based search data from your content collections. You then integrate this data with any search library—MiniSearch, Fuse.js, or Pagefind.

The v3 architecture uses **SQLite** for content storage (via WASM in browsers), enabling SQL-like queries across your content. For SSG, the database is dumped at build time and loaded client-side on first query. This means search happens entirely client-side after initial page load—perfect for zero-cost Cloudflare Pages deployment.

```typescript
// The NEW way in Nuxt Content 3 - no searchContent() exists
const { data: sections } = await useAsyncData('search', () => 
  queryCollectionSearchSections('docs', {
    ignoredTags: ['code'],  // Skip code blocks
    minHeading: 'h2',       // New in v3.10.0
    maxHeading: 'h3'        // Control granularity
  })
)
```

The `queryCollectionSearchSections()` returns an array of `Section` objects containing `id`, `title`, `titles` (hierarchy), `content`, and `level`. These map directly to MiniSearch's indexing requirements.

## MiniSearch 7.x configuration for optimal search quality

MiniSearch 7.2.0 (current as of December 2025) targets ES2018 and provides comprehensive full-text search capabilities. The **fuzzy parameter** accepts three value types: boolean (`true` for default fuzziness), integer (exact edit distance), or fractional **0-1 range** representing percentage of term length.

The recommended configuration for documentation sites uses `fuzzy: 0.2`, which calculates maximum edit distance as `round(0.2 × termLength)`. For "typescript" (10 characters), this allows 2 character edits—catching typos like "typscript" or "tyepscript" while avoiding irrelevant matches.

```typescript
import MiniSearch from 'minisearch'

const miniSearch = new MiniSearch({
  // Required: fields to index for full-text search
  fields: ['title', 'content'],
  
  // Optional: fields returned with results (not searchable)
  storeFields: ['title', 'titles', 'id'],
  
  // Default search options applied to all queries
  searchOptions: {
    boost: { title: 2 },        // Title matches score 2x
    fuzzy: 0.2,                 // 20% of term length
    prefix: true,               // Enable autocomplete matching
    combineWith: 'OR',          // Default for broad results
    weights: {
      fuzzy: 0.8,               // Fuzzy matches worth 80%
      prefix: 0.5               // Prefix matches worth 50%
    }
  }
})

// Add all sections from Nuxt Content
miniSearch.addAll(sections)
```

**Critical note**: MiniSearch has no built-in `limit` option in `.search()`. You must slice results manually: `miniSearch.search(query).slice(0, 10)`.

Boolean operators default to `'OR'` for search (higher recall) and `'AND'` for autoSuggest. Use `combineWith: 'AND'` when all terms must match, and leverage nested query objects for complex boolean expressions like `zen AND (motorcycle OR archery)`.

## Complete Nuxt Content 3 integration pattern

The integration requires combining `queryCollectionSearchSections()` with MiniSearch in a Vue 3 Composition API pattern. Use debouncing to prevent excessive searches during typing.

```vue
<script setup lang="ts">
import MiniSearch, { type SearchResult } from 'minisearch'
import { refDebounced } from '@vueuse/core'

interface Section {
  id: string
  title: string
  titles: string[]
  content: string
  level: number
}

interface ContentSearchResult extends SearchResult {
  title: string
  titles: string[]
}

const query = ref('')
const debouncedQuery = refDebounced(query, 300) // 300ms debounce

// Fetch sections at build time for SSG
const { data: sections } = await useAsyncData('search-sections', () =>
  queryCollectionSearchSections('docs')
)

// Initialize MiniSearch with sections
const miniSearch = computed(() => {
  if (!sections.value) return null
  
  const ms = new MiniSearch<Section>({
    fields: ['title', 'content', 'titles'],
    storeFields: ['title', 'titles', 'id'],
    searchOptions: {
      boost: { title: 4, titles: 2, content: 1 },
      fuzzy: 0.2,
      prefix: true
    },
    // Custom extraction for array fields
    extractField: (doc, fieldName) => {
      if (fieldName === 'titles') return doc.titles.join(' ')
      return doc[fieldName as keyof Section]?.toString() ?? ''
    }
  })
  
  ms.addAll(sections.value)
  return ms
})

const results = computed<ContentSearchResult[]>(() => {
  if (!miniSearch.value || !debouncedQuery.value.trim()) return []
  return miniSearch.value.search(debouncedQuery.value).slice(0, 10)
})
</script>

<template>
  <input v-model="query" type="search" placeholder="Search documentation..." />
  <ul v-if="results.length">
    <li v-for="result in results" :key="result.id">
      <NuxtLink :to="result.id">{{ result.title }}</NuxtLink>
    </li>
  </ul>
</template>
```

For i18n support, create separate collections per locale and dynamically select based on current language:

```typescript
const collection = `content_${locale.value}` as keyof Collections
const sections = await queryCollectionSearchSections(collection)
```

## SSG configuration and Cloudflare Pages deployment

Cloudflare Pages free tier provides unlimited bandwidth and requests with **25 MiB maximum file size** and **20,000 files per site**. These limits directly impact search index strategy.

Configure Nuxt for static generation with Cloudflare Pages preset:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/content'],
  
  ssr: true,
  
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      crawlLinks: true,
      routes: ['/']
    }
  },
  
  content: {
    database: {
      type: 'sqlite' // Default, works with SSG
    }
  },
  
  // Separate search library into its own chunk
  vite: {
    build: {
      rollupOptions: {
        output: {
          manualChunks: {
            search: ['minisearch']
          }
        }
      }
    }
  }
})
```

Typical index sizes scale linearly: **20 posts ≈ 100-150KB** (compressed: 20-30KB), **100 posts ≈ 400-700KB** (compressed: 80-150KB), **1000 posts ≈ 3-7MB** (compressed: 500KB-1.5MB). Stay well under the 25MB limit by indexing only necessary fields and truncating long content.

## Performance optimization strategies

Lazy loading search functionality prevents adding search library weight to the initial bundle. Load MiniSearch only when users interact with search:

```vue
<script setup lang="ts">
const searchIndex = shallowRef<InstanceType<typeof MiniSearch> | null>(null)
const isLoaded = ref(false)

const loadSearch = async () => {
  if (isLoaded.value) return
  
  const [MiniSearchModule, sections] = await Promise.all([
    import('minisearch'),
    queryCollectionSearchSections('docs')
  ])
  
  searchIndex.value = new MiniSearchModule.default({
    fields: ['title', 'content'],
    storeFields: ['title', 'id'],
    searchOptions: { fuzzy: 0.2, prefix: true }
  })
  
  searchIndex.value.addAll(sections)
  isLoaded.value = true
}
</script>

<template>
  <input @focus="loadSearch" @input="handleSearch" />
</template>
```

Index size optimization techniques include: **indexing truncated content** (first 500-1000 characters), **using integer IDs** instead of long slugs (one case study reduced index from 700KB to 62KB), **excluding code blocks** via `ignoredTags: ['code']`, and **limiting heading depth** with `minHeading`/`maxHeading` options added in v3.10.0.

For very large sites (500+ pages), consider **Pagefind** instead of MiniSearch. Pagefind splits indexes into chunks, loading only relevant portions—achieving under 300KB total for 10,000+ page sites (used by MDN).

## Field boosting and scoring configuration

MiniSearch provides three boosting mechanisms: field boosting (`boost`), document boosting (`boostDocument`), and term boosting (`boostTerm`, added in v7.1.0).

```typescript
const results = miniSearch.search(query, {
  // Field boosting: title matches score 4x
  boost: { title: 4, description: 2, content: 1 },
  
  // Document boosting: favor recent content
  boostDocument: (id, term, storedFields) => {
    if (!storedFields?.publishedAt) return 1
    const age = Date.now() - new Date(storedFields.publishedAt).getTime()
    const yearInMs = 365 * 24 * 60 * 60 * 1000
    return 1 + Math.max(0, 1 - age / yearInMs) // Up to 2x for recent
  },
  
  // Term boosting: first term most important
  boostTerm: (term, index, terms) => terms.length - index,
  
  // Filter results by category
  filter: (result) => result.category === 'tutorial'
})
```

The VitePress search configuration (widely adopted) uses `boost: { title: 4, text: 2, titles: 1 }` with `fuzzy: 0.2` and `prefix: true` as sensible defaults.

## Common mistakes and anti-patterns to avoid

**Indexing too many fields** wastes memory and slows searches. Only index fields users will search—store additional display fields with `storeFields`. The `body` field containing full content often shouldn't be in `storeFields` if you only need titles for display.

**Re-creating MiniSearch on re-renders** destroys index state and causes performance issues. Use `shallowRef` or initialize outside reactive scope.

**Ignoring debounce timing** degrades UX. Use **200-300ms** debounce delay—200ms is Algolia's recommendation, but 300ms works better for complex searches. Avoid going over 300ms, which feels sluggish.

**Setting fuzzy too high** (0.4+) returns too many irrelevant results. Stick to **0.2 as default**, use 0.1 for strict matching, and cap with `maxFuzzy: 4` to prevent excessive edit distances on long queries.

**Filtering without storeFields** fails silently. Filter functions can only access fields listed in `storeFields`—attempting to filter by `category` requires including it in stored fields even if you don't search by it.

## Conclusion

Implementing full-text search in Nuxt Content 3 for Cloudflare Pages SSG requires understanding the fundamental architecture shift from v2's `searchContent()` to v3's `queryCollectionSearchSections()` with manual MiniSearch integration. The optimal configuration uses **fuzzy: 0.2**, **prefix: true**, field boosting favoring titles, and **300ms debouncing**. Keep indexes under 10MB by truncating content, using integer IDs, and excluding code blocks. For sites exceeding 500 pages, evaluate Pagefind's chunked index approach. The v3.10.0 `minHeading`/`maxHeading` options provide granular control over search section generation, and lazy loading ensures search functionality doesn't impact initial page load on zero-cost Cloudflare Pages deployments.