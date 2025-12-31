# INP (Interaction to Next Paint)

## Debounce et Throttle avec VueUse

## Règle d'or

| Technique | Comportement | Cas d'usage |
|-----------|--------------|-------------|
| **Debounce** | Attend que l'utilisateur arrête | Recherche, validation, autocomplétion |
| **Throttle** | Max 1 exécution par intervalle | Scroll, resize, mousemove |

## Debounce pour inputs utilisateur

```typescript
import { useDebounceFn } from '@vueuse/core'

const searchQuery = ref('')
const results = ref([])

// Debounce : attend 300ms après la dernière frappe
// maxWait : force exécution après 1s max (même si l'utilisateur tape encore)
const debouncedSearch = useDebounceFn(async (query: string) => {
  if (query.length < 2) {
    results.value = []
    return
  }
  results.value = await fetchSearchResults(query)
}, 300, { maxWait: 1000 })

// Déclenché à chaque frappe
watch(searchQuery, (query) => {
  debouncedSearch(query)
})
```

**Options avancées :**

| Option | Effet | Valeur recommandée |
|--------|-------|-------------------|
| `delay` | Délai avant exécution | 300ms (search), 500ms (validation) |
| `maxWait` | Force exécution après délai max | 1000ms (évite attente infinie) |

## Throttle pour événements continus

```typescript
import { useThrottleFn, useEventListener } from '@vueuse/core'

// Throttle : max 1 exécution par 100ms
const throttledScrollHandler = useThrottleFn(() => {
  // Calculs de position, lazy loading, etc.
  updateScrollPosition()
}, 100)

// useEventListener avec passive: true pour performances scroll
useEventListener(window, 'scroll', throttledScrollHandler, { passive: true })
```

## Pattern combiné pour search avec UI feedback

```vue
<script setup lang="ts">
import { useDebounceFn, refDebounced } from '@vueuse/core'

const query = ref('')
const queryDebounced = refDebounced(query, 300)  // Pour affichage
const isSearching = ref(false)
const results = ref([])

// Debounce la fonction de recherche
const performSearch = useDebounceFn(async (q: string) => {
  if (q.length < 2) {
    results.value = []
    return
  }

  isSearching.value = true
  try {
    results.value = await fetchSearchResults(q)
  } finally {
    isSearching.value = false
  }
}, 300, { maxWait: 1000 })

watch(query, performSearch)
</script>

<template>
  <input v-model="query" placeholder="Rechercher..." />
  <p v-if="isSearching">Recherche en cours...</p>
  <p v-else-if="queryDebounced && !results.length">Aucun résultat</p>
  <ul v-else>
    <li v-for="result in results" :key="result.id">{{ result.title }}</li>
  </ul>
</template>
```

---
