# Composant SearchBox Accessible (Alternative)

Pour une recherche hors command palette, avec accessibilité WCAG complète :

```vue
<!-- app/components/search/SearchBox.vue -->
<script setup lang="ts">
import { useFocusTrap, onKeyStroke } from '@vueuse/core'

const props = defineProps<{
  placeholder?: string
}>()

const emit = defineEmits<{
  select: [result: SearchResult]
}>()

const { query, results, isLoading, isEmpty, hasResults, loadIndex, clear } = useSearch({
  debounceMs: 250,
  maxResults: 8
})

// État UI
const isOpen = ref(false)
const selectedIndex = ref(-1)
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

onKeyStroke('Escape', closeSearch)

// Reset selection on results change
watch(results, () => {
  selectedIndex.value = -1
})

function closeSearch() {
  isOpen.value = false
  selectedIndex.value = -1
  deactivate()
}

// Highlighting sécurisé (prévient XSS)
function highlightMatch(text: string, searchQuery: string): string {
  if (!searchQuery || searchQuery.length < 2) return text
  // Échapper les caractères regex spéciaux
  const escaped = searchQuery.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
  const regex = new RegExp(`(${escaped})`, 'gi')
  return text.replace(regex, '<mark class="bg-yellow-200 dark:bg-yellow-800 rounded px-0.5">$1</mark>')
}

// Click outside
onClickOutside(containerRef, closeSearch)
</script>

<template>
  <div ref="containerRef" class="relative w-full max-w-md" role="search">
    <!-- Input -->
    <label for="search-input" class="sr-only">{{ $t('search.label') }}</label>
    <input
      id="search-input"
      v-model="query"
      type="search"
      role="combobox"
      :placeholder="placeholder ?? $t('search.placeholder')"
      :aria-expanded="isOpen && hasResults"
      aria-controls="search-results"
      aria-autocomplete="list"
      :aria-activedescendant="selectedIndex >= 0 ? `result-${selectedIndex}` : undefined"
      class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-primary focus:outline-none"
      @focus="handleFocus"
    />

    <!-- Loading indicator -->
    <div v-if="isLoading" class="absolute right-3 top-2.5">
      <svg class="animate-spin h-5 w-5 text-muted-foreground" viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
      </svg>
    </div>

    <!-- Results dropdown -->
    <ul
      v-show="isOpen && (hasResults || isEmpty)"
      id="search-results"
      role="listbox"
      :aria-label="$t('search.results')"
      class="absolute z-50 w-full mt-1 bg-background border rounded-lg shadow-lg max-h-96 overflow-y-auto"
    >
      <!-- Empty state -->
      <li v-if="isEmpty" class="px-4 py-3 text-muted-foreground" role="status">
        {{ $t('search.noResultsFor', { query }) }}
      </li>

      <!-- Results -->
      <li
        v-for="(result, index) in results"
        :id="`result-${index}`"
        :key="result.id"
        role="option"
        :aria-selected="selectedIndex === index"
        :class="[
          'px-4 py-3 cursor-pointer border-b last:border-b-0 transition-colors',
          selectedIndex === index
            ? 'bg-accent'
            : 'hover:bg-muted'
        ]"
        @click="emit('select', result); closeSearch()"
        @mouseenter="selectedIndex = index"
      >
        <div
          class="font-medium"
          v-html="highlightMatch(result.title, query)"
        />
        <div
          v-if="result.description"
          class="text-sm text-muted-foreground line-clamp-2 mt-1"
          v-html="highlightMatch(result.description, query)"
        />
      </li>
    </ul>

    <!-- Live region for screen readers -->
    <div aria-live="polite" aria-atomic="true" class="sr-only">
      <span v-if="isLoading">{{ $t('search.loading') }}</span>
      <span v-else-if="hasResults">{{ $t('search.resultsCount', { count: results.length }) }}</span>
      <span v-else-if="isEmpty">{{ $t('search.noResults') }}</span>
    </div>
  </div>
</template>
```

**Attributs ARIA essentiels :**

| Attribut | Élément | Rôle |
|----------|---------|------|
| `role="search"` | Container | Identifie la région de recherche |
| `role="combobox"` | Input | Pattern ARIA combobox |
| `aria-expanded` | Input | Indique si dropdown est ouvert |
| `aria-activedescendant` | Input | Pointe vers l'option sélectionnée |
| `role="listbox"` | Liste résultats | Container d'options |
| `role="option"` | Chaque résultat | Option sélectionnable |
| `aria-selected` | Option | Indique la sélection courante |
| `aria-live="polite"` | Live region | Annonce les changements aux lecteurs d'écran |
