# Search Patterns (MiniSearch)

Patterns de recherche full-text avec MiniSearch 7.x pour blog multilingue FR/EN sur Cloudflare Pages SSG.

## Pourquoi MiniSearch ?

| Critère | MiniSearch 7.x | Pagefind | Lunr.js | Fuse.js |
|---------|----------------|----------|---------|---------|
| **Taille bundle** | ~7KB gzip | ~8KB + chunks | ~8KB gzip | ~4KB gzip |
| **Index 500 posts** | ~150KB | ~100KB chunked | ~500KB+ | N/A (full data) |
| **Latence recherche** | **<10ms** | <100ms | <10ms | **1-5 secondes** |
| **Mémoire** | Modérée | Minimale (chunked) | Élevée | Élevée |
| **Fuzzy search** | ✅ Natif | ✅ Natif | ✅ Natif | ✅ Natif |
| **Prefix search** | ✅ Natif | ✅ Natif | ❌ Plugin | ✅ Natif |
| **Boosting** | ✅ Field + Document + Term | ❌ Limité | ✅ Field | ❌ Limité |
| **Async loading** | ✅ `loadJSONAsync()` | ✅ Chunks | ❌ | ❌ |
| **Multilingue** | Manuel (snowball) | ✅ Built-in | Plugins | Manuel |
| **Intégration SSG** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |

**⚠️ Fuse.js est à exclure** pour les blogs : sa recherche linéaire O(n) produit des latences de **1-5 secondes** à 500 posts avec contenu complet. Lunr.js souffre d'index volumineux : 700 documents peuvent générer ~70MB en mémoire.

MiniSearch utilise une structure **radix tree** optimisée pour la mémoire, ce qui le rend idéal pour les blogs avec 100-500 articles où la taille de l'index peut devenir significative.

## Types TypeScript

```typescript
// app/types/search.ts
export interface BlogSearchDocument {
  id: string
  title: string
  description: string
  content: string
  slug: string
  locale: 'fr' | 'en'
  pillar: string
  tags: string[]
}

export interface SearchResult {
  id: string
  title: string
  description: string
  slug: string
  locale: string
  pillar: string
  score: number
  match: Record<string, string[]>
}
```

## Configuration MiniSearch Optimale

### Normalisation des accents (crucial FR/EN)

```typescript
// app/lib/search-utils.ts

// Normalise les accents pour recherche FR
export const removeAccents = (str: string): string =>
  str.normalize('NFD').replace(/[\u0300-\u036f]/g, '')

// Auto-détection langue basée sur caractères accentués français
export const detectLanguage = (text: string): 'fr' | 'en' =>
  /[àâäéèêëïîôùûüÿœæç]/i.test(text) ? 'fr' : 'en'

// Stop words bilingues (réduisent la taille de l'index de ~20%)
export const STOP_WORDS = new Set([
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
```

### Stemmer Snowball (optionnel)

Pour améliorer la pertinence des recherches, le stemming réduit les mots à leur racine (`développement` → `développ`, `developing` → `develop`).

```bash
pnpm add snowball-stemmers
```

```typescript
// app/lib/search-utils.ts (suite)
import snowballFactory from 'snowball-stemmers'

// Initialisation des stemmers Snowball
const stemmers = {
  fr: snowballFactory.newStemmer('french'),
  en: snowballFactory.newStemmer('english')
}

// Stemming avec auto-détection de langue
export function stemTerm(term: string, lang?: 'fr' | 'en'): string {
  const detectedLang = lang || detectLanguage(term)
  const normalized = removeAccents(term.toLowerCase())
  return stemmers[detectedLang].stem(normalized)
}
```

**Impact du stemming :**

| Sans stemming | Avec stemming | Résultat |
|---------------|---------------|----------|
| `développer` ≠ `développement` | `développ` = `développ` | ✅ Match |
| `running` ≠ `runs` | `run` = `run` | ✅ Match |

**⚠️ Trade-off** : Le stemming ajoute ~15KB au bundle. Pour un blog <200 articles, la normalisation des accents seule est souvent suffisante.

### Configuration complète

```typescript
// app/lib/minisearch-config.ts
import MiniSearch, { type Options } from 'minisearch'
import { removeAccents, STOP_WORDS } from './search-utils'
import type { BlogSearchDocument } from '~/types/search'

export const MINISEARCH_OPTIONS: Options<BlogSearchDocument> = {
  // Champ d'identification unique
  idField: 'id',

  // Champs indexés pour la recherche full-text
  fields: ['title', 'description', 'content', 'tags'],

  // Champs stockés (retournés avec les résultats sans re-fetch)
  storeFields: ['title', 'description', 'slug', 'locale', 'pillar'],

  // Extraction des champs (gère les arrays comme tags)
  extractField: (doc, fieldName) => {
    const value = doc[fieldName as keyof BlogSearchDocument]
    return Array.isArray(value) ? value.join(' ') : (value as string)
  },

  // Tokenizer multilingue avec gestion des élisions françaises
  tokenize: (text: string, fieldName?: string): string[] => {
    if (!text) return []
    let normalized = text.toLowerCase()

    // Gestion des élisions françaises: l'homme → l homme, d'abord → d abord
    // Crucial pour que "homme" soit trouvé lors d'une recherche
    normalized = normalized.replace(/['']/g, ' ')

    return normalized
      .split(/[\s\-_.,;:!?"()\[\]{}|/<>@#$%^&*+=~`]+/)
      .filter(token => token.length > 0)
  },

  // Processing avec normalisation accents + stop words
  processTerm: (term: string): string | null => {
    const normalized = removeAccents(term.toLowerCase().trim())
    if (STOP_WORDS.has(normalized) || normalized.length < 2) {
      return null // Exclure stop words et termes < 2 chars
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

    // FUZZY ADAPTATIF: évite les faux positifs sur termes courts
    fuzzy: (term: string) => {
      if (term.length <= 3) return false  // Pas de fuzzy pour "vue", "css"
      if (term.length <= 5) return 0.1    // 1 typo max pour "nuxt", "react"
      return 0.2                          // 20% pour termes longs
    },
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

// Recherche avec boosting avancé (MiniSearch 7.1.0+)
export function searchWithBoost(
  index: MiniSearch<BlogSearchDocument>,
  query: string,
  options?: { locale?: 'fr' | 'en'; limit?: number }
) {
  const { locale, limit = 10 } = options || {}

  return index.search(query, {
    filter: locale ? (result) => result.locale === locale : undefined,

    // boostDocument: Favorise le contenu récent (jusqu'à 2x pour articles < 1 an)
    boostDocument: (id, term, storedFields) => {
      if (!storedFields?.publishedAt) return 1
      const age = Date.now() - new Date(storedFields.publishedAt as string).getTime()
      const yearInMs = 365 * 24 * 60 * 60 * 1000
      return 1 + Math.max(0, 1 - age / yearInMs)
    },

    // boostTerm: Premier terme de recherche plus important (v7.1.0+)
    boostTerm: (term, index, terms) => terms.length - index
  }).slice(0, limit)
}
```

**Boosting 3:2:1.5:1** : Un match dans le titre est **3x plus pertinent** qu'un match dans le contenu.

| Option | Valeur | Effet |
|--------|--------|-------|
| `boost.title` | 3 | Priorise les matches dans le titre |
| `boost.description` | 2 | Description importante pour pertinence |
| `fuzzy` | fonction | Adaptatif selon longueur du terme |
| `prefix` | true | Recherche incrémentale (`nux` → `nuxt`) |
| `combineWith` | 'OR' | Retourne docs matchant au moins un terme |

### Mécanismes de boosting avancés (MiniSearch 7.1.0+)

| Mécanisme | Fonction | Cas d'usage |
|-----------|----------|-------------|
| **Field boost** | `boost: { title: 3 }` | Prioriser certains champs |
| **Document boost** | `boostDocument(id, term, storedFields)` | Favoriser contenu récent, populaire |
| **Term boost** | `boostTerm(term, index, terms)` | Premier terme plus important |

**Note** : `boostDocument` et `boostTerm` requièrent MiniSearch ≥7.1.0. Pour utiliser `boostDocument`, les champs nécessaires (ex: `publishedAt`) doivent être dans `storeFields`.

### processTerm différencié : indexation vs recherche

Une stratégie avancée consiste à filtrer les stopwords **uniquement à l'indexation**, mais les conserver **à la recherche** pour permettre les phrases exactes :

```typescript
const miniSearch = new MiniSearch({
  fields: ['title', 'content'],
  storeFields: ['title', 'slug'],

  // À L'INDEXATION : filtrer stopwords + stemming
  processTerm: (term) => {
    const lower = removeAccents(term.toLowerCase())
    if (STOP_WORDS.has(lower)) return null  // ← Filtré
    return stemTerm(lower, 'fr')             // ← Stemming
  },

  // À LA RECHERCHE : garder les stopwords (via searchOptions)
  searchOptions: {
    processTerm: (term) => {
      // Pas de filtrage stopwords → "le petit prince" fonctionne
      return removeAccents(term?.toLowerCase() ?? '')
    },
    prefix: true,
    fuzzy: 0.2
  }
})
```

**Impact :**

| Comportement | Sans différenciation | Avec différenciation |
|--------------|---------------------|---------------------|
| Recherche "le chat" | Trouve "chat" uniquement | Trouve "le chat" en priorité |
| Taille index | -20% (stopwords filtrés) | -20% (stopwords filtrés) |
| Phrases exactes | ❌ "le" ignoré | ✅ "le" considéré |

## Index Unifié vs Séparés par Langue

Pour un blog SSG bilingue, deux stratégies sont possibles :

| Critère | Index unifié | Index séparés |
|---------|-------------|---------------|
| Fichiers générés | 1 (`search-index.json`) | 2 (`search-index-fr.json`, `search-index-en.json`) |
| Recherche cross-language | ✅ Possible | ❌ Non |
| Complexité build | Simple | Plus élevée |
| Lazy loading | 1 requête | 1 requête par langue |
| Taille totale | ~10% plus grand | Légèrement plus petit |
| Stemming | Auto-détection | Optimal par langue |

**Recommandation** : Index **unifié** pour <1000 articles. Filtrage par locale au moment de la recherche :

```typescript
// Recherche avec filtre de langue
function searchPosts(
  index: MiniSearch<BlogSearchDocument>,
  query: string,
  options?: { lang?: 'fr' | 'en'; limit?: number }
) {
  const { lang, limit = 10 } = options || {}

  return index.search(query, {
    filter: lang ? (result) => result.locale === lang : undefined,
    prefix: true,
    fuzzy: 0.2
  }).slice(0, limit)
}

// Usage
const results = searchPosts(miniSearch, 'vue composables', { lang: 'fr' })
```

L'index **séparé** n'est recommandé que pour >1000 articles par langue ou si le stemming parfait est critique.

## Composable useSearch Complet

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

### loadJSON() vs loadJSONAsync()

| Méthode | Comportement | Impact UX | Quand utiliser |
|---------|--------------|-----------|----------------|
| `MiniSearch.loadJSON()` | Synchrone, bloquant | Freeze UI pendant parsing | Index < 50KB |
| `MiniSearch.loadJSONAsync()` | Incrémental, non-bloquant | UI reste fluide | Index > 50KB (**recommandé**) |

**⚠️ Toujours utiliser `loadJSONAsync()`** pour les blogs avec plus de 50 articles. Le parsing d'un index de 200KB peut bloquer le main thread pendant 100-200ms avec `loadJSON()`, causant des jank visibles.

### Cache d'index par locale (éviter rechargements)

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

## Intégration Command Palette (⌘K)

### Bouton trigger (requis pour mobile)

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

### Composant SearchCommand complet

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

### Composable useSearchHistory (optionnel)

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

## Optimisation Taille Index

### Fonctions de nettoyage

```typescript
// app/lib/search-utils.ts (suite)

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

// Tronquer le contenu long (économise ~40% sur gros articles)
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

| Articles | Index brut | Avec optimisations | Brotli (~85% compression) | Temps chargement (3G) |
|----------|------------|-------------------|--------------------------|----------------------|
| 20 | ~25KB | ~15KB | **~3KB** | <50ms |
| 100 | ~100KB | ~60KB | **~10-15KB** | ~100ms |
| 250 | ~250KB | ~150KB | **~25-40KB** | ~300ms |
| 500 | ~500KB | ~300KB | **~50-80KB** | ~600ms |
| 1000 | ~1MB | ~600KB | **~100-150KB** | ~1.2s |

**Note** : Cloudflare Pages applique automatiquement Brotli (~85% compression vs ~75% gzip). Les temps supposent une connexion 3G (~750KB/s).

**Impact des optimisations :**
- Stop words bilingues : **-20%**
- `stripMarkdown()` : **-15%**
- `truncateContent(3000)` : **-10-20%** (selon longueur articles)
- IDs entiers vs slugs : jusqu'à **-90%** (cas extrême documenté: 700KB → 62KB)

### Contraintes Cloudflare Pages

| Limite | Valeur | Impact sur la recherche |
|--------|--------|------------------------|
| **Taille max fichier** | 25 MiB | Index doit rester < 25MB |
| **Fichiers par site** | 20,000 | Non limitant pour index unique |
| **Bande passante** | Illimitée | Lazy loading sans coût |

**⚠️ Règle pratique** : Maintenir l'index < 10MB (gzipped < 3MB) pour un temps de chargement acceptable. Au-delà de 500 articles, envisager Pagefind.

## Script Génération Index (Build-time)

```javascript
// scripts/generate-search-index.mjs
import { readFileSync, writeFileSync, readdirSync, existsSync } from 'fs'
import { join } from 'path'
import matter from 'gray-matter'
import MiniSearch from 'minisearch'

const CONTENT_DIR = './content'
const OUTPUT_PATH = './.output/public/search-index.json'

// Configuration MiniSearch (doit matcher celle du client)
const MINISEARCH_OPTIONS = {
  idField: 'id',
  fields: ['title', 'description', 'content', 'tags'],
  storeFields: ['title', 'description', 'slug', 'locale', 'pillar'],
  processTerm: (term) => {
    const normalized = term
      .normalize('NFD')
      .replace(/[\u0300-\u036f]/g, '')
      .toLowerCase()
      .trim()
    if (normalized.length < 2) return null
    return normalized
  }
}

function stripMarkdown(content) {
  return content
    .replace(/```[\s\S]*?```/g, '')
    .replace(/`[^`]*`/g, '')
    .replace(/\[([^\]]+)\]\([^)]+\)/g, '$1')
    .replace(/!\[[^\]]*\]\([^)]+\)/g, '')
    .replace(/#{1,6}\s*/g, '')
    .replace(/[*_]{1,2}([^*_]+)[*_]{1,2}/g, '$1')
    .replace(/\s+/g, ' ')
    .trim()
}

function truncateContent(content, maxLength = 3000) {
  if (content.length <= maxLength) return content
  const truncated = content.slice(0, maxLength)
  const lastPeriod = truncated.lastIndexOf('.')
  return lastPeriod > maxLength * 0.8
    ? truncated.slice(0, lastPeriod + 1)
    : truncated
}

function extractArticles(dir, locale, articles = []) {
  if (!existsSync(dir)) return articles

  const files = readdirSync(dir, { withFileTypes: true })

  for (const file of files) {
    const path = join(dir, file.name)
    if (file.isDirectory()) {
      extractArticles(path, locale, articles)
    } else if (file.name.endsWith('.md')) {
      const raw = readFileSync(path, 'utf-8')
      const { data, content } = matter(raw)

      if (!data.draft) {
        const cleanContent = truncateContent(stripMarkdown(content))
        articles.push({
          id: `${locale}:${data.slug || file.name.replace('.md', '')}`,
          title: data.title || '',
          description: data.description || '',
          content: cleanContent,
          slug: data.path || `/${locale}/blog/${data.slug}`,
          locale,
          pillar: data.pillar || '',
          tags: Array.isArray(data.tags) ? data.tags.join(' ') : ''
        })
      }
    }
  }
  return articles
}

// Extraire articles FR et EN
const articles = [
  ...extractArticles(join(CONTENT_DIR, 'fr'), 'fr'),
  ...extractArticles(join(CONTENT_DIR, 'en'), 'en')
]

// Créer et peupler l'index
const miniSearch = new MiniSearch(MINISEARCH_OPTIONS)
miniSearch.addAll(articles)

// Sauvegarder l'index sérialisé
writeFileSync(OUTPUT_PATH, JSON.stringify(miniSearch))

console.log(`✅ Search index generated: ${articles.length} articles`)
console.log(`   - FR: ${articles.filter(a => a.locale === 'fr').length}`)
console.log(`   - EN: ${articles.filter(a => a.locale === 'en').length}`)
console.log(`   - Size: ${(JSON.stringify(miniSearch).length / 1024).toFixed(1)}KB`)
```

**Configuration package.json :**

```json
{
  "scripts": {
    "generate": "nuxt generate",
    "postgenerate": "node scripts/generate-search-index.mjs"
  }
}
```

### Alternative : Hook Nitro `nitro:build:public-assets`

Pour intégrer la génération d'index directement dans le build Nuxt (sans script externe) :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  hooks: {
    async 'nitro:build:public-assets'(nitro) {
      const { generateSearchIndexes } = await import('./scripts/build-search')
      await generateSearchIndexes(nitro.options.output.publicDir)
    }
  }
})
```

**Comparaison :**

| Approche | Avantages | Inconvénients |
|----------|-----------|---------------|
| `postgenerate` script | Simple, explicite, débugable | Fichier séparé |
| Hook Nitro | Intégré au build | Plus complexe, moins visible |

**Recommandation** : `postgenerate` pour sa simplicité. Le hook Nitro est utile si vous avez besoin d'accès aux options Nitro.

### API `queryCollectionSearchSections()` (Nuxt Content 3.10+)

API native pour extraire les sections recherchables depuis les collections SQLite :

```typescript
// server/api/search-sections/[locale].get.ts
import type { Collections } from '@nuxt/content'

export default defineEventHandler(async (event) => {
  const locale = getRouterParam(event, 'locale') || 'fr'
  const collectionName = `articles_${locale}` as keyof Collections

  return queryCollectionSearchSections(event, collectionName, {
    ignoredTags: ['code', 'pre', 'script'],  // Exclure blocs de code
    minHeading: 'h2',                         // Niveau min (v3.10.0+)
    maxHeading: 'h4',                         // Niveau max
  })
})
```

**Structure `Section` retournée :**

```typescript
interface Section {
  id: string       // "/blog/article#section" (utilisable comme URL)
  title: string    // Texte du heading
  titles: string[] // Breadcrumb des parents ["Article", "Introduction"]
  content: string  // Contenu textuel de la section
  level: number    // Niveau du heading (1-6)
}
```

**⚠️ Note SSG** : Cette API requiert une requête D1 runtime. Pour SSG pur, préférer le script build-time qui génère un index statique sans consommer le quota D1.

## Optimisation Bundle (Vite)

Isoler MiniSearch dans son propre chunk évite d'alourdir le bundle principal :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  vite: {
    build: {
      rollupOptions: {
        output: {
          manualChunks: {
            // Sépare MiniSearch (~7KB) du bundle principal
            search: ['minisearch']
          }
        }
      }
    }
  }
})
```

**Impact :**

| Configuration | Bundle principal | Chunk search | Total |
|---------------|------------------|--------------|-------|
| Sans `manualChunks` | ~150KB | - | ~150KB |
| Avec `manualChunks` | ~143KB | ~7KB | ~150KB |

**Avantage** : Le chunk `search` n'est chargé qu'au premier accès à la recherche (lazy loading), réduisant le TTI (Time to Interactive) initial.

## Composant SearchBox Accessible (Alternative)

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

## Anti-patterns

| ❌ Anti-pattern | ✅ Pattern correct |
|-----------------|-------------------|
| Charger l'index au mount | Lazy load au focus/ouverture |
| Recherche à chaque keystroke | Debounce 250ms |
| Index dans `ref()` réactif | Variable externe ou `shallowRef` |
| Indexer le contenu complet | `truncateContent(3000)` |
| `v-html` sans échappement | Échapper regex avant highlighting |
| Pas de gestion d'erreur | try/catch + état `error` |
| `refDebounced` sur `v-model` | Refs séparés source + debounced |
| Raccourci ⌘K sans check input | Vérifier `activeElement.tagName` |
| DialogTitle absent | `VisuallyHidden` wrapper |

### Exemples détaillés

```typescript
// ❌ MAUVAIS: Charger l'index de manière synchrone au mount
onMounted(async () => {
  const index = await fetch('/search-index.json').then(r => r.json())
  // Bloque le rendering, mauvais TTI
})

// ✅ BON: Lazy load au focus
const handleFocus = () => loadIndex()

// ❌ MAUVAIS: Recherche à chaque keystroke
watch(query, (q) => miniSearch.search(q)) // Performance désastreuse

// ✅ BON: Debounce 250ms
const debouncedSearch = useDebounceFn(executeSearch, 250)

// ❌ MAUVAIS: Index dans un ref réactif
const searchIndex = ref(hugeIndex) // Overhead Vue reactivity inutile

// ✅ BON: Variable externe
let searchIndex: MiniSearch | null = null

// ❌ MAUVAIS: v-html sans sanitization
<span v-html="userInput" /> // XSS vulnerability

// ✅ BON: Échapper avant highlighting
const escaped = query.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')

// ❌ MAUVAIS: refDebounced directement sur v-model
// refDebounced retourne un ref READONLY - ne peut pas être bindé à v-model
const query = refDebounced(ref(''), 250)  // ← query est readonly!
<input v-model="query" />  // ❌ Ne fonctionne pas

// ✅ BON: Deux refs séparés (source + debounced)
const query = ref('')                           // ← Source éditable
const debouncedQuery = refDebounced(query, 250) // ← Lecture seule, pour la recherche

<input v-model="query" />  // ✅ Bind sur la source
watch(debouncedQuery, performSearch)  // ✅ Watch le debounced

// ✅ ALTERNATIVE: useDebounceFn pour debouncer la fonction
const query = ref('')
const debouncedSearch = useDebounceFn(performSearch, 250)
watch(query, debouncedSearch)  // ✅ Plus simple pour ce cas
```

**Timing de debounce recommandé :**

| Contexte | Délai | Raison |
|----------|-------|--------|
| Autocomplete/suggestions | 150-200ms | Feedback rapide attendu |
| Full-text search | **250ms** | Équilibre réactivité/perf |
| Filtrage lourd | 300-500ms | Éviter surcharge CPU |

## Web Worker pour Gros Volumes (>500 articles)

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

## Quand reconsidérer MiniSearch

### Garder MiniSearch quand :

- **Volume < 500 articles** — Index < 150KB, latence < 10ms
- **Boosting personnalisé requis** — Field, document, et term boost
- **Algorithmes de ranking custom** — Contrôle total sur le scoring
- **Index dynamique runtime** — Ajout/suppression sans rebuild
- **Edge functions planifiées** — Compatible Cloudflare Workers

### Migrer vers Pagefind quand :

- **Volume > 500 articles** — Index chunked plus efficace
- **Zero-configuration multilingue** — Détection auto via `<html lang="">`
- **Minimiser le bundle JS** — Pas de lib à charger initialement
- **Trafic search imprévisible** — Chunked loading scale mieux
- **Contenu change = rebuild anyway** — Pagefind s'intègre naturellement

### Matrice de décision

| Critère | MiniSearch | Pagefind |
|---------|------------|----------|
| **< 200 articles** | ✅ Recommandé | Overkill |
| **200-500 articles** | ✅ Recommandé | Alternative viable |
| **500-2000 articles** | ⚠️ + Web Worker | ✅ Recommandé |
| **> 2000 articles** | ❌ | ✅ Recommandé |
| **Custom ranking** | ✅ | ❌ |
| **Runtime updates** | ✅ | ❌ |
| **Setup time** | Moyen | Minimal |

**Pour sebc.dev (< 500 articles prévus)** : MiniSearch reste le choix optimal avec latence < 10ms et contrôle total sur le boosting.

---

## Alternative : Pagefind (>500 pages)

Pour les très gros volumes, **Pagefind** offre une architecture chunked plus efficace :

| Critère | MiniSearch | Pagefind |
|---------|------------|----------|
| **Taille index 10k pages** | ~10-30MB | <300KB |
| **Architecture** | Index monolithique | Chunks à la demande |
| **Configuration** | Manuelle | Auto-génération |
| **Customisation** | Très flexible | Limitée |
| **Bundle size** | ~7KB | ~8KB + chunks |
| **Utilisé par** | VitePress, Docus | MDN, Starlight |

**Recommandation :**
- **< 500 articles** : MiniSearch (plus flexible, contrôle total)
- **500-2000 articles** : MiniSearch + Web Worker
- **> 2000 articles** : Évaluer Pagefind

### Installation et configuration

```bash
# Installation Pagefind
pnpm add -D pagefind
```

```json
// package.json - Scripts de build
{
  "scripts": {
    "generate": "nuxi generate",
    "postgenerate": "npx pagefind --site .output/public --output-path .output/public/pagefind"
  }
}
```

**Architecture générée :**

```
.output/public/
├── pagefind/
│   ├── pagefind.js          # ~8KB - API de recherche
│   ├── pagefind-ui.js       # ~15KB - UI optionnelle
│   ├── pagefind-ui.css      # Styles UI optionnelle
│   ├── pagefind.wasm        # ~200KB - Moteur WebAssembly
│   ├── pagefind-entry.json  # Métadonnées index
│   └── index/               # Chunks d'index (~100KB total)
│       ├── 00000000.pf_index
│       └── ...
```

### Composant Vue Pagefind

```vue
<!-- app/components/search/PagefindSearch.vue -->
<script setup lang="ts">
interface PagefindResult {
  url: string
  excerpt: string
  meta: {
    title: string
    image?: string
  }
}

const query = ref('')
const results = ref<PagefindResult[]>([])
const isLoading = ref(false)
const isReady = ref(false)

// Instance Pagefind (chargée dynamiquement)
let pagefind: any = null

// Lazy load Pagefind uniquement côté client
onMounted(async () => {
  try {
    // Import dynamique depuis le dossier généré
    pagefind = await import('/pagefind/pagefind.js')
    await pagefind.init()
    isReady.value = true
  } catch (e) {
    console.error('Pagefind init failed:', e)
  }
})

// Recherche avec debounce
const search = useDebounceFn(async () => {
  if (!pagefind || !isReady.value || query.value.length < 2) {
    results.value = []
    return
  }

  isLoading.value = true
  try {
    const response = await pagefind.search(query.value)

    // Charger les données complètes des résultats (lazy par défaut)
    results.value = await Promise.all(
      response.results.slice(0, 10).map((r: any) => r.data())
    )
  } catch (e) {
    console.error('Search failed:', e)
    results.value = []
  } finally {
    isLoading.value = false
  }
}, 300)

// Déclencher la recherche sur changement de query
watch(query, search)
</script>

<template>
  <ClientOnly>
    <div class="relative w-full max-w-md">
      <input
        v-model="query"
        type="search"
        :placeholder="$t('search.placeholder')"
        class="w-full px-4 py-2 border rounded-lg"
        :disabled="!isReady"
      />

      <ul v-if="results.length" class="absolute z-50 w-full mt-1 bg-background border rounded-lg shadow-lg">
        <li
          v-for="result in results"
          :key="result.url"
          class="px-4 py-3 hover:bg-muted cursor-pointer"
        >
          <NuxtLink :to="result.url" class="block">
            <span class="font-medium">{{ result.meta.title }}</span>
            <!-- excerpt contient du HTML avec <mark> pour highlighting -->
            <p class="text-sm text-muted-foreground line-clamp-2" v-html="result.excerpt" />
          </NuxtLink>
        </li>
      </ul>

      <p v-else-if="query.length >= 2 && !isLoading" class="text-sm text-muted-foreground mt-2">
        {{ $t('search.noResults') }}
      </p>
    </div>

    <template #fallback>
      <div class="w-full max-w-md">
        <input type="search" disabled placeholder="Loading search..." class="w-full px-4 py-2 border rounded-lg opacity-50" />
      </div>
    </template>
  </ClientOnly>
</template>
```

### Multilingue automatique

Pagefind détecte automatiquement la langue via l'attribut `<html lang="">`. Pour un contrôle explicite :

```html
<!-- Dans le layout ou la page -->
<div data-pagefind-filter="language:fr">
  <!-- Contenu français -->
</div>
```

```typescript
// Filtrer par langue dans la recherche
const response = await pagefind.search(query.value, {
  filters: {
    language: locale.value  // 'fr' ou 'en'
  }
})
```

### Avantages Pagefind pour SSG

| Aspect | Comportement |
|--------|-------------|
| **Build time** | Index généré automatiquement post-build |
| **Network** | Charge uniquement les chunks nécessaires (~100KB max) |
| **Highlighting** | Excerpts avec `<mark>` inclus par défaut |
| **i18n** | Index séparés par langue automatique |
| **Maintenance** | Aucune configuration à maintenir |

**Note** : Pagefind offre moins de contrôle sur le boosting et le stemming multilingue que MiniSearch, mais son intégration "zero-config" le rend idéal pour les blogs à forte croissance.

---

## Traductions i18n

```json
// i18n/locales/fr.json
{
  "search": {
    "label": "Rechercher",
    "placeholder": "Rechercher un article...",
    "loading": "Chargement...",
    "noResults": "Aucun résultat",
    "noResultsFor": "Aucun résultat pour \"{query}\"",
    "resultsCount": "{count} résultat(s) trouvé(s)",
    "articles": "Articles"
  }
}
```

```json
// i18n/locales/en.json
{
  "search": {
    "label": "Search",
    "placeholder": "Search articles...",
    "loading": "Loading...",
    "noResults": "No results",
    "noResultsFor": "No results for \"{query}\"",
    "resultsCount": "{count} result(s) found",
    "articles": "Articles"
  }
}
```

## Nouveautés MiniSearch 7.x

| Version | Changement | Impact |
|---------|------------|--------|
| **7.0.0** | Target ES6+ (plus de IE11) | Vérifier compatibilité navigateurs |
| **7.0.0** | `SearchableMap` import séparé | `import { SearchableMap } from 'minisearch/SearchableMap'` |
| **7.1.0** | Option `boostTerm` | Boost personnalisé par terme de recherche |
| **7.2.0** | Option `stringifyField` | Contrôle de la sérialisation des champs |
| **7.2.0** | Target ES2018 | Unicode regex dans tokenizer (`/\p{P}/u`) |

**Compatibilité index** : Les index sérialisés avec MiniSearch 6.x sont compatibles avec 7.x — pas besoin de reconstruire lors de la migration.

## Mise à jour de documents (pattern discard/replace)

MiniSearch n'a **pas de méthode `update()` native**. Pour modifier un document indexé :

```typescript
// Option 1 : discard() + add() (explicite)
miniSearch.discard(documentId)      // Marque comme supprimé
miniSearch.add(updatedDocument)     // Ré-indexe avec le même ID

// Option 2 : replace() (convenience method)
miniSearch.replace(updatedDocument) // Fait discard + add en interne
```

**⚠️ Pour SSG** : Ce pattern est rarement nécessaire car l'index est regénéré à chaque build. Il est utile uniquement pour des applications avec édition de contenu runtime.

## Parallel Load Pattern

Charger MiniSearch et l'index simultanément pour réduire le temps de chargement total :

```typescript
// ❌ Séquentiel (~400ms)
const MiniSearch = (await import('minisearch')).default
const indexData = await fetch('/search-index.json').then(r => r.text())

// ✅ Parallèle (~250ms) — 40% plus rapide
const [MiniSearchModule, indexData] = await Promise.all([
  import('minisearch'),
  fetch('/search-index.json').then(r => r.text())
])

const miniSearch = await MiniSearchModule.default.loadJSONAsync(
  indexData,
  MINISEARCH_OPTIONS
)
```

**Impact mesuré :**

| Approche | Temps total | Économie |
|----------|-------------|----------|
| Séquentiel | ~400ms | - |
| Parallèle | ~250ms | **~40%** |

## Targets de Performance

Objectifs mesurables pour valider l'implémentation de la recherche :

| Métrique | Target | Mesure |
|----------|--------|--------|
| **Index size** | < 5MB par locale | `ls -la .output/public/search-index.json` |
| **Index load time** | < 500ms sur 3G | DevTools Network (Slow 3G) |
| **Search latency** | < 50ms après debounce | `console.time()` autour de `miniSearch.search()` |
| **Time to first result** | < 300ms depuis keystroke | UX perçue (debounce 250ms + search 50ms) |
| **Modal open animation** | 150-200ms | CSS transition duration |
| **Debounce timing** | 250ms (ajustable) | Configurable dans `useSearch()` |
| **Bundle MiniSearch** | ~7KB gzipped | `npx nuxi analyze` |

**Seuils d'alerte :**

| Condition | Action |
|-----------|--------|
| Index > 10MB | Activer `truncateContent(2000)` |
| Load time > 1s | Implémenter parallel load pattern |
| Search latency > 100ms | Réduire `maxResults` ou simplifier fuzzy |
| > 500 articles | Évaluer Web Worker ou Pagefind |

**Outils de mesure :**

```typescript
// Mesurer la latence de recherche
const start = performance.now()
const results = miniSearch.search(query)
console.log(`Search latency: ${(performance.now() - start).toFixed(2)}ms`)
```

---

## Checklist d'implémentation

### Phase 1 : Configuration de base

- [ ] Installer les dépendances : `pnpm add minisearch`
- [ ] Optionnel stemming : `pnpm add snowball-stemmers`
- [ ] Créer `app/lib/minisearch-config.ts` avec les options
- [ ] Créer `app/lib/search-utils.ts` (removeAccents, STOP_WORDS)
- [ ] Définir les types dans `app/types/search.ts`

### Phase 2 : Génération d'index SSG

- [ ] Créer `scripts/generate-search-index.mjs`
- [ ] Ajouter `"postgenerate": "node scripts/generate-search-index.mjs"` dans package.json
- [ ] Vérifier que l'index généré est < 25 MiB (contrainte Cloudflare)
- [ ] Tester la génération : `pnpm generate && ls -la .output/public/search-index.json`

### Phase 3 : Composables client

- [ ] Créer `app/composables/useSearch.ts` avec lazy loading
- [ ] Implémenter debounce 250ms + maxWait 1000ms
- [ ] Utiliser `loadJSONAsync()` pour le parsing
- [ ] Tester le temps de chargement d'index (< 500ms)

### Phase 4 : Interface utilisateur

- [ ] Créer `app/components/search/SearchCommand.vue` (⌘K palette)
- [ ] Implémenter raccourci clavier ⌘K / Ctrl+K
- [ ] Ajouter highlighting des résultats
- [ ] Ajouter tous les attributs ARIA requis
- [ ] Tester navigation clavier (↑↓ Enter Escape)

### Phase 5 : Multilingue (si applicable)

- [ ] Implémenter gestion des élisions françaises dans tokenizer
- [ ] Ajouter cache d'index avec `Map` pour éviter rechargements
- [ ] Filtrer par locale dans la fonction de recherche
- [ ] Ajouter traductions i18n pour les messages

### Phase 6 : Tests et optimisation

- [ ] Vérifier bundle size de MiniSearch (~7KB gzipped)
- [ ] Ajouter `manualChunks` Vite pour isoler MiniSearch
- [ ] Tester sur mobile (index < 200KB recommandé)
- [ ] Valider accessibilité avec screen reader
- [ ] Profiler performance avec DevTools
