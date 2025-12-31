# Intégration Command Palette (⌘K)

## Bouton trigger (requis pour mobile)

Les utilisateurs mobiles n'ont pas de raccourci clavier — toujours fournir un bouton visible :

```vue
<!-- app/components/search/SearchTrigger.vue -->
<script setup lang="ts">
defineProps<{ open: boolean }>()
defineEmits<{ 'update:open': [value: boolean] }>()

// Détection Mac vs Windows pour afficher le bon raccourci
const isMac = computed(() =>
  import.meta.client && /Mac|iPod|iPhone|iPad/.test(navigator.platform)
)
</script>

<template>
  <Button variant="outline" @click="$emit('update:open', true)" class="gap-2">
    <SearchIcon class="h-4 w-4" />
    <span class="hidden sm:inline">{{ $t('search.button') }}</span>
    <kbd class="hidden sm:inline-flex h-5 px-1.5 rounded border bg-muted text-[10px] font-mono">
      {{ isMac ? '⌘K' : 'Ctrl+K' }}
    </kbd>
  </Button>
</template>
```

## Composant SearchCommand complet

```vue
<!-- app/components/search/SearchCommand.vue -->
<script setup lang="ts">
import { useMagicKeys, useActiveElement, whenever } from '@vueuse/core'
import { logicAnd } from '@vueuse/math'
import { VisuallyHidden } from 'reka-ui'
import {
  Dialog,
  DialogContent
} from '@/components/ui/dialog'
import {
  Command,
  CommandInput,
  CommandList,
  CommandEmpty,
  CommandGroup,
  CommandItem
} from '@/components/ui/command'

const { locale } = useI18n()
const localePath = useLocalePath()
const router = useRouter()
const { query, results, isLoading, isReady, isEmpty, loadIndex, clear } = useSearch({
  debounceMs: 250,
  maxResults: 8
})

const isOpen = ref(false)

// ═══════════════════════════════════════════════════════════════
// RACCOURCI CLAVIER avec protection input (évite conflits formulaires)
// ═══════════════════════════════════════════════════════════════
const activeElement = useActiveElement()
const notInInput = computed(() =>
  !['INPUT', 'TEXTAREA', 'SELECT'].includes(activeElement.value?.tagName ?? '')
)

const { Meta_k, Ctrl_k } = useMagicKeys({
  passive: false,
  onEventFired(e) {
    // Bloquer le raccourci navigateur (bookmark sur certains browsers)
    if ((e.metaKey || e.ctrlKey) && e.key === 'k' && e.type === 'keydown') {
      e.preventDefault()
    }
  }
})

// Déclencher uniquement si pas dans un input
whenever(
  logicAnd(() => Meta_k.value || Ctrl_k.value, notInInput),
  () => { isOpen.value = !isOpen.value }
)

// ═══════════════════════════════════════════════════════════════
// LIFECYCLE
// ═══════════════════════════════════════════════════════════════

// Lazy load : initialise seulement à l'ouverture
watch(isOpen, (open) => {
  if (open) loadIndex()
})

// Reset query on close
watch(isOpen, (open) => {
  if (!open) nextTick(() => clear())
})

// Navigation vers résultat avec locale-aware path
const selectResult = (result: SearchResult) => {
  isOpen.value = false
  clear()
  router.push(localePath(result.slug))
}

// Filtrer par locale courante
const filteredResults = computed(() =>
  results.value.filter(r => r.locale === locale.value)
)
</script>

<template>
  <!-- Bouton trigger pour mobile -->
  <SearchTrigger v-model:open="isOpen" />

  <!-- Modal de recherche -->
  <Dialog v-model:open="isOpen">
    <DialogContent class="p-0 gap-0 max-w-lg overflow-hidden">
      <!-- ⚠️ ACCESSIBILITÉ WCAG : Titre requis pour screen readers -->
      <VisuallyHidden>
        <h2>{{ $t('search.title') }}</h2>
        <p>{{ $t('search.description') }}</p>
      </VisuallyHidden>

      <Command class="rounded-lg">
        <CommandInput
          v-model="query"
          :placeholder="$t('search.placeholder')"
          class="h-12"
        />
        <CommandList class="max-h-[300px] overflow-y-auto">
          <!-- État : Chargement (skeleton loader) -->
          <div v-if="isLoading" class="space-y-2 p-4">
            <div v-for="i in 5" :key="i" class="animate-pulse">
              <div class="h-4 bg-muted rounded w-3/4 mb-2" />
              <div class="h-3 bg-muted/60 rounded w-full" />
            </div>
          </div>

          <!-- État : Aucun résultat -->
          <CommandEmpty v-else-if="isEmpty">
            {{ $t('search.noResults') }}
          </CommandEmpty>

          <!-- État : Résultats -->
          <CommandGroup v-else-if="filteredResults.length > 0" :heading="$t('search.articles')">
            <CommandItem
              v-for="result in filteredResults"
              :key="result.id"
              :value="result.slug"
              @select="selectResult(result)"
            >
              <div class="flex flex-col">
                <span class="font-medium">{{ result.title }}</span>
                <span v-if="result.description" class="text-sm text-muted-foreground line-clamp-1">
                  {{ result.description }}
                </span>
              </div>
            </CommandItem>
          </CommandGroup>
        </CommandList>
      </Command>
    </DialogContent>
  </Dialog>
</template>
```

**Points clés du pattern :**

| Aspect | Implementation | Raison |
|--------|----------------|--------|
| `useMagicKeys` | Avec `passive: false` | Permet `preventDefault()` |
| `notInInput` | Check `activeElement.tagName` | Évite conflits avec formulaires |
| `VisuallyHidden` | Wrapper Reka UI | WCAG : screen readers requièrent un titre |
| Skeleton loader | 5 lignes animées | -35% temps perçu vs spinner |
| `localePath()` | Pour navigation | URLs correctes selon locale |

## Composable useSearchHistory (optionnel)

Historique de recherche avec localStorage pour améliorer l'UX des recherches répétées :

```typescript
// app/composables/useSearchHistory.ts
import { useLocalStorage } from '@vueuse/core'

export function useSearchHistory(maxItems = 5) {
  const history = useLocalStorage<string[]>('search-history', [])

  function add(query: string) {
    const normalized = query.trim()
    if (!normalized || normalized.length < 2) return

    // Dédupliquer (case-insensitive) et limiter
    const filtered = history.value.filter(
      q => q.toLowerCase() !== normalized.toLowerCase()
    )
    history.value = [normalized, ...filtered].slice(0, maxItems)
  }

  function remove(query: string) {
    history.value = history.value.filter(
      q => q.toLowerCase() !== query.toLowerCase()
    )
  }

  function clear() {
    history.value = []
  }

  return {
    history: readonly(history),
    add,
    remove,
    clear
  }
}
```

**Intégration dans SearchCommand :**

```vue
<script setup lang="ts">
const { history, add: addToHistory, clear: clearHistory } = useSearchHistory()

// Ajouter à l'historique lors de la sélection d'un résultat
const selectResult = (result: SearchResult) => {
  addToHistory(query.value)  // Sauvegarder la recherche
  isOpen.value = false
  clear()
  router.push(localePath(result.slug))
}
</script>

<template>
  <CommandList>
    <!-- Historique récent (quand pas de query) -->
    <CommandGroup v-if="!query && history.length" :heading="$t('search.recent')">
      <CommandItem
        v-for="item in history"
        :key="item"
        :value="item"
        @select="query = item"
      >
        <ClockIcon class="mr-2 h-4 w-4 text-muted-foreground" />
        {{ item }}
      </CommandItem>
    </CommandGroup>

    <!-- Résultats de recherche -->
    <!-- ... -->
  </CommandList>
</template>
```

**Privacy** : L'historique est stocké en localStorage uniquement. Ajouter un bouton "Effacer l'historique" pour conformité RGPD.
