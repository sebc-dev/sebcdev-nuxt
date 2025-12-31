# Full-text search in Nuxt 4 SSG: MiniSearch vs Pagefind

For a Nuxt 4 / Nuxt Content 3 blog with 50-500 posts deployed as SSG on Cloudflare Pages, **Pagefind emerges as the optimal choice** due to its SSG-native architecture, chunked index loading (~100KB total network payload), and built-in multilingual support. However, **MiniSearch 7.2.0** remains the best traditional in-memory alternative when you need dynamic index updates or greater control over search behavior. This guide covers both approaches with implementation patterns specific to the Nuxt 4 `app/` directory structure.

---

## MiniSearch 7.x delivers excellent performance for blog-scale content

**Version status**: MiniSearch 7.2.0 (released December 2024) is the current stable release. The v7.0.0 update introduced a breaking change targeting ES9 (ES2018) instead of ES6, using Unicode character class escapes in the tokenizer RegExp. If you need ES2015 compatibility, apply the `babel-plugin-transform-unicode-sets-regex` plugin. Indexes serialized with v4.x or later remain compatible—no migration needed for most users.

### Optimal field configuration for blog content

```typescript
// app/utils/searchConfig.ts
import MiniSearch from 'minisearch'

export const createSearchIndex = () => new MiniSearch({
  idField: 'slug',
  fields: ['title', 'description', 'body', 'tags'],
  storeFields: ['title', 'slug', 'date', 'excerpt'], // Minimize for size
  
  searchOptions: {
    boost: {
      title: 4,        // Highest priority
      tags: 3,         // Categorical matches valuable
      description: 2,  // Summary content
      body: 1          // Base weight (larger content dilutes matches)
    },
    fuzzy: 0.2,        // 20% of term length as max edit distance
    prefix: true       // Enable prefix matching
  }
})
```

The **storeFields vs fields distinction** is critical for index size optimization: `fields` are indexed for search but not returned in results, while `storeFields` appear in results. For a 500-post blog, storing only metadata (**title, slug, date, excerpt**) instead of full body text reduces index size from 300KB+ to approximately **100-150KB uncompressed** (20-40KB gzipped).

### French/English multilingual configuration

MiniSearch has no built-in stemming, so use the `snowball-stemmers` package for French and English support. **Separate indexes per language** yield better relevance than a combined index:

```typescript
// app/utils/stemmer.ts
import snowballFactory from 'snowball-stemmers'

const stemmers = {
  fr: snowballFactory.newStemmer('french'),
  en: snowballFactory.newStemmer('english')
}

export const createLocalizedIndex = (locale: 'fr' | 'en') => {
  return new MiniSearch({
    idField: 'slug',
    fields: ['title', 'description', 'body', 'tags'],
    storeFields: ['title', 'slug', 'date', 'excerpt'],
    
    processTerm: (term, _fieldName) => {
      if (!term || term.length < 2) return null
      const lowered = term.toLowerCase()
      return stemmers[locale].stem(lowered)
    },
    
    tokenize: (text) => text.split(/[\s\p{P}]+/u).filter(t => t.length > 1),
    
    searchOptions: {
      fuzzy: 0.2,
      prefix: true,
      boost: { title: 4, tags: 3, description: 2, body: 1 }
    }
  })
}
```

---

## Pre-built index generation leverages Nuxt Content 3's search sections API

Nuxt Content 3 introduces `queryCollectionSearchSections()`, a purpose-built function that breaks content into searchable sections with headings and content text. This eliminates manual MDC component parsing and provides clean, indexable content.

### Server route pattern for SSG pre-rendering

Create a server route that generates the search index, then configure Nitro to pre-render it:

```typescript
// server/routes/search/[locale].json.ts
import MiniSearch from 'minisearch'
import type { H3Event } from 'h3'

export default defineEventHandler(async (event: H3Event) => {
  const locale = getRouterParam(event, 'locale') || 'fr'
  const validLocales = ['en', 'fr']
  
  if (!validLocales.includes(locale)) {
    throw createError({ statusCode: 404, message: 'Locale not found' })
  }
  
  // Use language-separated collections
  const collectionName = `blog_${locale}` as const
  
  const sections = await queryCollectionSearchSections(event, collectionName, {
    ignoredTags: ['code', 'pre', 'script', 'style'],
  })
  
  // Pre-build MiniSearch index for instant client loading
  const miniSearch = new MiniSearch({
    fields: ['title', 'content'],
    storeFields: ['title', 'content', 'id'],
    searchOptions: { prefix: true, fuzzy: 0.2 }
  })
  
  miniSearch.addAll(sections)
  
  return {
    locale,
    sections,
    index: JSON.stringify(miniSearch), // Serialized for MiniSearch.loadJSON()
    generatedAt: new Date().toISOString()
  }
})
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare-pages-static',
    prerender: {
      crawlLinks: true,
      routes: [
        '/search/fr.json',
        '/search/en.json'
      ]
    }
  }
})
```

### Content configuration with language-separated collections

```typescript
// content.config.ts
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { z } from 'zod'

const blogSchema = z.object({
  title: z.string(),
  description: z.string().optional(),
  date: z.string(),
  tags: z.array(z.string()).optional()
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

---

## Client-side loading requires SSG-safe patterns

For SSG sites, the search index must load **only on the client after hydration**. This prevents SSR mismatches and ensures MiniSearch (a browser-only library) initializes correctly.

### Complete useSearch composable with TypeScript

```typescript
// app/composables/useSearch.ts
import { ref, computed, watch, onUnmounted, shallowRef } from 'vue'
import { useDebounceFn } from '@vueuse/core'
import type MiniSearchType from 'minisearch'

interface SearchResult {
  id: string
  title: string
  slug?: string
  score: number
  terms: string[]
}

interface UseSearchOptions {
  debounceMs?: number
  maxResults?: number
  minQueryLength?: number
}

export function useSearch(options: UseSearchOptions = {}) {
  const { debounceMs = 300, maxResults = 10, minQueryLength = 2 } = options
  const { locale } = useI18n()
  
  const query = ref('')
  const results = ref<SearchResult[]>([])
  const isLoading = ref(false)
  const isIndexLoaded = ref(false)
  const error = ref<Error | null>(null)
  const selectedIndex = ref(-1)
  
  const miniSearch = shallowRef<MiniSearchType | null>(null)
  
  const hasResults = computed(() => results.value.length > 0)
  const selectedResult = computed(() => 
    selectedIndex.value >= 0 ? results.value[selectedIndex.value] : null
  )

  async function loadIndex(): Promise<void> {
    if (isIndexLoaded.value) return
    isLoading.value = true
    error.value = null

    try {
      // Dynamic import—only loads on client
      const MiniSearch = (await import('minisearch')).default
      
      // Fetch pre-built index
      const response = await $fetch<{ index: string }>(`/search/${locale.value}.json`)
      
      // Load serialized index instantly
      miniSearch.value = MiniSearch.loadJSON(response.index, {
        fields: ['title', 'content'],
        storeFields: ['title', 'content', 'id']
      })
      
      isIndexLoaded.value = true
    } catch (e) {
      error.value = e instanceof Error ? e : new Error('Index load failed')
    } finally {
      isLoading.value = false
    }
  }

  function performSearch(): void {
    if (!miniSearch.value || query.value.length < minQueryLength) {
      results.value = []
      selectedIndex.value = -1
      return
    }

    try {
      const searchResults = miniSearch.value.search(query.value, {
        fuzzy: 0.2,
        prefix: true,
        combineWith: 'AND'
      })
      results.value = searchResults.slice(0, maxResults) as SearchResult[]
      selectedIndex.value = results.value.length > 0 ? 0 : -1
    } catch (e) {
      error.value = e instanceof Error ? e : new Error('Search failed')
    }
  }

  const debouncedSearch = useDebounceFn(performSearch, debounceMs)

  watch(query, (newQuery) => {
    if (newQuery.length < minQueryLength) {
      results.value = []
      return
    }
    isLoading.value = true
    debouncedSearch()
  })

  // Reload index when locale changes
  watch(locale, () => {
    isIndexLoaded.value = false
    miniSearch.value = null
    loadIndex()
  })

  function navigateDown() {
    if (results.value.length === 0) return
    selectedIndex.value = (selectedIndex.value + 1) % results.value.length
  }

  function navigateUp() {
    if (results.value.length === 0) return
    selectedIndex.value = selectedIndex.value <= 0 
      ? results.value.length - 1 
      : selectedIndex.value - 1
  }

  function clearSearch() {
    query.value = ''
    results.value = []
    selectedIndex.value = -1
  }

  onUnmounted(() => {
    miniSearch.value = null
  })

  return {
    query, results, isLoading, isIndexLoaded, error,
    selectedIndex, hasResults, selectedResult,
    loadIndex, performSearch, navigateDown, navigateUp, clearSearch
  }
}
```

### Search component with lazy initialization

```vue
<!-- app/components/SearchModal.vue -->
<script setup lang="ts">
const searchInput = ref<HTMLInputElement | null>(null)
const isOpen = ref(false)

const {
  query, results, isLoading, isIndexLoaded,
  loadIndex, navigateDown, navigateUp, selectedResult, clearSearch, selectedIndex
} = useSearch({ debounceMs: 300, maxResults: 8 })

// Load index ONLY after hydration
onMounted(async () => {
  await loadIndex()
})

function handleKeydown(e: KeyboardEvent) {
  switch (e.key) {
    case 'ArrowDown': e.preventDefault(); navigateDown(); break
    case 'ArrowUp': e.preventDefault(); navigateUp(); break
    case 'Enter':
      e.preventDefault()
      if (selectedResult.value) navigateTo(selectedResult.value.id)
      break
    case 'Escape': isOpen.value = false; clearSearch(); break
  }
}
</script>

<template>
  <ClientOnly>
    <div v-if="isOpen" class="search-modal">
      <input
        ref="searchInput"
        v-model="query"
        type="search"
        placeholder="Rechercher..."
        @keydown="handleKeydown"
      />
      <ul v-if="results.length > 0" role="listbox">
        <li
          v-for="(result, index) in results"
          :key="result.id"
          :class="{ selected: selectedIndex === index }"
          @click="navigateTo(result.id)"
        >
          {{ result.title }}
        </li>
      </ul>
    </div>
    <template #fallback>
      <button disabled>Search loading...</button>
    </template>
  </ClientOnly>
</template>
```

---

## Performance comparison reveals Pagefind's SSG advantage

Benchmarks for a 500-post blog reveal significant differences between libraries:

| Metric | Pagefind | MiniSearch | Fuse.js | Lunr.js |
|--------|----------|------------|---------|---------|
| **Index size (500 posts)** | ~100KB chunked | ~150KB | N/A (full data) | ~500KB+ |
| **Search latency** | <100ms | <10ms | 1-5 seconds | <10ms |
| **Memory usage** | Minimal (chunked) | Moderate | High | High |
| **Multilingual** | Built-in | Manual (snowball) | Manual | Plugins |
| **SSG integration** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |

**Fuse.js is unsuitable** for this use case—its linear O(n) search produces **1-5 second latencies** at 500 posts with full content. Lunr.js suffers from index bloat: 700 documents can generate **~70MB TokenStore in memory**, with serialized indexes for large datasets reaching 61MB raw.

Pagefind's chunked architecture loads only the index fragments needed per query, keeping total network payload under **300KB even for 10,000-page sites**. Most blogs see approximately **100KB total** including the WebAssembly search engine.

---

## Pagefind provides a simpler SSG-native alternative

For pure SSG sites where rebuild-on-content-change is acceptable, Pagefind eliminates the complexity of manual index generation:

### Integration with Nuxt generate

```json
// package.json
{
  "scripts": {
    "generate": "nuxi generate",
    "postgenerate": "npx pagefind --site .output/public --output-path .output/public/pagefind"
  }
}
```

### Vue component implementation

```vue
<!-- app/components/PagefindSearch.vue -->
<script setup lang="ts">
const query = ref('')
const results = ref<any[]>([])
let pagefind: any = null

onMounted(async () => {
  pagefind = await import('/pagefind/pagefind.js')
  await pagefind.init()
})

async function search() {
  if (!pagefind || query.value.length < 2) {
    results.value = []
    return
  }
  const response = await pagefind.search(query.value)
  results.value = await Promise.all(
    response.results.slice(0, 10).map((r: any) => r.data())
  )
}

watchDebounced(query, search, { debounce: 300 })
</script>

<template>
  <ClientOnly>
    <input v-model="query" type="search" placeholder="Search..." />
    <ul v-if="results.length">
      <li v-for="result in results" :key="result.url">
        <a :href="result.url">{{ result.meta.title }}</a>
        <p v-html="result.excerpt"></p>
      </li>
    </ul>
  </ClientOnly>
</template>
```

Pagefind automatically detects languages from the `<html lang="">` attribute and creates separate indexes. For explicit multilingual control, add `data-pagefind-filter="language:fr"` attributes to your content wrapper.

---

## Nuxt 4 directory structure for search implementation

```
project-root/
├── nuxt.config.ts
├── content.config.ts
├── app/
│   ├── composables/
│   │   └── useSearch.ts           # Search composable
│   ├── components/
│   │   └── SearchModal.vue        # Search UI
│   ├── utils/
│   │   └── searchConfig.ts        # MiniSearch configuration
│   └── pages/
│       └── [...slug].vue
├── content/
│   ├── fr/blog/                   # French content
│   └── en/blog/                   # English content
├── server/
│   └── routes/
│       └── search/
│           └── [locale].json.ts   # Pre-rendered search index
└── public/                        # Static assets (Pagefind output)
```

Key placement rules: composables go in `app/composables/` for auto-import, server routes must remain in `server/` at the project root (not inside `app/`), and content stays in `content/` outside the app directory.

---

## Decision framework for your implementation

Choose **Pagefind** when:
- Content changes trigger full rebuilds anyway
- You want zero-configuration multilingual support
- Minimizing JavaScript bundle size is priority
- Search traffic is unpredictable (chunked loading scales better)

Choose **MiniSearch** when:
- You need runtime index updates without rebuilds
- Custom ranking algorithms or search behavior required
- Building SPA features with dynamic content
- Edge function search is planned (Netlify Edge, Cloudflare Workers)

For a multilingual Nuxt 4 blog with ~500 posts on Cloudflare Pages' free tier, **Pagefind is the recommended default**. It requires no additional dependencies, handles French/English automatically, and delivers sub-100ms search with minimal network overhead. Reserve MiniSearch for scenarios requiring dynamic content or custom search logic that Pagefind cannot accommodate.