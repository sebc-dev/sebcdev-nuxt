# Composable useSearch Complet

```typescript
// app/composables/useSearch.ts
import { ref, computed, watch, shallowRef } from 'vue'
import MiniSearch from 'minisearch'
import { useDebounceFn } from '@vueuse/core'
import { MINISEARCH_OPTIONS } from '~/lib/minisearch-config'
import type { BlogSearchDocument, SearchResult } from '~/types/search'

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

      // loadJSONAsync() : parsing incrémental qui ne bloque PAS le main thread
      // Contrairement à loadJSON() (synchrone), permet au navigateur de rester interactif
      // pendant le parsing d'un index volumineux (>100KB)
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

  // Recherche debounced (250ms par défaut, maxWait 1s pour garantir une réponse)
  const debouncedSearch = useDebounceFn(executeSearch, debounceMs, { maxWait: 1000 })

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

  // Auto-suggestions pour autocomplétion (dropdown)
  const getSuggestions = (partialQuery: string, limit = 5): string[] => {
    if (!searchIndex || !isReady.value || partialQuery.length < 2) return []

    return searchIndex.autoSuggest(partialQuery, {
      fuzzy: 0.2,
      prefix: true
    })
      .slice(0, limit)
      .map(suggestion => suggestion.suggestion)
  }

  return {
    // State
    query,
    results,
    isLoading: readonly(isLoading),
    isReady: readonly(isReady),
    error: readonly(error),
    isEmpty,
    hasResults,
    hasQuery,
    // Actions
    loadIndex,
    clear,
    getSuggestions
  }
}
```

**Points clés :**

| Aspect | Implementation | Raison |
|--------|----------------|--------|
| `shallowRef` pour results | Évite la réactivité profonde | Performance sur grands tableaux |
| `loadJSONAsync()` | Parsing incrémental asynchrone | Ne bloque pas le main thread — navigateur reste interactif |
| Debounce 250ms + `maxWait: 1000` | Via `useDebounceFn` | Évite les recherches à chaque keystroke, garantit réponse en 1s max |
| Variable externe pour index | `let searchIndex` | Pas d'overhead Vue reactivity |

## loadJSON() vs loadJSONAsync()

| Méthode | Comportement | Impact UX | Quand utiliser |
|---------|--------------|-----------|----------------|
| `MiniSearch.loadJSON()` | Synchrone, bloquant | Freeze UI pendant parsing | Index < 50KB |
| `MiniSearch.loadJSONAsync()` | Incrémental, non-bloquant | UI reste fluide | Index > 50KB (**recommandé**) |

**⚠️ Toujours utiliser `loadJSONAsync()`** pour les blogs avec plus de 50 articles. Le parsing d'un index de 200KB peut bloquer le main thread pendant 100-200ms avec `loadJSON()`, causant des jank visibles.

## Cache d'index par locale (éviter rechargements)

Pour les sites multilingues, un cache `Map` évite de recharger l'index lors du changement de locale :

```typescript
// app/composables/useSearchWithCache.ts
const indexCache = new Map<string, MiniSearch<BlogSearchDocument>>()

export function useSearchWithCache() {
  const { locale } = useI18n()
  const index = shallowRef<MiniSearch<BlogSearchDocument> | null>(null)
  const isLoading = ref(false)

  const loadIndex = async (localeCode: string): Promise<void> => {
    // Retour depuis cache si disponible (instantané)
    if (indexCache.has(localeCode)) {
      index.value = indexCache.get(localeCode)!
      return
    }

    isLoading.value = true
    try {
      const response = await fetch(`/search-index-${localeCode}.json`)
      const json = await response.text()
      const miniSearch = await MiniSearch.loadJSONAsync(json, MINISEARCH_OPTIONS)

      // Mise en cache pour réutilisation
      indexCache.set(localeCode, miniSearch)
      index.value = miniSearch
    } finally {
      isLoading.value = false
    }
  }

  // Rechargement automatique lors du changement de locale
  watch(() => locale.value, (newLocale) => loadIndex(newLocale), { immediate: true })

  return { index, isLoading, loadIndex }
}
```

**Comportement :**

| Action | Sans cache | Avec cache |
|--------|------------|------------|
| Premier chargement FR | ~200ms (fetch + parse) | ~200ms |
| Switch vers EN | ~200ms | ~200ms (premier load) |
| Retour vers FR | ~200ms | **~0ms** (instantané) |

**⚠️ Mémoire** : Le cache conserve tous les index chargés. Pour >5 langues, implémenter une stratégie LRU (Least Recently Used).
