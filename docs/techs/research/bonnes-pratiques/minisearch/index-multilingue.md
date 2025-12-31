# Recherche full-text multilingue avec MiniSearch 7.x et Nuxt Content 3.10+

La combinaison MiniSearch 7.x + Nuxt Content 3.10+ offre une solution robuste pour implémenter une recherche full-text côté client dans un contexte Nuxt 4.x SSG multilingue. L'architecture recommandée repose sur des **index séparés par langue**, générés au build time et chargés paresseusement côté client. Cette approche optimise la mémoire, permet des analyseurs linguistiques spécifiques (stemmers, stopwords), et s'intègre parfaitement avec Cloudflare Pages en restant sous la limite de **25 MiB** par fichier.

---

## Configuration MiniSearch 7.x multilingue

MiniSearch est **language-agnostic par design** — il ne fournit pas de stemmers ni stopwords intégrés. Vous devez intégrer des bibliothèques externes comme `snowball-stemmers` (24+ langues) et `stopword` (62 langues).

### Configuration TypeScript complète

```typescript
// lib/search/minisearch-config.ts
import MiniSearch, { type Options } from 'minisearch'
import snowballFactory from 'snowball-stemmers'
import { eng, fra } from 'stopword'

// Types
export interface SearchDocument {
  id: string
  title: string
  content: string
  path: string
  excerpt?: string
  locale: 'fr' | 'en'
}

// Stemmers par langue
const stemmers = {
  en: snowballFactory.newStemmer('english'),
  fr: snowballFactory.newStemmer('french'),
} as const

// Stopwords avec lookup O(1)
const stopwords = {
  en: new Set(eng),
  fr: new Set(fra),
} as const

// Normalisation des accents (crucial pour le français)
function removeAccents(str: string): string {
  return str.normalize('NFD').replace(/[\u0300-\u036f]/g, '')
}

// Configuration MiniSearch par locale
export function createMiniSearchOptions(locale: 'fr' | 'en'): Options<SearchDocument> {
  return {
    idField: 'id',
    fields: ['title', 'content', 'excerpt'],
    storeFields: ['title', 'path', 'excerpt'],
    
    // Tokenisation avec gestion des élisions françaises (l'homme → l homme)
    tokenize: (text) => {
      if (!text) return []
      let normalized = text.toLowerCase()
      if (locale === 'fr') {
        normalized = normalized.replace(/['']/g, ' ')
      }
      normalized = removeAccents(normalized)
      return normalized.split(/[\s\p{P}]+/u).filter(t => t.length > 0)
    },
    
    // Traitement des termes: stopwords + stemming
    processTerm: (term) => {
      if (!term || term.length < 2) return null
      const lower = removeAccents(term.toLowerCase())
      
      // Filtrage des stopwords (seulement à l'indexation)
      if (stopwords[locale].has(lower)) return null
      
      // Application du stemmer
      return stemmers[locale].stem(lower)
    },
    
    // Options de recherche par défaut
    searchOptions: {
      prefix: true,
      fuzzy: 0.2,
      boost: { title: 2, excerpt: 1.5, content: 1 },
      weights: { fuzzy: 0.45, prefix: 0.375 }
    }
  }
}
```

### Options `processTerm` et `tokenize` par langue

| Option | Fonction | Retour |
|--------|----------|--------|
| `tokenize(text, fieldName?)` | Découpe le texte en tokens | `string[]` |
| `processTerm(term, fieldName?)` | Normalise/filtre chaque token | `string \| null` (null = supprimé) |

```typescript
// Exemple: processTerm différent pour indexation vs recherche
const miniSearch = new MiniSearch({
  fields: ['content'],
  
  // À l'indexation: filtrer stopwords
  processTerm: (term) => {
    const lower = term.toLowerCase()
    if (stopwords.fr.has(lower)) return null
    return stemmers.fr.stem(lower)
  },
  
  searchOptions: {
    // À la recherche: garder les stopwords (meilleur pour phrases exactes)
    processTerm: (term) => term?.toLowerCase()
  }
})
```

### Boosting et scoring adapté

```typescript
// Recherche avec préférence linguistique
function searchWithBoost(
  index: MiniSearch<SearchDocument>,
  query: string,
  options?: { limit?: number }
) {
  return index.search(query, {
    prefix: true,
    fuzzy: 0.2,
    boost: { title: 2, excerpt: 1.5 },
    // Boost par terme (nouveau en v7.1.0)
    boostTerm: (term) => {
      // Boost les termes importants
      if (term.length > 8) return 1.2
      return 1.0
    }
  }).slice(0, options?.limit ?? 20)
}
```

---

## Intégration Nuxt Content 3.10+ avec MiniSearch

### API `queryCollectionSearchSections()` 

Cette API extrait automatiquement les sections recherchables depuis vos collections SQLite:

```typescript
// Signature
queryCollectionSearchSections(
  collection: keyof Collections,
  options?: {
    ignoredTags?: string[]  // Tags à ignorer (ex: 'code', 'pre')
    minHeading?: string     // Niveau min (default: 'h1')
    maxHeading?: string     // Niveau max (default: 'h6')
  }
): ChainablePromise<Section[]>

// Structure Section retournée
interface Section {
  id: string       // "/docs/guide#installation" (utilisable comme URL)
  title: string    // Texte du heading
  titles: string[] // Breadcrumb des parents ["Guide", "Getting Started"]
  content: string  // Contenu textuel de la section
  level: number    // Niveau du heading (1-6)
}
```

### Configuration des collections multilingues

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { z } from 'zod'

const blogSchema = z.object({
  title: z.string(),
  description: z.string(),
  date: z.date(),
  author: z.string(),
  tags: z.array(z.string()),
  searchKeywords: z.array(z.string()).optional(), // Mots-clés additionnels
})

export default defineContentConfig({
  collections: {
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/blog/**/*.md', prefix: '' },
      schema: blogSchema,
    }),
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'en/blog/**/*.md', prefix: '' },
      schema: blogSchema,
    }),
  },
})
```

### Structure des fichiers content

```
content/
├── fr/
│   └── blog/
│       ├── introduction-nuxt.md
│       └── guide-typescript.md
└── en/
    └── blog/
        ├── nuxt-introduction.md
        └── typescript-guide.md
```

---

## Architecture d'index par locale

### Pattern recommandé: index séparés par langue

Pour les applications avec **3+ langues** ou un volume important de contenu, utilisez des **index séparés par locale**. Cette approche optimise la mémoire client et permet des analyseurs linguistiques spécifiques.

| Critère | Index séparés | Index unifié |
|---------|---------------|--------------|
| **Mémoire client** | Faible (1 locale) | Élevée (toutes) |
| **Changement locale** | Rechargement (~100-500ms) | Instantané |
| **Analyseurs linguistiques** | ✅ Possible | ⚠️ Complexe |
| **Recherche cross-langue** | ❌ Impossible | ✅ Possible |
| **Recommandé pour** | >3 langues, >1000 docs | <3 langues, <500 docs |

### Composable `useSearch` avec cache d'index

```typescript
// app/composables/useSearch.ts
import MiniSearch from 'minisearch'
import type { Collections } from '@nuxt/content'
import { createMiniSearchOptions, type SearchDocument } from '~/lib/search/minisearch-config'

export function useSearch() {
  const { locale } = useI18n()
  const localePath = useLocalePath()
  
  const index = shallowRef<MiniSearch<SearchDocument> | null>(null)
  const isLoading = ref(false)
  const isReady = ref(false)
  const error = ref<Error | null>(null)
  
  // Cache pour éviter les rechargements
  const indexCache = new Map<string, MiniSearch<SearchDocument>>()
  
  const loadIndex = async (localeCode: string): Promise<void> => {
    // Retour depuis cache si disponible
    if (indexCache.has(localeCode)) {
      index.value = indexCache.get(localeCode)!
      isReady.value = true
      return
    }
    
    isLoading.value = true
    error.value = null
    
    try {
      // Fetch de l'index pré-construit (SSG)
      const response = await fetch(`/search/index-${localeCode}.json`)
      if (!response.ok) throw new Error(`Index not found for ${localeCode}`)
      
      const json = await response.text()
      
      // Désérialisation avec les mêmes options
      const miniSearch = MiniSearch.loadJSON<SearchDocument>(
        json,
        createMiniSearchOptions(localeCode as 'fr' | 'en')
      )
      
      indexCache.set(localeCode, miniSearch)
      index.value = miniSearch
      isReady.value = true
    } catch (err) {
      error.value = err instanceof Error ? err : new Error('Failed to load index')
    } finally {
      isLoading.value = false
    }
  }
  
  // Rechargement automatique lors du changement de locale
  watch(
    () => locale.value,
    (newLocale) => loadIndex(newLocale),
    { immediate: true }
  )
  
  const search = (query: string, options?: { limit?: number }) => {
    if (!index.value || !query.trim()) return []
    
    return index.value
      .search(query, { prefix: true, fuzzy: 0.2, boost: { title: 2 } })
      .slice(0, options?.limit ?? 15)
      .map(result => ({
        ...result,
        path: localePath(result.path) // Chemin localisé
      }))
  }
  
  return {
    search,
    isLoading: readonly(isLoading),
    isReady: readonly(isReady),
    error: readonly(error),
    currentLocale: readonly(locale)
  }
}
```

### Gestion du changement de locale

```vue
<script setup lang="ts">
const { search, isLoading, isReady } = useSearch()
const query = ref('')

// Debouncing avec VueUse
const debouncedQuery = refDebounced(query, 300)

const results = computed(() => {
  if (!isReady.value || !debouncedQuery.value) return []
  return search(debouncedQuery.value, { limit: 20 })
})
</script>

<template>
  <div>
    <input 
      v-model="query" 
      :disabled="isLoading"
      :placeholder="isLoading ? 'Chargement...' : 'Rechercher...'"
    />
    <div v-if="isLoading" class="animate-pulse">Chargement de l'index...</div>
    <ul v-else-if="results.length">
      <li v-for="result in results" :key="result.id">
        <NuxtLink :to="result.path">{{ result.title }}</NuxtLink>
      </li>
    </ul>
  </div>
</template>
```

---

## SSG et pré-génération des index

### Génération au build time avec hook Nitro

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/content', '@nuxtjs/i18n'],
  
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      crawlLinks: true,
      routes: ['/']
    }
  },
  
  hooks: {
    async 'nitro:build:public-assets'(nitro) {
      const { generateSearchIndexes } = await import('./scripts/build-search')
      await generateSearchIndexes(nitro.options.output.publicDir)
    }
  }
})
```

### Script de génération d'index

```typescript
// scripts/build-search.ts
import MiniSearch from 'minisearch'
import { writeFile, mkdir } from 'fs/promises'
import { join } from 'path'
import { createMiniSearchOptions, type SearchDocument } from '../lib/search/minisearch-config'

interface ContentSection {
  id: string
  title: string
  titles: string[]
  content: string
  level: number
}

export async function generateSearchIndexes(publicDir: string) {
  const locales = ['fr', 'en'] as const
  
  await mkdir(join(publicDir, 'search'), { recursive: true })
  
  for (const locale of locales) {
    // Fetch sections via API prerender (ou directement depuis DB)
    const sections: ContentSection[] = await $fetch(`/api/search-sections/${locale}`)
    
    // Transformation en SearchDocument
    const documents: SearchDocument[] = sections.map((section, idx) => ({
      id: section.id,
      title: section.title,
      content: section.content,
      path: section.id.split('#')[0], // Chemin sans anchor
      excerpt: section.content.slice(0, 200),
      locale
    }))
    
    // Création de l'index
    const miniSearch = new MiniSearch<SearchDocument>(createMiniSearchOptions(locale))
    miniSearch.addAll(documents)
    
    // Sérialisation JSON
    const indexPath = join(publicDir, 'search', `index-${locale}.json`)
    await writeFile(indexPath, JSON.stringify(miniSearch), 'utf-8')
    
    console.log(`✅ Index ${locale}: ${documents.length} documents`)
  }
}
```

### API route pour les sections de recherche

```typescript
// server/api/search-sections/[locale].get.ts
import type { Collections } from '@nuxt/content'

export default defineEventHandler(async (event) => {
  const locale = getRouterParam(event, 'locale') || 'en'
  const collectionName = `blog_${locale}` as keyof Collections
  
  const sections = await queryCollectionSearchSections(event, collectionName, {
    ignoredTags: ['code', 'pre', 'script'],
    minHeading: 'h2',
    maxHeading: 'h4',
  })
  
  return sections
})
```

### Configuration Cloudflare Pages

```typescript
// nuxt.config.ts - Configuration SSG complète
export default defineNuxtConfig({
  ssr: true,
  
  nitro: {
    preset: 'cloudflare_pages',
    output: {
      publicDir: '.output/public'
    },
    prerender: {
      crawlLinks: true,
      routes: [
        '/',
        '/api/search-sections/fr',
        '/api/search-sections/en'
      ]
    }
  },
  
  // Route rules pour les fichiers statiques
  routeRules: {
    '/search/**': { 
      headers: { 
        'Cache-Control': 'public, max-age=86400',
        'Content-Type': 'application/json'
      }
    }
  }
})
```

**Limites Cloudflare Pages à respecter:**
- **Max file size**: 25 MiB (splitter les gros index si nécessaire)
- **Max files**: 20,000 (plan gratuit)
- **Build timeout**: 20 minutes

---

## UX recherche multilingue

### Composable complet avec debouncing et erreurs

```typescript
// app/composables/useSearchWithUX.ts
import { useDebounceFn } from '@vueuse/core'
import type { SearchResult } from 'minisearch'

interface UseSearchOptions {
  debounceMs?: number
  minQueryLength?: number
  maxResults?: number
}

export function useSearchWithUX(options: UseSearchOptions = {}) {
  const { debounceMs = 300, minQueryLength = 2, maxResults = 20 } = options
  const { search: performSearch, isLoading: indexLoading, isReady, error: indexError } = useSearch()
  
  const query = ref('')
  const results = ref<SearchResult[]>([])
  const isSearching = ref(false)
  const error = ref<string | null>(null)
  
  // Status pour annonces ARIA
  const statusMessage = computed(() => {
    if (indexLoading.value) return 'Chargement de l\'index de recherche...'
    if (isSearching.value) return 'Recherche en cours...'
    if (error.value) return `Erreur: ${error.value}`
    if (query.value.length < minQueryLength) return ''
    if (results.value.length === 0) return `Aucun résultat pour "${query.value}"`
    return `${results.value.length} résultat${results.value.length > 1 ? 's' : ''} trouvé${results.value.length > 1 ? 's' : ''}`
  })
  
  const executeSearch = useDebounceFn(async (q: string) => {
    if (q.length < minQueryLength) {
      results.value = []
      return
    }
    
    isSearching.value = true
    error.value = null
    
    try {
      results.value = performSearch(q, { limit: maxResults })
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Erreur de recherche'
      results.value = []
    } finally {
      isSearching.value = false
    }
  }, debounceMs, { maxWait: 1000 })
  
  watch(query, (newQuery) => executeSearch(newQuery))
  
  const clear = () => {
    query.value = ''
    results.value = []
    error.value = null
  }
  
  return {
    query,
    results: readonly(results),
    isLoading: computed(() => indexLoading.value || isSearching.value),
    isReady,
    error: computed(() => indexError.value?.message || error.value),
    statusMessage,
    isEmpty: computed(() => results.value.length === 0 && query.value.length >= minQueryLength && !isSearching.value),
    hasResults: computed(() => results.value.length > 0),
    clear
  }
}
```

### Highlighting des résultats

```typescript
// utils/highlight.ts
export interface HighlightChunk {
  text: string
  highlighted: boolean
}

export function highlightText(text: string, query: string): HighlightChunk[] {
  if (!query || !text) return [{ text, highlighted: false }]
  
  const escapedQuery = query.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
  const regex = new RegExp(`(${escapedQuery})`, 'gi')
  const parts = text.split(regex)
  
  return parts.filter(Boolean).map(part => ({
    text: part,
    highlighted: part.toLowerCase() === query.toLowerCase()
  }))
}
```

```vue
<!-- components/SearchHighlight.vue -->
<script setup lang="ts">
import { highlightText, type HighlightChunk } from '~/utils/highlight'

const props = defineProps<{
  text: string
  query: string
}>()

const chunks = computed<HighlightChunk[]>(() => 
  highlightText(props.text, props.query)
)
</script>

<template>
  <span>
    <template v-for="(chunk, index) in chunks" :key="index">
      <mark v-if="chunk.highlighted" class="bg-yellow-200 text-inherit rounded px-0.5">
        {{ chunk.text }}
      </mark>
      <span v-else>{{ chunk.text }}</span>
    </template>
  </span>
</template>
```

### Composant de recherche accessible avec shadcn-vue

```vue
<!-- components/SearchModal.vue -->
<script setup lang="ts">
import { Search, X, Loader2 } from 'lucide-vue-next'
import {
  CommandDialog,
  CommandEmpty,
  CommandGroup,
  CommandInput,
  CommandItem,
  CommandList,
} from '@/components/ui/command'
import { DialogTitle, DialogDescription } from '@/components/ui/dialog'

const open = defineModel<boolean>('open', { default: false })
const { query, results, isLoading, isEmpty, statusMessage, clear } = useSearchWithUX()
const localePath = useLocalePath()

// Raccourci clavier Cmd/Ctrl + K
const { meta_k, ctrl_k } = useMagicKeys()
whenever(logicOr(meta_k, ctrl_k), () => {
  open.value = true
})

// Navigation clavier
const activeIndex = ref(-1)

function handleSelect(result: any) {
  navigateTo(localePath(result.path))
  open.value = false
  clear()
}

watch(open, (isOpen) => {
  if (!isOpen) {
    clear()
    activeIndex.value = -1
  }
})
</script>

<template>
  <CommandDialog v-model:open="open">
    <!-- Accessibilité: titres pour screen readers -->
    <DialogTitle class="sr-only">Recherche</DialogTitle>
    <DialogDescription class="sr-only">
      Recherchez dans la documentation. Utilisez les flèches pour naviguer.
    </DialogDescription>
    
    <CommandInput 
      v-model="query"
      placeholder="Rechercher..."
      :class="{ 'animate-pulse': isLoading }"
    >
      <template #prefix>
        <Loader2 v-if="isLoading" class="w-4 h-4 animate-spin" />
        <Search v-else class="w-4 h-4" />
      </template>
    </CommandInput>
    
    <!-- Status pour ARIA live region -->
    <div role="status" aria-live="polite" class="sr-only">
      {{ statusMessage }}
    </div>
    
    <CommandList>
      <CommandEmpty v-if="isEmpty">
        Aucun résultat pour "{{ query }}"
      </CommandEmpty>
      
      <CommandEmpty v-else-if="isLoading">
        Recherche en cours...
      </CommandEmpty>
      
      <CommandGroup v-if="results.length" heading="Résultats">
        <CommandItem
          v-for="(result, index) in results"
          :key="result.id"
          :value="result.id"
          :aria-selected="activeIndex === index"
          @select="handleSelect(result)"
        >
          <div class="flex flex-col gap-1">
            <SearchHighlight 
              :text="result.title" 
              :query="query" 
              class="font-medium"
            />
            <span v-if="result.excerpt" class="text-sm text-muted-foreground line-clamp-1">
              {{ result.excerpt }}
            </span>
          </div>
        </CommandItem>
      </CommandGroup>
    </CommandList>
  </CommandDialog>
</template>
```

### Attributs ARIA essentiels

| Attribut | Élément | Valeur | Objectif |
|----------|---------|--------|----------|
| `role="combobox"` | Input | — | Identifie le widget |
| `aria-expanded` | Input | `true/false` | Popup visible? |
| `aria-controls` | Input | ID de la liste | Référence les résultats |
| `aria-activedescendant` | Input | ID de l'option | Option focusée |
| `aria-autocomplete` | Input | `"list"` | Type d'autocomplete |
| `role="listbox"` | Liste résultats | — | Conteneur de résultats |
| `role="option"` | Chaque résultat | — | Item sélectionnable |
| `aria-selected` | Option | `true/false` | État de sélection |
| `aria-live="polite"` | Status | — | Annonces dynamiques |

---

## Anti-patterns à éviter

### ❌ Ne pas faire

```typescript
// ❌ Index unique avec tous les documents de toutes les langues
const globalIndex = new MiniSearch({ fields: ['title', 'content'] })
globalIndex.addAll([...frenchDocs, ...englishDocs]) // Problème: pas de stemming par langue

// ❌ Stopwords filtrés à la recherche (empêche les phrases exactes)
searchOptions: {
  processTerm: (term) => stopwords.has(term) ? null : term
}

// ❌ Chargement de l'index au SSR (bloque le rendu)
const { data } = await useAsyncData('search', () => 
  queryCollectionSearchSections('docs')
) // Augmente le payload SSR

// ❌ ref() pour stocker un gros index (overhead réactivité)
const index = ref(new MiniSearch({ ... })) // Utiliser shallowRef()

// ❌ Pas de debouncing sur l'input de recherche
watch(query, (q) => index.search(q)) // Trop de requêtes

// ❌ Index > 25 MiB sur Cloudflare Pages
// Fichier rejeté au déploiement
```

### ✅ Bonnes pratiques

```typescript
// ✅ Index séparé par langue avec stemmer approprié
const frIndex = new MiniSearch(createMiniSearchOptions('fr'))
const enIndex = new MiniSearch(createMiniSearchOptions('en'))

// ✅ Stopwords filtrés seulement à l'indexation
processTerm: (term) => stopwords.has(term) ? null : stemmer.stem(term)
searchOptions: { processTerm: (term) => term?.toLowerCase() }

// ✅ Chargement client-only
useLazyAsyncData('search', () => loadIndex(), { server: false })

// ✅ shallowRef pour les gros objets
const index = shallowRef<MiniSearch | null>(null)

// ✅ Debouncing 300ms avec maxWait
const debouncedSearch = useDebounceFn(search, 300, { maxWait: 1000 })

// ✅ Split des index si trop volumineux
// index-fr-blog.json, index-fr-docs.json, etc.
```

---

## Checklist d'implémentation pour Claude Code

### Phase 1: Configuration de base

- [ ] Installer les dépendances: `pnpm add minisearch snowball-stemmers stopword`
- [ ] Créer `lib/search/minisearch-config.ts` avec les options par langue
- [ ] Configurer `content.config.ts` avec collections `blog_fr`, `blog_en`
- [ ] Créer la structure `content/fr/` et `content/en/`

### Phase 2: Génération d'index SSG

- [ ] Créer `server/api/search-sections/[locale].get.ts`
- [ ] Créer `scripts/build-search.ts` pour génération au build
- [ ] Ajouter hook `nitro:build:public-assets` dans `nuxt.config.ts`
- [ ] Configurer `nitro.preset: 'cloudflare_pages'`
- [ ] Vérifier que les index générés sont < 25 MiB

### Phase 3: Composables client

- [ ] Créer `app/composables/useSearch.ts` avec cache d'index
- [ ] Créer `app/composables/useSearchWithUX.ts` avec debouncing
- [ ] Créer `utils/highlight.ts` pour le highlighting
- [ ] Tester le rechargement d'index au changement de locale

### Phase 4: Interface utilisateur

- [ ] Créer `components/SearchHighlight.vue`
- [ ] Créer `components/SearchModal.vue` avec shadcn-vue Command
- [ ] Implémenter raccourci clavier ⌘K / Ctrl+K
- [ ] Ajouter tous les attributs ARIA requis
- [ ] Tester navigation clavier (↑↓ Enter Escape)

### Phase 5: Tests et optimisation

- [ ] Vérifier le bundle size de MiniSearch (~7KB gzipped)
- [ ] Tester le temps de chargement d'index par locale (< 500ms)
- [ ] Vérifier les headers Cache-Control sur `/search/*.json`
- [ ] Tester sur mobile (index < 200KB recommandé)
- [ ] Valider accessibilité avec screen reader

### Phase 6: Déploiement Cloudflare

- [ ] Build command: `nuxt generate`
- [ ] Output directory: `.output/public`
- [ ] Désactiver Rocket Loader™ dans Cloudflare
- [ ] Désactiver Email Obfuscation
- [ ] Vérifier compression Brotli/Gzip des index JSON

---

## Nouveautés MiniSearch 7.x

| Version | Changement | Impact |
|---------|------------|--------|
| **7.0.0** | Target ES6+ (plus de IE11) | Vérifier compatibilité navigateurs |
| **7.0.0** | `SearchableMap` import séparé | `import { SearchableMap } from 'minisearch/SearchableMap'` |
| **7.1.0** | Option `boostTerm` | Boost personnalisé par terme de recherche |
| **7.2.0** | Option `stringifyField` | Contrôle de la sérialisation des champs |
| **7.2.0** | Target ES2018 | Unicode regex dans tokenizer |

**Compatibilité index**: Les index sérialisés avec MiniSearch 6.x sont compatibles avec 7.x — pas besoin de reconstruire lors de la migration.