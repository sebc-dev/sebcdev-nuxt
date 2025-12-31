# Scheduling et Yield

## Le problème

Les tâches JavaScript longues (> 50ms) bloquent le main thread et dégradent l'INP. Les patterns "yield" et "scheduling" découpent les tâches pour permettre aux interactions utilisateur de s'intercaler.

## Support navigateurs (Décembre 2025)

| API | Chrome | Edge | Firefox | Safari |
|-----|--------|------|---------|--------|
| `requestIdleCallback` | 47+ | 79+ | 55+ | ❌ (flag) |
| `scheduler.postTask` | 94+ | 94+ | 101+ | ❌ |
| `scheduler.yield` | 129+ | 129+ | 142+ | ❌ |

## Composable useScheduler (avec fallbacks universels)

```typescript
// composables/useScheduler.ts
export function useScheduler() {
  const idleCallbackId = ref<number | null>(null)

  /**
   * Exécute une tâche pendant les idle periods du navigateur.
   * Idéal pour analytics, prefetch, warm cache.
   */
  function scheduleIdle(
    callback: () => void,
    options: { timeout?: number } = { timeout: 2000 }
  ) {
    if (idleCallbackId.value !== null) {
      cancelIdleCallback(idleCallbackId.value)
    }

    if ('requestIdleCallback' in window) {
      idleCallbackId.value = requestIdleCallback(callback, options)
    } else {
      // Fallback Safari
      setTimeout(callback, 1)
    }
  }

  /**
   * Schedule avec priorité (Scheduler API).
   * - 'user-blocking': critique, exécution immédiate
   * - 'user-visible': important mais différable
   * - 'background': non-critique
   */
  async function scheduleTask(
    callback: () => any,
    priority: 'user-blocking' | 'user-visible' | 'background' = 'background'
  ): Promise<any> {
    if ('scheduler' in globalThis && 'postTask' in (globalThis as any).scheduler) {
      return (globalThis as any).scheduler.postTask(callback, { priority })
    }

    // Fallback requestIdleCallback pour background
    if (priority === 'background' && 'requestIdleCallback' in window) {
      return new Promise(resolve => {
        requestIdleCallback(() => resolve(callback()))
      })
    }

    // Fallback final setTimeout
    return new Promise(resolve => {
      setTimeout(() => resolve(callback()), priority === 'user-blocking' ? 0 : 1)
    })
  }

  /**
   * Yield au main thread avec continuation prioritaire.
   * Utilise scheduler.yield() si disponible, fallback sur setTimeout.
   */
  function yieldToMain(): Promise<void> {
    if ((globalThis as any).scheduler?.yield) {
      return (globalThis as any).scheduler.yield()
    }
    return new Promise(resolve => setTimeout(resolve, 0))
  }

  onUnmounted(() => {
    if (idleCallbackId.value !== null) {
      cancelIdleCallback(idleCallbackId.value)
    }
  })

  return { scheduleIdle, scheduleTask, yieldToMain }
}
```

## Pattern traitement par chunks avec yield

```typescript
// composables/useChunkedProcessor.ts
export function useChunkedProcessor<T, R>(
  processor: (item: T) => R,
  options: { chunkSize?: number; yieldInterval?: number } = {}
) {
  const { chunkSize = 10, yieldInterval = 50 } = options
  const isProcessing = ref(false)
  const progress = ref(0)

  async function processItems(items: T[]): Promise<R[]> {
    isProcessing.value = true
    progress.value = 0

    const results: R[] = []
    let lastYield = performance.now()

    for (let i = 0; i < items.length; i++) {
      results.push(processor(items[i]))
      progress.value = ((i + 1) / items.length) * 100

      // Yield toutes les ~50ms pour rester sous le seuil long task
      if (performance.now() - lastYield > yieldInterval) {
        await ((globalThis as any).scheduler?.yield?.() ??
               new Promise(r => setTimeout(r, 0)))
        lastYield = performance.now()
      }
    }

    isProcessing.value = false
    return results
  }

  return { processItems, isProcessing, progress }
}
```

## Utilisation dans les lifecycle hooks

```vue
<script setup lang="ts">
const { scheduleIdle, scheduleTask, yieldToMain } = useScheduler()

onMounted(async () => {
  // 1. Critique : initialisation UI immédiate
  initializeCriticalUI()

  // 2. Yield avant travail non-critique
  await yieldToMain()

  // 3. Travail user-visible mais différable
  await scheduleTask(async () => {
    await fetchSecondaryData()
  }, 'user-visible')

  // 4. Background : analytics, prefetch
  scheduleIdle(() => {
    initAnalytics()
    prefetchRelatedContent()
    warmImageCache()
  }, { timeout: 5000 })
})
</script>
```

## Pattern event handler optimisé

```vue
<script setup lang="ts">
const { yieldToMain } = useScheduler()

// ❌ MAL : Tout synchrone
const badHandler = () => {
  heavyComputation()
  updateAnalytics()
  saveToLocalStorage()
  items.value = result
}

// ✅ BIEN : UI d'abord, puis yield et defer
const goodHandler = async () => {
  // 1. Update UI critique immédiatement
  isLoading.value = true

  // 2. Yield pour permettre le paint
  await yieldToMain()

  // 3. Travail non-critique après paint
  requestAnimationFrame(() => {
    setTimeout(() => {
      updateAnalytics()
      saveToLocalStorage()
    }, 0)
  })
}
</script>
```

## Composable useYieldingProcessor (avec annulation)

```typescript
// composables/useYieldingProcessor.ts
export function useYieldingProcessor<T>() {
  const isProcessing = ref(false)
  const progress = ref(0)
  const controller = ref<AbortController | null>(null)

  async function process(
    items: T[],
    processor: (item: T) => void,
    chunkSize: number = 5
  ): Promise<void> {
    controller.value = new AbortController()
    isProcessing.value = true
    progress.value = 0

    try {
      for (let i = 0; i < items.length; i++) {
        if (controller.value.signal.aborted) break

        processor(items[i])
        progress.value = Math.round((i / items.length) * 100)

        if (i % chunkSize === 0 && i > 0) {
          await yieldToMain()
        }
      }
    } finally {
      isProcessing.value = false
      progress.value = 100
    }
  }

  function cancel() {
    controller.value?.abort()
  }

  return { process, cancel, isProcessing, progress }
}

async function yieldToMain(): Promise<void> {
  if ((globalThis as any).scheduler?.yield) {
    return (globalThis as any).scheduler.yield()
  }
  return new Promise(resolve => setTimeout(resolve, 0))
}
```

---
