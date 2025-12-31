# Web Worker pour Gros Volumes (>500 articles)

Pour les blogs dépassant 500 articles, déplacer la recherche dans un Web Worker évite de bloquer le thread principal :

```typescript
// public/search.worker.ts
import MiniSearch from 'minisearch'

let searchIndex: MiniSearch | null = null

self.onmessage = async (e: MessageEvent) => {
  const { type, data } = e.data

  if (type === 'init') {
    try {
      const response = await fetch('/search-index.json')
      const indexData = await response.text()

      searchIndex = MiniSearch.loadJSON(indexData, {
        fields: ['title', 'description', 'content', 'tags'],
        storeFields: ['title', 'description', 'slug', 'locale', 'pillar']
      })

      self.postMessage({ type: 'ready' })
    } catch (error) {
      self.postMessage({ type: 'error', error: String(error) })
    }
  }

  if (type === 'search' && searchIndex) {
    const results = searchIndex.search(data.query, {
      filter: data.locale ? (r) => r.locale === data.locale : undefined,
      prefix: true,
      fuzzy: 0.2
    }).slice(0, data.limit || 10)

    self.postMessage({ type: 'results', results })
  }
}
```

```typescript
// app/composables/useSearchWorker.ts
export function useSearchWorker() {
  const worker = shallowRef<Worker | null>(null)
  const results = shallowRef<SearchResult[]>([])
  const isReady = ref(false)
  const isLoading = ref(false)

  const initWorker = () => {
    if (worker.value) return

    worker.value = new Worker('/search.worker.ts', { type: 'module' })

    worker.value.onmessage = (e: MessageEvent) => {
      const { type, results: searchResults } = e.data

      if (type === 'ready') {
        isReady.value = true
        isLoading.value = false
      }

      if (type === 'results') {
        results.value = searchResults
      }
    }

    isLoading.value = true
    worker.value.postMessage({ type: 'init' })
  }

  const search = (query: string, locale?: string) => {
    if (!worker.value || !isReady.value) return
    worker.value.postMessage({ type: 'search', data: { query, locale, limit: 10 } })
  }

  onUnmounted(() => {
    worker.value?.terminate()
  })

  return { initWorker, search, results, isReady, isLoading }
}
```

**Quand utiliser un Web Worker :**

| Volume articles | Approche recommandée |
|-----------------|---------------------|
| < 200 | Composable standard |
| 200-500 | Composable standard + `loadJSONAsync()` |
| 500-1000 | Web Worker optionnel |
| > 1000 | Web Worker ou **Pagefind** |

**Avantages Web Worker :**
- Parsing index non-bloquant
- Recherche hors main thread
- TTI (Time to Interactive) préservé

**Inconvénients :**
- Complexité supplémentaire
- Communication asynchrone
- Debugging plus difficile
