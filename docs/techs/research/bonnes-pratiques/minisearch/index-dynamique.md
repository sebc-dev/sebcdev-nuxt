# MiniSearch 7.x with Nuxt Content 3 for SSG search

Nuxt Content 3 provides **native MiniSearch integration** through its `queryCollectionSearchSections` API, making client-side full-text search straightforward for static sites on Cloudflare Pages. The recommended pattern is build-time index generation using Nuxt Content's section-based extraction, combined with lazy-loaded client-side search. This approach delivers sub-100ms search responses with **zero recurring costs** and approximately 15-25KB compressed payload for typical blog-sized indexes.

---

## MiniSearch 7.x API fundamentals

MiniSearch 7.2.0 is a lightweight (~7KB gzipped) full-text search engine designed for browser environments. The library uses a radix tree data structure that consumes **50% less memory than Lunr.js** while providing fuzzy matching, prefix search, and field boosting.

**Core configuration options** define what gets indexed and returned:

```typescript
const miniSearch = new MiniSearch({
  idField: 'id',                         // Document identifier field
  fields: ['title', 'content'],          // Fields to index for search
  storeFields: ['title', 'url'],         // Fields returned with results
  
  processTerm: (term) => term.toLowerCase(),  // Term normalization
  tokenize: (text) => text.split(/[\s-]+/),   // Custom tokenization
  
  searchOptions: {
    boost: { title: 2 },                 // Field weight multipliers
    fuzzy: 0.2,                          // 20% edit distance tolerance
    prefix: true,                        // Enable prefix matching
    combineWith: 'AND'                   // Term combination logic
  }
})
```

The **document update limitation** is critical to understand: MiniSearch has no native `update()` method. To modify indexed documents, use the `discard()` + `add()` pattern or the `replace()` convenience method:

```typescript
// Recommended update pattern
miniSearch.discard(documentId)           // Mark as removed
miniSearch.add(updatedDocument)          // Re-index with same ID

// Or use replace() which does the same internally
miniSearch.replace(updatedDocument)
```

**Index serialization** enables pre-built static indexes. The critical requirement is passing identical options during deserialization:

```typescript
// Serialize at build time
const json = JSON.stringify(miniSearch)

// Deserialize client-side (options MUST match original)
const restored = MiniSearch.loadJSON(json, {
  fields: ['title', 'content'],
  storeFields: ['title', 'url']
})

// Async version for large indexes (won't block UI)
const restored = await MiniSearch.loadJSONAsync(json, options)
```

---

## Nuxt Content 3's native search extraction

Nuxt Content 3 provides `queryCollectionSearchSections`, which parses content files into searchable sections optimized for MiniSearch. Each section includes an `id` (usable as navigation link), `title`, parent `titles` hierarchy, and `content` text.

```typescript
const { data: sections } = await useAsyncData('search', () =>
  queryCollectionSearchSections('docs', {
    ignoredTags: ['code'],    // Exclude code blocks
    minHeading: 'h2',         // Section split boundaries
    maxHeading: 'h3'
  })
)
```

**The official MiniSearch recipe** from Nuxt Content documentation:

```vue
<script setup lang="ts">
import MiniSearch from 'minisearch'

const query = ref('')
const { data } = await useAsyncData('search', () =>
  queryCollectionSearchSections('docs')
)

const miniSearch = new MiniSearch({
  fields: ['title', 'content'],
  storeFields: ['title', 'content'],
  searchOptions: { prefix: true, fuzzy: 0.2 }
})

miniSearch.addAll(toValue(data.value))
const results = computed(() => miniSearch.search(toValue(query)))
</script>
```

For **multilingual content** with separate collections (blog_fr, blog_en), define parallel collections in `content.config.ts`:

```typescript
// content.config.ts
export default defineContentConfig({
  collections: {
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'en/blog/**', prefix: '' },
      schema: blogSchema
    }),
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/blog/**', prefix: '' },
      schema: blogSchema
    })
  }
})
```

Query each collection separately and build locale-specific indexes:

```typescript
// server/api/search-data.ts
export default eventHandler(async (event) => {
  const [enSections, frSections] = await Promise.all([
    queryCollectionSearchSections(event, 'blog_en'),
    queryCollectionSearchSections(event, 'blog_fr')
  ])
  return { en: enSections, fr: frSections }
})
```

---

## Build-time index generation for SSG

The optimal SSG pattern generates a serialized MiniSearch index during `nuxt generate`, then loads it on-demand client-side. This approach eliminates runtime indexing costs while keeping the initial bundle clean.

**Server API route for index generation:**

```typescript
// server/api/search-index.ts
import MiniSearch from 'minisearch'

export default eventHandler(async (event) => {
  const sections = await queryCollectionSearchSections(event, 'docs')
  
  const miniSearch = new MiniSearch({
    fields: ['title', 'content'],
    storeFields: ['title', 'content', 'id'],
    searchOptions: { prefix: true, fuzzy: 0.2, boost: { title: 2 } }
  })
  
  miniSearch.addAll(sections)
  return JSON.stringify(miniSearch)
})
```

For **prerendering the index** as a static JSON file, use Nuxt hooks:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  hooks: {
    'nitro:build:public-assets': async (nitro) => {
      // Index generation runs during static generation
      const sections = await fetchSearchSections()
      const index = buildMiniSearchIndex(sections)
      
      await writeFile(
        resolve(nitro.options.output.publicDir, 'search-index.json'),
        JSON.stringify(index)
      )
    }
  }
})
```

---

## The useSearch composable pattern

A production-ready composable handles lazy loading, debouncing, and reactive state management:

```typescript
// composables/useSearch.ts
import { ref, computed, watch } from 'vue'
import { useDebounceFn } from '@vueuse/core'
import type MiniSearch from 'minisearch'

interface SearchResult {
  id: string
  title: string
  content: string
  score: number
}

export function useSearch(collectionName: string = 'docs') {
  const query = ref('')
  const results = ref<SearchResult[]>([])
  const isLoading = ref(false)
  const isIndexLoaded = ref(false)
  
  let miniSearch: MiniSearch | null = null
  
  const loadIndex = async () => {
    if (miniSearch) return
    isLoading.value = true
    
    // Parallel load: library + index data
    const [MiniSearchModule, indexData] = await Promise.all([
      import('minisearch'),
      $fetch<string>(`/search-index-${collectionName}.json`)
    ])
    
    miniSearch = MiniSearchModule.default.loadJSON(indexData, {
      fields: ['title', 'content'],
      storeFields: ['title', 'content', 'id']
    })
    
    isIndexLoaded.value = true
    isLoading.value = false
  }
  
  const performSearch = useDebounceFn((term: string) => {
    if (!miniSearch || !term.trim()) {
      results.value = []
      return
    }
    results.value = miniSearch.search(term, { limit: 20 })
  }, 250)
  
  watch(query, (newQuery) => {
    if (newQuery.trim()) {
      loadIndex().then(() => performSearch(newQuery))
    } else {
      results.value = []
    }
  })
  
  return { query, results, isLoading, isIndexLoaded, loadIndex }
}
```

**Usage in a search component:**

```vue
<script setup lang="ts">
const { query, results, isLoading } = useSearch('blog_en')
</script>

<template>
  <input v-model="query" @focus="loadIndex" placeholder="Search..." />
  <div v-if="isLoading">Loading...</div>
  <ul v-else>
    <li v-for="result in results" :key="result.id">
      <NuxtLink :to="result.id">{{ result.title }}</NuxtLink>
    </li>
  </ul>
</template>
```

---

## Cloudflare Pages deployment and caching

Cloudflare Pages **does not cache JSON files by default**—only standard static assets (JS, CSS, images). Configure explicit caching via `public/_headers`:

```
# Search index with daily refresh
/search-index*.json
  Cache-Control: public, max-age=86400, stale-while-revalidate=604800
  Content-Type: application/json

# Nuxt assets (hashed filenames)
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

# HTML pages
/*.html
  Cache-Control: public, max-age=0, must-revalidate
```

Cloudflare automatically applies **Brotli compression** (75-85% reduction), so a 100KB search index transfers as ~15-25KB. The free tier provides unlimited bandwidth and requests with no server functions required for client-side search.

**Lazy loading MiniSearch** ensures zero initial bundle impact:

```typescript
// Only loads when search is triggered
const initSearch = async () => {
  const MiniSearch = (await import('minisearch')).default
  const indexData = await fetch('/search-index.json').then(r => r.json())
  return MiniSearch.loadJSON(indexData, options)
}
```

For the Nuxt 4 `app/` directory structure, place composables in `app/composables/` with auto-imports configured:

```
app/
├── composables/
│   ├── useSearch.ts
│   └── useSearchIndex.ts
├── components/
│   └── SearchModal.vue
public/
├── search-index-en.json
├── search-index-fr.json
└── _headers
```

---

## Index size optimization strategies

For large document sets, apply these optimizations to reduce index payload:

- **Minimize stored fields**: Only store fields needed for result display
- **Truncate content**: Limit indexed content to first 3000-5000 characters per document
- **Filter stopwords**: Remove common words that inflate index size without improving relevance
- **Separate locale indexes**: Build per-language indexes rather than one combined index

```typescript
const stopWords = new Set(['and', 'or', 'the', 'a', 'an', 'in', 'on', 'at'])

const miniSearch = new MiniSearch({
  fields: ['title', 'content'],
  storeFields: ['title', 'id'],  // Minimal stored fields
  processTerm: (term) => stopWords.has(term) ? null : term.toLowerCase()
})

// Truncate long content before indexing
const optimizedSections = sections.map(s => ({
  ...s,
  content: s.content.slice(0, 3000)
}))
```

**Typical index sizes:**
- 100 blog posts (~500 words each): 50-100KB uncompressed, **10-20KB compressed**
- 500 documentation pages: 200-400KB uncompressed, **40-80KB compressed**

---

## Complete implementation checklist

The integration requires these components working together:

1. **content.config.ts**: Define multilingual collections with appropriate source patterns
2. **server/api/search-index.ts**: Generate serialized MiniSearch indexes per locale
3. **composables/useSearch.ts**: Handle lazy loading, debouncing, and reactive search state
4. **public/_headers**: Configure Cloudflare caching for JSON assets
5. **nuxt.config.ts**: Set Cloudflare Pages preset and configure build hooks if generating static index files

**Build-time index generation** runs during `nuxt generate`, producing static JSON files that Cloudflare serves with edge caching. Client-side code dynamically imports MiniSearch only when users interact with search, keeping initial page loads fast while delivering instant search results once loaded.

## Conclusion

The MiniSearch + Nuxt Content 3 combination provides an efficient client-side search solution for static sites. Key implementation insights: use `queryCollectionSearchSections` for automatic content parsing, serialize indexes at build time with `JSON.stringify(miniSearch)`, lazy-load both the library and index data on user interaction, and configure explicit `Cache-Control` headers for JSON files on Cloudflare Pages. For multilingual sites, build separate per-locale indexes and load them based on current locale. This architecture achieves sub-100ms search latency with zero server costs and minimal payload overhead.