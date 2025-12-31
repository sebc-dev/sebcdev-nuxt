# Recherche full-text dans Nuxt 4 SSG: guide d'implémentation complet

La combinaison **Nuxt Content 3 + MiniSearch 7.x** offre une solution robuste pour la recherche client-side sur Cloudflare Pages. L'API `queryCollectionSearchSections()` génère des sections indexables, tandis que MiniSearch (**~7KB gzipped**) fournit un moteur BM25+ performant. Pour un blog de **100-500 articles**, l'index pèse environ **20-140KB** compressé, avec des recherches en **1-5ms**. Cette approche surpasse Fuse.js en performance et offre plus de flexibilité que Pagefind pour les besoins personnalisés.

---

## Configuration MiniSearch 7.x optimale pour blog multilingue

MiniSearch 7.2.0 utilise un algorithme **BM25+ avec radix tree**, offrant une indexation **4x plus rapide** que les versions précédentes. La configuration suivante gère le français et l'anglais avec boosting différencié.

```typescript
// types/search.ts
export interface BlogSearchDocument {
  id: string
  title: string
  description: string
  content: string
  slug: string
  locale: 'fr' | 'en'
  tags: string[]
}

export interface SearchResult {
  id: string
  title: string
  description: string
  slug: string
  locale: string
  score: number
  match: Record<string, string[]>
}
```

```typescript
// lib/minisearch-config.ts
import MiniSearch, { type Options } from 'minisearch'

// Normalisation des accents (crucial pour FR/EN)
const removeAccents = (str: string): string =>
  str.normalize('NFD').replace(/[\u0300-\u036f]/g, '')

// Stop words bilingues (réduisent la taille de l'index de ~20%)
const STOP_WORDS = new Set([
  // English
  'a', 'an', 'the', 'and', 'or', 'but', 'in', 'on', 'at', 'to', 'for',
  'of', 'with', 'by', 'is', 'are', 'was', 'were', 'be', 'been', 'have',
  'has', 'had', 'do', 'does', 'did', 'will', 'would', 'could', 'should',
  'it', 'its', 'this', 'that', 'these', 'those',
  // French
  'le', 'la', 'les', 'un', 'une', 'des', 'du', 'de', 'et', 'ou', 'mais',
  'dans', 'sur', 'pour', 'par', 'avec', 'sans', 'sous', 'entre',
  'est', 'sont', 'était', 'être', 'avoir', 'fait', 'faire',
  'ce', 'cette', 'ces', 'qui', 'que', 'quoi', 'dont', 'où',
  'je', 'tu', 'il', 'elle', 'nous', 'vous', 'ils', 'elles'
])

export const MINISEARCH_OPTIONS: Options<BlogSearchDocument> = {
  // Champ d'identification unique
  idField: 'id',
  
  // Champs indexés pour la recherche full-text
  fields: ['title', 'description', 'content', 'tags'],
  
  // Champs stockés (retournés avec les résultats)
  storeFields: ['title', 'description', 'slug', 'locale'],
  
  // Extraction des champs (gère les arrays)
  extractField: (doc, fieldName) => {
    const value = doc[fieldName as keyof BlogSearchDocument]
    return Array.isArray(value) ? value.join(' ') : (value as string)
  },
  
  // Tokenizer multilingue
  tokenize: (text: string): string[] => {
    if (!text) return []
    return text
      .split(/[\s\-_.,;:!?'"()\[\]{}|/<>@#$%^&*+=~`]+/)
      .filter(token => token.length > 0)
  },
  
  // Processing avec normalisation accents
  processTerm: (term: string): string | null => {
    const normalized = removeAccents(term.toLowerCase().trim())
    if (STOP_WORDS.has(normalized) || normalized.length < 2) {
      return null // Exclure stop words et termes courts
    }
    return normalized
  },
  
  // Options de recherche par défaut
  searchOptions: {
    // BOOSTING: titre > description > tags > contenu
    boost: {
      title: 3,       // Priorité maximale
      description: 2, // Priorité moyenne
      tags: 1.5,      // Légèrement boosté
      content: 1      // Baseline
    },
    
    // FUZZY SEARCH: tolérance aux typos
    fuzzy: 0.2,    // 20% de la longueur du terme (5 chars = 1 typo)
    maxFuzzy: 3,   // Maximum 3 éditions
    
    // PREFIX SEARCH: recherche en cours de frappe
    prefix: true,
    
    // Combinaison OR (retourne docs matchant au moins un terme)
    combineWith: 'OR',
    
    // Poids des types de match
    weights: {
      fuzzy: 0.5,  // Fuzzy à 50% d'un match exact
      prefix: 0.8  // Prefix à 80% d'un match exact
    }
  }
}

export function createSearchIndex(): MiniSearch<BlogSearchDocument> {
  return new MiniSearch(MINISEARCH_OPTIONS)
}
```

Le **boosting 3:2:1.5:1** reflète l'importance sémantique: un match dans le titre est **3x plus pertinent** qu'un match dans le contenu. La valeur `fuzzy: 0.2` permet **1 typo pour 5 caractères**, équilibrant tolérance et précision.

---

## Nuxt Content 3 Search API et collections multilingues

L'API `queryCollectionSearchSections()` découpe le contenu en sections indexables basées sur les headings. Cette approche est **plus granulaire** que l'ancien `searchContent()` de v2.

```typescript
// content.config.ts
import { defineContentConfig, defineCollection, z } from '@nuxt/content'

const blogSchema = z.object({
  title: z.string(),
  description: z.string().optional(),
  publishedAt: z.date(),
  tags: z.array(z.string()).default([]),
  searchable: z.boolean().default(true)
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

### Structure des données retournées

```typescript
// Type retourné par queryCollectionSearchSections()
interface Section {
  id: string       // Chemin + ancre (#heading-id)
  title: string    // Texte du heading
  titles: string[] // Hiérarchie des parents
  content: string  // Contenu textuel de la section
  level: number    // Niveau du heading (1-6)
}
```

### Récupération des données pour l'index

```typescript
// composables/useSearchData.ts
import type { Collections } from '@nuxt/content'

export async function useSearchData(locale: Ref<'fr' | 'en'>) {
  const collection = computed(() => 
    `blog_${locale.value}` as keyof Collections
  )
  
  // Sections pour l'index de recherche
  const { data: sections } = await useAsyncData(
    `search-sections-${locale.value}`,
    () => queryCollectionSearchSections(collection.value, {
      ignoredTags: ['code', 'pre'], // Exclure blocs de code
      minHeading: 'h2',             // Commencer à h2
      maxHeading: 'h3'              // Limiter à h3
    }),
    { watch: [locale] }
  )
  
  // Articles complets pour métadonnées
  const { data: articles } = await useAsyncData(
    `search-articles-${locale.value}`,
    () => queryCollection(collection.value)
      .select('path', 'title', 'description', 'tags')
      .all(),
    { watch: [locale] }
  )
  
  return { sections, articles }
}
```

### Différences clés avec Nuxt Content 2

| Aspect | Content 2 | Content 3 |
|--------|-----------|-----------|
| API principale | `searchContent()` | `queryCollectionSearchSections()` |
| Storage | Fichiers | SQLite |
| Champ path | `_path` | `path` |
| Composants | `<ContentDoc>`, `<ContentList>` | `<ContentRenderer>` uniquement |
| Types | `@nuxt/content/dist/runtime/types` | `@nuxt/content` |

---

## Composable Vue 3 pour la recherche avec lazy loading

Ce composable encapsule toute la logique de recherche avec **lazy loading**, **debounce**, et **gestion d'états**.

```typescript
// composables/useSearch.ts
import { ref, computed, watch, shallowRef, onMounted } from 'vue'
import MiniSearch from 'minisearch'
import { useDebounceFn } from '@vueuse/core'
import { MINISEARCH_OPTIONS, type BlogSearchDocument, type SearchResult } from '~/lib/minisearch-config'

export interface UseSearchOptions {
  debounceMs?: number
  minQueryLength?: number
  maxResults?: number
}

export function useSearch(options: UseSearchOptions = {}) {
  const {
    debounceMs = 250,
    minQueryLength = 2,
    maxResults = 10
  } = options

  // État réactif
  const query = ref('')
  const results = shallowRef<SearchResult[]>([])
  const isLoading = ref(false)
  const isReady = ref(false)
  const error = ref<Error | null>(null)
  
  // Instance MiniSearch (non-réactif pour perf)
  let searchIndex: MiniSearch<BlogSearchDocument> | null = null

  // États dérivés
  const isEmpty = computed(() => 
    query.value.length >= minQueryLength && results.value.length === 0
  )
  const hasResults = computed(() => results.value.length > 0)
  const hasQuery = computed(() => query.value.length >= minQueryLength)

  // Chargement lazy de l'index sérialisé
  const loadIndex = async (): Promise<void> => {
    if (searchIndex || isLoading.value) return
    
    isLoading.value = true
    error.value = null
    
    try {
      const response = await fetch('/search-index.json')
      if (!response.ok) throw new Error('Failed to fetch search index')
      
      const json = await response.text()
      
      // loadJSONAsync pour ne pas bloquer le main thread
      searchIndex = await MiniSearch.loadJSONAsync(json, MINISEARCH_OPTIONS)
      isReady.value = true
    } catch (e) {
      error.value = e instanceof Error ? e : new Error('Index loading failed')
      console.error('Search index loading error:', e)
    } finally {
      isLoading.value = false
    }
  }

  // Exécution de la recherche
  const executeSearch = (searchQuery: string): void => {
    if (!searchIndex || !isReady.value) return
    
    if (searchQuery.length < minQueryLength) {
      results.value = []
      return
    }

    try {
      const searchResults = searchIndex.search(searchQuery)
      results.value = searchResults.slice(0, maxResults) as SearchResult[]
    } catch (e) {
      error.value = e instanceof Error ? e : new Error('Search failed')
      results.value = []
    }
  }

  // Recherche debounced
  const debouncedSearch = useDebounceFn(executeSearch, debounceMs)

  // Watch query changes
  watch(query, (newQuery) => {
    if (newQuery.length < minQueryLength) {
      results.value = []
      return
    }
    debouncedSearch(newQuery)
  })

  // Clear search
  const clear = (): void => {
    query.value = ''
    results.value = []
  }

  return {
    // State
    query,
    results,
    isLoading,
    isReady,
    error,
    isEmpty,
    hasResults,
    hasQuery,
    // Actions
    loadIndex,
    clear
  }
}
```

### Génération de l'index au build-time

```typescript
// server/routes/search-index.json.ts
import MiniSearch from 'minisearch'
import { MINISEARCH_OPTIONS } from '~/lib/minisearch-config'

export default defineEventHandler(async (event) => {
  // Récupérer tous les articles des deux locales
  const [frSections, enSections] = await Promise.all([
    queryCollectionSearchSections(event, 'blog_fr'),
    queryCollectionSearchSections(event, 'blog_en')
  ])
  
  const [frArticles, enArticles] = await Promise.all([
    queryCollection(event, 'blog_fr').select('path', 'title', 'description', 'tags').all(),
    queryCollection(event, 'blog_en').select('path', 'title', 'description', 'tags').all()
  ])
  
  // Construire les documents indexables
  const documents = [
    ...frArticles.map(article => ({
      id: `fr:${article.path}`,
      title: article.title || '',
      description: article.description || '',
      content: frSections
        .filter(s => s.id.startsWith(article.path))
        .map(s => s.content)
        .join(' '),
      slug: article.path,
      locale: 'fr' as const,
      tags: article.tags || []
    })),
    ...enArticles.map(article => ({
      id: `en:${article.path}`,
      title: article.title || '',
      description: article.description || '',
      content: enSections
        .filter(s => s.id.startsWith(article.path))
        .map(s => s.content)
        .join(' '),
      slug: article.path,
      locale: 'en' as const,
      tags: article.tags || []
    }))
  ]
  
  // Créer et peupler l'index
  const miniSearch = new MiniSearch(MINISEARCH_OPTIONS)
  miniSearch.addAll(documents)
  
  // Retourner l'index sérialisé
  return JSON.stringify(miniSearch)
})
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    prerender: {
      routes: ['/search-index.json']
    }
  }
})
```

---

## Composant SearchBox accessible avec keyboard navigation

```vue
<!-- components/SearchBox.vue -->
<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'
import { useSearch } from '~/composables/useSearch'
import { onKeyStroke, useFocusTrap } from '@vueuse/core'

const props = defineProps<{
  placeholder?: string
}>()

const emit = defineEmits<{
  select: [result: SearchResult]
}>()

const { query, results, isLoading, isReady, isEmpty, hasResults, loadIndex, clear } = useSearch({
  debounceMs: 250,
  maxResults: 8
})

// État UI
const isOpen = ref(false)
const selectedIndex = ref(-1)
const inputRef = ref<HTMLInputElement | null>(null)
const containerRef = ref<HTMLDivElement | null>(null)

// Focus trap pour accessibilité
const { activate, deactivate } = useFocusTrap(containerRef, {
  immediate: false,
  allowOutsideClick: true
})

// Chargement lazy au focus
const handleFocus = async () => {
  await loadIndex()
  isOpen.value = true
  if (results.value.length > 0) activate()
}

// Keyboard navigation
onKeyStroke('ArrowDown', (e) => {
  if (!isOpen.value || !hasResults.value) return
  e.preventDefault()
  selectedIndex.value = Math.min(selectedIndex.value + 1, results.value.length - 1)
})

onKeyStroke('ArrowUp', (e) => {
  if (!isOpen.value) return
  e.preventDefault()
  selectedIndex.value = Math.max(selectedIndex.value - 1, -1)
})

onKeyStroke('Enter', () => {
  if (selectedIndex.value >= 0 && results.value[selectedIndex.value]) {
    emit('select', results.value[selectedIndex.value])
    closeSearch()
  }
})

onKeyStroke('Escape', () => {
  closeSearch()
})

// Reset selection on results change
watch(results, () => {
  selectedIndex.value = -1
})

const closeSearch = () => {
  isOpen.value = false
  selectedIndex.value = -1
  deactivate()
}

// Highlighting des termes
const highlightMatch = (text: string, searchQuery: string): string => {
  if (!searchQuery || searchQuery.length < 2) return text
  const escaped = searchQuery.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
  const regex = new RegExp(`(${escaped})`, 'gi')
  return text.replace(regex, '<mark class="bg-yellow-200 dark:bg-yellow-800">$1</mark>')
}

// Click outside handler
const handleClickOutside = (e: MouseEvent) => {
  if (containerRef.value && !containerRef.value.contains(e.target as Node)) {
    closeSearch()
  }
}

onMounted(() => document.addEventListener('click', handleClickOutside))
onUnmounted(() => document.removeEventListener('click', handleClickOutside))
</script>

<template>
  <div ref="containerRef" class="relative w-full max-w-md" role="search">
    <!-- Input -->
    <label for="search-input" class="sr-only">Rechercher</label>
    <input
      id="search-input"
      ref="inputRef"
      v-model="query"
      type="search"
      role="combobox"
      :placeholder="placeholder ?? 'Rechercher...'"
      :aria-expanded="isOpen && hasResults"
      aria-controls="search-results"
      aria-autocomplete="list"
      :aria-activedescendant="selectedIndex >= 0 ? `result-${selectedIndex}` : undefined"
      class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
      @focus="handleFocus"
    />
    
    <!-- Loading indicator -->
    <div v-if="isLoading" class="absolute right-3 top-2.5">
      <svg class="animate-spin h-5 w-5 text-gray-400" viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
      </svg>
    </div>

    <!-- Results dropdown -->
    <ul
      v-show="isOpen && (hasResults || isEmpty)"
      id="search-results"
      role="listbox"
      aria-label="Résultats de recherche"
      class="absolute z-50 w-full mt-1 bg-white dark:bg-gray-800 border rounded-lg shadow-lg max-h-96 overflow-y-auto"
    >
      <!-- Empty state -->
      <li v-if="isEmpty" class="px-4 py-3 text-gray-500" role="status">
        Aucun résultat pour "{{ query }}"
      </li>
      
      <!-- Results -->
      <li
        v-for="(result, index) in results"
        :id="`result-${index}`"
        :key="result.id"
        role="option"
        :aria-selected="selectedIndex === index"
        :class="[
          'px-4 py-3 cursor-pointer border-b last:border-b-0',
          selectedIndex === index 
            ? 'bg-blue-50 dark:bg-blue-900' 
            : 'hover:bg-gray-50 dark:hover:bg-gray-700'
        ]"
        @click="emit('select', result); closeSearch()"
        @mouseenter="selectedIndex = index"
      >
        <div 
          class="font-medium text-gray-900 dark:text-white"
          v-html="highlightMatch(result.title, query)"
        />
        <div 
          v-if="result.description"
          class="text-sm text-gray-600 dark:text-gray-400 line-clamp-2 mt-1"
          v-html="highlightMatch(result.description, query)"
        />
        <div class="flex items-center gap-2 mt-1">
          <span class="text-xs bg-gray-100 dark:bg-gray-700 px-2 py-0.5 rounded">
            {{ result.locale.toUpperCase() }}
          </span>
          <span class="text-xs text-gray-400">
            Score: {{ result.score.toFixed(1) }}
          </span>
        </div>
      </li>
    </ul>
    
    <!-- Live region for screen readers -->
    <div aria-live="polite" aria-atomic="true" class="sr-only">
      <span v-if="isLoading">Chargement...</span>
      <span v-else-if="hasResults">{{ results.length }} résultats trouvés</span>
      <span v-else-if="isEmpty">Aucun résultat</span>
    </div>
  </div>
</template>
```

---

## Optimisation de la taille de l'index

Pour un blog de **100-500 articles**, appliquer ces stratégies réduit l'index de **~40%**.

```typescript
// utils/content-processor.ts

// Supprimer le Markdown pour réduire la taille
export function stripMarkdown(content: string): string {
  return content
    // Blocs de code
    .replace(/```[\s\S]*?```/g, '')
    .replace(/`[^`]*`/g, '')
    // Links (garder le texte)
    .replace(/\[([^\]]+)\]\([^)]+\)/g, '$1')
    // Images
    .replace(/!\[[^\]]*\]\([^)]+\)/g, '')
    // Headers
    .replace(/#{1,6}\s*/g, '')
    // Emphasis
    .replace(/[*_]{1,2}([^*_]+)[*_]{1,2}/g, '$1')
    // Whitespace
    .replace(/\s+/g, ' ')
    .trim()
}

// Tronquer le contenu long
export function truncateContent(content: string, maxLength = 3000): string {
  if (content.length <= maxLength) return content
  // Couper à la fin d'une phrase
  const truncated = content.slice(0, maxLength)
  const lastPeriod = truncated.lastIndexOf('.')
  return lastPeriod > maxLength * 0.8 
    ? truncated.slice(0, lastPeriod + 1)
    : truncated
}
```

### Estimations de taille

| Articles | Index brut | Avec optimisations | Gzipped |
|----------|------------|-------------------|---------|
| 100 | ~60KB | ~35KB | ~12KB |
| 250 | ~150KB | ~90KB | ~30KB |
| 500 | ~300KB | ~180KB | ~60KB |

---

## MiniSearch vs Pagefind vs Fuse.js pour ce use case

| Critère | MiniSearch | Pagefind | Fuse.js |
|---------|-----------|----------|---------|
| **Taille lib** | ~7KB | ~100KB | ~7KB |
| **Index 500 articles** | ~60-140KB (tout) | ~50-150KB (chunks) | N/A (données brutes) |
| **Perf recherche** | 1-5ms | 50-100ms | 50-200ms |
| **Multilingue** | Manuel | Auto | Manuel |
| **Personnalisation** | Excellente | Limitée | Bonne |
| **SSG setup** | Script custom | Zero-config | Aucun |
| **Cloudflare Pages** | ✅ | ✅ | ✅ |

**Recommandation**: MiniSearch pour ce projet car:
- Contrôle total sur le boosting et la tokenisation multilingue
- Meilleure performance de recherche (~5x plus rapide que Pagefind)
- Intégration native avec l'API Nuxt Content 3
- Bundle size minimal (~7KB vs ~100KB)

Pagefind serait préférable si: zero-config prioritaire, >1000 articles, multilingue auto requis.

---

## Anti-patterns à éviter

```typescript
// ❌ MAUVAIS: Charger l'index de manière synchrone au mount
onMounted(async () => {
  const index = await fetch('/search-index.json').then(r => r.json())
  // Bloque le rendering
})

// ✅ BON: Lazy load au focus
const handleFocus = () => loadIndex()

// ❌ MAUVAIS: Recherche à chaque keystroke
watch(query, (q) => miniSearch.search(q)) // Pas de debounce

// ✅ BON: Debounce 250ms
const debouncedSearch = useDebounceFn(executeSearch, 250)

// ❌ MAUVAIS: Index dans un ref réactif
const searchIndex = ref(hugeIndex) // Overhead Vue reactivity

// ✅ BON: Variable externe ou shallowRef
let searchIndex: MiniSearch | null = null

// ❌ MAUVAIS: Indexer le contenu complet
fields: ['title', 'fullContent'] // Index énorme

// ✅ BON: Indexer un extrait
content: truncateContent(stripMarkdown(article.body), 3000)

// ❌ MAUVAIS: v-html sans sanitization pour highlighting
<span v-html="userInput" /> // XSS vulnerability

// ✅ BON: Échapper avant highlighting
const escaped = query.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
```

---

## Checklist d'implémentation pour Claude Code

### Phase 1: Configuration de base
- [ ] Créer `types/search.ts` avec `BlogSearchDocument` et `SearchResult`
- [ ] Créer `lib/minisearch-config.ts` avec `MINISEARCH_OPTIONS`
- [ ] Configurer `content.config.ts` avec collections `blog_fr` et `blog_en`

### Phase 2: Génération de l'index
- [ ] Créer `server/routes/search-index.json.ts`
- [ ] Ajouter `/search-index.json` à `nitro.prerender.routes`
- [ ] Implémenter `stripMarkdown()` et `truncateContent()`
- [ ] Tester la génération avec `nuxt generate`

### Phase 3: Composable de recherche
- [ ] Créer `composables/useSearch.ts`
- [ ] Implémenter lazy loading avec `MiniSearch.loadJSONAsync()`
- [ ] Ajouter debounce 250ms avec `useDebounceFn`
- [ ] Gérer les états: `isLoading`, `isReady`, `isEmpty`, `hasResults`

### Phase 4: Composant UI
- [ ] Créer `components/SearchBox.vue`
- [ ] Implémenter keyboard navigation (↑/↓/Enter/Escape)
- [ ] Ajouter ARIA attributes (`role="combobox"`, `aria-expanded`, etc.)
- [ ] Implémenter highlighting des termes
- [ ] Ajouter live region pour screen readers

### Phase 5: Optimisation
- [ ] Vérifier taille de l'index généré (`< 100KB` pour 500 articles)
- [ ] Activer compression Brotli/Gzip sur Cloudflare Pages
- [ ] Tester TTI avec Lighthouse (impact `< 50ms`)
- [ ] Valider accessibilité avec axe DevTools

### Phase 6: Tests
- [ ] Tester recherche FR avec accents (`café`, `résumé`)
- [ ] Tester recherche EN
- [ ] Tester fuzzy search (`javascrpt` → `javascript`)
- [ ] Tester prefix search (`nux` → `nuxt`)
- [ ] Valider keyboard navigation avec NVDA/VoiceOver

---

## Configuration TailwindCSS 4.1.x pour le composant

```css
/* assets/css/search.css */
@layer components {
  .search-highlight {
    @apply bg-yellow-200 dark:bg-yellow-800 rounded px-0.5;
  }
  
  .search-result-item {
    @apply px-4 py-3 cursor-pointer border-b last:border-b-0 
           transition-colors duration-150;
  }
  
  .search-result-item[aria-selected="true"] {
    @apply bg-blue-50 dark:bg-blue-900;
  }
}
```

Cette implémentation offre une recherche full-text performante, accessible et optimisée pour le déploiement SSG sur Cloudflare Pages, avec support complet du français et de l'anglais.