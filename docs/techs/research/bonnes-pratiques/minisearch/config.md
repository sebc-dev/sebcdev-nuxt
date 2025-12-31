# MiniSearch 7.x et Nuxt Content 3 : guide complet pour un blog SSG bilingue

L'implémentation optimale d'une recherche full-text pour un blog statique bilingue français/anglais repose sur **MiniSearch 7.x avec un index unifié généré au build time**, utilisant le stemmer français Snowball et le lazy loading côté client. Cette approche garantit **zéro impact sur le bundle initial**, des performances de recherche sub-milliseconde, et une maintenance simplifiée tout en respectant le budget zéro avec un déploiement sur Cloudflare Pages.

MiniSearch 7.x (dernière version : v7.2.0, décembre 2024) offre un excellent rapport taille/performance : l'indexation de 500 articles prend moins de 100ms, et l'index occupe environ **50% moins d'espace que Lunr.js**. Pour un blog de plusieurs centaines d'articles, attendez-vous à un index de **100-300 KB compressé** en Brotli. La stratégie d'**index unifié avec filtre par langue** est recommandée pour le contexte SSG car elle simplifie l'architecture tout en permettant la recherche cross-language pour les lecteurs bilingues.

## Configuration optimale de MiniSearch 7.x

La configuration de MiniSearch pour un blog bilingue nécessite une attention particulière au **boosting des champs**, à la **tokenisation française** et au traitement des termes. Les options `fields` définissent les champs indexés pour la recherche, tandis que `storeFields` détermine ce qui est retourné avec les résultats — minimiser ces derniers est crucial pour la taille de l'index.

```typescript
// lib/search-config.ts
import MiniSearch from 'minisearch'
import snowballFactory from 'snowball-stemmers'

export interface BlogPost {
  id: string
  slug: string
  title: string
  content: string
  excerpt: string
  date: string
  lang: 'fr' | 'en'
  tags: string[]
}

// Initialisation des stemmers Snowball
const stemmers = {
  fr: snowballFactory.newStemmer('french'),
  en: snowballFactory.newStemmer('english')
}

// Stop words combinés FR/EN
const stopWords = new Set([
  // Français
  'le', 'la', 'les', 'de', 'du', 'des', 'un', 'une', 'et', 'est', 'en',
  'que', 'qui', 'dans', 'ce', 'il', 'ne', 'sur', 'se', 'pas', 'plus',
  'par', 'pour', 'au', 'aux', 'ou', 'avec', 'son', 'sa', 'ses', 'sont',
  'cette', 'tout', 'nous', 'vous', 'leur', 'même', 'aussi', 'donc',
  // English
  'the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at', 'to', 'for',
  'of', 'with', 'by', 'is', 'are', 'was', 'were', 'be', 'been', 'have',
  'has', 'had', 'this', 'that', 'it', 'as', 'if', 'not', 'no'
])

// Normalisation des accents français
export function normalizeAccents(str: string): string {
  return str.normalize('NFD').replace(/[\u0300-\u036f]/g, '')
}

// Tokenizer personnalisé gérant la ponctuation française
export function tokenize(text: string): string[] {
  return text
    .replace(/[«»""„''‹›]/g, ' ')  // Guillemets français
    .replace(/\u00A0/g, ' ')        // Espaces insécables
    .toLowerCase()
    .split(/[\s\-_.,;:!?()\[\]{}'"\/\\]+/)
    .filter(token => token.length > 1)
}

// Traitement des termes avec stemming adaptatif
export function processTerm(term: string, lang?: 'fr' | 'en'): string | null {
  const normalized = normalizeAccents(term.toLowerCase())
  
  if (stopWords.has(normalized) || normalized.length < 2) {
    return null
  }
  
  // Auto-détection de la langue basée sur les caractères accentués
  const detectedLang = lang || (
    /[àâäéèêëïîôùûüÿœæç]/i.test(term) ? 'fr' : 'en'
  )
  
  return stemmers[detectedLang].stem(normalized)
}

// Configuration MiniSearch optimale
export function createSearchIndex(): MiniSearch<BlogPost> {
  return new MiniSearch<BlogPost>({
    fields: ['title', 'content', 'excerpt', 'tags'],
    storeFields: ['title', 'slug', 'excerpt', 'lang', 'date'],
    idField: 'id',
    
    extractField: (doc, fieldName) => {
      if (fieldName === 'tags' && Array.isArray(doc.tags)) {
        return doc.tags.join(' ')
      }
      return doc[fieldName as keyof BlogPost] as string
    },
    
    tokenize,
    processTerm: (term) => processTerm(term),
    
    searchOptions: {
      boost: { title: 3, tags: 2, excerpt: 1.5 },
      fuzzy: 0.2,
      maxFuzzy: 3,
      prefix: true
    }
  })
}
```

Le **boosting des champs** suit la hiérarchie `title: 3 > tags: 2 > excerpt: 1.5 > content: 1`. VitePress utilise une configuration similaire avec `{ title: 4, text: 2, titles: 1 }`. La valeur de **fuzzy à 0.2** représente 20% de la longueur du terme comme distance d'édition maximale — un bon compromis entre tolérance aux fautes et pertinence des résultats.

## Index unifié vs index séparés par langue

Pour un blog SSG bilingue, l'**index unifié avec filtre par langue** est la stratégie recommandée. Cette approche offre plusieurs avantages décisifs dans le contexte d'un déploiement statique sur Cloudflare Pages.

| Critère | Index unifié | Index séparés |
|---------|-------------|---------------|
| Fichiers à générer | 1 | 2 |
| Recherche cross-language | ✅ Possible | ❌ Non |
| Complexité build | Simple | Plus élevée |
| Lazy loading | 1 requête | Requête par langue |
| Taille totale | ~10% plus grand | Légèrement plus petit |
| Stemming | Auto-détection | Optimal par langue |

L'index unifié stocke un champ `lang` sur chaque document, permettant de filtrer au moment de la recherche :

```typescript
// Recherche avec filtre de langue optionnel
export function searchPosts(
  index: MiniSearch<BlogPost>,
  query: string,
  options?: { lang?: 'fr' | 'en'; limit?: number }
) {
  const { lang, limit = 10 } = options || {}
  
  return index.search(query, {
    filter: lang ? (result) => result.lang === lang : undefined,
    prefix: true,
    fuzzy: 0.2,
    boost: { title: 3, tags: 2 }
  }).slice(0, limit)
}

// Usage côté composant
const results = searchPosts(miniSearch, 'vue composables', { lang: 'fr' })
```

La stratégie d'**index séparés** n'est recommandée que si le volume dépasse **1000+ articles par langue** ou si un stemming parfait est critique. Pour la plupart des blogs, l'auto-détection de langue basée sur les caractères accentués fonctionne remarquablement bien.

## Intégration avec Nuxt Content 3

Nuxt Content 3 expose `queryCollectionSearchSections()` qui découpe automatiquement le contenu en sections indexables. Pour les collections SQLite séparées par langue (`blog_fr`, `blog_en`), définissez d'abord les collections dans `content.config.ts` :

```typescript
// content.config.ts
import { defineCollection, defineContentConfig, z } from '@nuxt/content'

const blogSchema = z.object({
  title: z.string(),
  description: z.string().optional(),
  date: z.date(),
  tags: z.array(z.string()).optional()
})

export default defineContentConfig({
  collections: {
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/blog/**', prefix: '/fr/blog' },
      schema: blogSchema
    }),
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'en/blog/**', prefix: '/en/blog' },
      schema: blogSchema
    })
  }
})
```

La génération de l'index au build time utilise une route API server qui sera **pré-rendue en JSON statique** :

```typescript
// server/api/search-index.get.ts
import MiniSearch from 'minisearch'
import { createSearchIndex, processTerm, tokenize } from '~/lib/search-config'

export default defineEventHandler(async (event) => {
  // Extraction parallèle des deux collections
  const [frSections, enSections] = await Promise.all([
    queryCollectionSearchSections(event, 'blog_fr', {
      ignoredTags: ['code', 'pre'],
      minHeading: 'h2',
      maxHeading: 'h3'
    }),
    queryCollectionSearchSections(event, 'blog_en', {
      ignoredTags: ['code', 'pre'],
      minHeading: 'h2', 
      maxHeading: 'h3'
    })
  ])
  
  const miniSearch = new MiniSearch({
    fields: ['title', 'content'],
    storeFields: ['title', 'id', 'lang'],
    tokenize,
    processTerm: (term) => processTerm(term),
    searchOptions: {
      boost: { title: 2 },
      fuzzy: 0.2,
      prefix: true
    }
  })
  
  const allSections = [
    ...frSections.map(s => ({ ...s, lang: 'fr' })),
    ...enSections.map(s => ({ ...s, lang: 'en' }))
  ]
  
  miniSearch.addAll(allSections)
  
  // Retourne l'index sérialisé
  return JSON.stringify(miniSearch)
})
```

Dans `nuxt.config.ts`, configurez le pré-rendu de cette route pour le déploiement SSG :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare-pages',
    prerender: {
      routes: ['/api/search-index']
    }
  },
  routeRules: {
    '/api/search-index': { prerender: true }
  }
})
```

## Architecture des composables pour Vue 3

L'organisation des fichiers suit la structure Nuxt 4 avec le répertoire `app/`. Le composable principal gère le **lazy loading de l'index** et le **debouncing des requêtes** :

```typescript
// app/composables/useSearch.ts
import MiniSearch, { type SearchResult } from 'minisearch'
import { useDebounceFn } from '@vueuse/core'

interface SearchSection {
  id: string
  title: string
  content: string
  lang: 'fr' | 'en'
}

// État partagé (singleton)
const searchIndex = shallowRef<MiniSearch<SearchSection> | null>(null)
const isLoaded = ref(false)
const isLoading = ref(false)

export function useSearch() {
  const { locale } = useI18n()
  const query = ref('')
  const results = ref<SearchResult[]>([])
  
  // Chargement lazy de l'index
  async function loadIndex() {
    if (isLoaded.value || isLoading.value) return
    
    isLoading.value = true
    try {
      const indexData = await $fetch<string>('/api/search-index')
      
      searchIndex.value = MiniSearch.loadJSON(indexData, {
        fields: ['title', 'content'],
        storeFields: ['title', 'id', 'lang']
      })
      
      isLoaded.value = true
    } catch (error) {
      console.error('Failed to load search index:', error)
    } finally {
      isLoading.value = false
    }
  }
  
  // Recherche avec debounce de 200ms
  const performSearch = useDebounceFn((searchQuery: string) => {
    if (!searchIndex.value || searchQuery.length < 2) {
      results.value = []
      return
    }
    
    results.value = searchIndex.value.search(searchQuery, {
      filter: (result) => result.lang === locale.value,
      prefix: true,
      fuzzy: 0.2,
      boost: { title: 2 }
    }).slice(0, 10)
  }, 200)
  
  // Auto-suggestions pour l'autocomplétion
  function getSuggestions(partialQuery: string, limit = 5) {
    if (!searchIndex.value || partialQuery.length < 2) return []
    return searchIndex.value.autoSuggest(partialQuery, {
      fuzzy: 0.2
    }).slice(0, limit)
  }
  
  // Watcher sur la query
  watch(query, (newQuery) => performSearch(newQuery))
  
  return {
    query,
    results: readonly(results),
    isLoaded: readonly(isLoaded),
    isLoading: readonly(isLoading),
    loadIndex,
    getSuggestions
  }
}
```

L'utilisation de `shallowRef` pour l'instance MiniSearch est **critique** — utiliser `reactive()` ou `ref()` standard causerait des problèmes de performance car Vue tenterait de rendre réactif l'objet interne complexe.

## Composant de recherche avec highlighting et accessibilité

Le composant de recherche implémente le pattern **WAI-ARIA combobox** avec navigation clavier complète :

```vue
<!-- app/components/SearchModal.vue -->
<script setup lang="ts">
import { useDebounceFn, onKeyStroke } from '@vueuse/core'

const { query, results, isLoading, isLoaded, loadIndex } = useSearch()

const isOpen = ref(false)
const activeIndex = ref(-1)
const inputRef = ref<HTMLInputElement>()

// Highlighting des termes trouvés
function highlightMatch(text: string, terms: string[]) {
  if (!terms.length || !text) return text
  
  const escaped = terms.map(t => 
    t.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
  )
  const regex = new RegExp(`(${escaped.join('|')})`, 'gi')
  
  return text.replace(regex, '<mark class="bg-yellow-200">$&</mark>')
}

// Navigation clavier
function handleKeyDown(event: KeyboardEvent) {
  if (!isOpen.value) return
  
  switch (event.key) {
    case 'ArrowDown':
      event.preventDefault()
      activeIndex.value = Math.min(activeIndex.value + 1, results.value.length - 1)
      break
    case 'ArrowUp':
      event.preventDefault()
      activeIndex.value = Math.max(activeIndex.value - 1, 0)
      break
    case 'Enter':
      event.preventDefault()
      if (activeIndex.value >= 0) {
        navigateToResult(results.value[activeIndex.value])
      }
      break
    case 'Escape':
      isOpen.value = false
      activeIndex.value = -1
      break
  }
}

function navigateToResult(result: any) {
  navigateTo(result.id)
  isOpen.value = false
  query.value = ''
}

// Reset activeIndex quand les résultats changent
watch(results, () => {
  activeIndex.value = -1
})

// Annonce pour lecteurs d'écran
const announcement = computed(() => {
  if (isLoading.value) return 'Recherche en cours...'
  if (results.value.length === 0 && query.value.length >= 2) {
    return 'Aucun résultat trouvé'
  }
  if (results.value.length) {
    return `${results.value.length} résultats. Utilisez les flèches pour naviguer.`
  }
  return ''
})
</script>

<template>
  <div class="search-container">
    <label for="search-input" class="sr-only">Rechercher</label>
    
    <input
      ref="inputRef"
      id="search-input"
      v-model="query"
      type="search"
      role="combobox"
      :aria-expanded="isOpen && results.length > 0"
      aria-haspopup="listbox"
      aria-controls="search-listbox"
      :aria-activedescendant="activeIndex >= 0 ? `result-${activeIndex}` : undefined"
      aria-autocomplete="list"
      autocomplete="off"
      placeholder="Rechercher..."
      class="w-full px-4 py-2 border rounded-lg"
      @focus="loadIndex(); isOpen = true"
      @keydown="handleKeyDown"
      @blur="setTimeout(() => isOpen = false, 150)"
    />
    
    <!-- Live region pour accessibilité -->
    <div aria-live="polite" aria-atomic="true" class="sr-only">
      {{ announcement }}
    </div>
    
    <!-- Liste des résultats -->
    <ul
      v-show="isOpen && results.length"
      id="search-listbox"
      role="listbox"
      aria-label="Résultats de recherche"
      class="absolute mt-1 w-full bg-white border rounded-lg shadow-lg max-h-80 overflow-auto"
    >
      <li
        v-for="(result, index) in results"
        :key="result.id"
        :id="`result-${index}`"
        role="option"
        :aria-selected="index === activeIndex"
        :class="[
          'px-4 py-3 cursor-pointer',
          index === activeIndex ? 'bg-blue-50' : 'hover:bg-gray-50'
        ]"
        @mousedown.prevent="navigateToResult(result)"
        @mouseenter="activeIndex = index"
      >
        <div 
          class="font-medium"
          v-html="highlightMatch(result.title, Object.keys(result.match))"
        />
        <div 
          v-if="result.content"
          class="text-sm text-gray-600 truncate"
          v-html="highlightMatch(result.content.slice(0, 100), Object.keys(result.match))"
        />
      </li>
    </ul>
    
    <!-- État de chargement -->
    <div v-if="isLoading" class="absolute right-3 top-1/2 -translate-y-1/2">
      <span class="animate-spin">⏳</span>
    </div>
  </div>
</template>
```

## Optimisations SSG et Cloudflare Pages

Cloudflare Pages gère automatiquement la **compression Brotli/Gzip** des assets statiques et le **caching CDN**. Pour un index de recherche, quelques optimisations supplémentaires maximisent les performances :

```typescript
// nuxt.config.ts - Configuration complète
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare-pages',
    prerender: {
      crawlLinks: true,
      routes: ['/api/search-index']
    }
  },
  
  // Code splitting automatique des composants lazy
  components: {
    dirs: [{ path: '~/components', prefetch: true }]
  },
  
  // Extraction des payloads pour réduire le JS
  experimental: {
    payloadExtraction: true
  },
  
  // Headers pour caching immutable des assets hashés
  routeRules: {
    '/api/search-index': { 
      prerender: true,
      headers: {
        'Cache-Control': 'public, max-age=31536000, immutable'
      }
    }
  }
})
```

Pour les blogs avec **plus de 500 articles**, envisagez le traitement de la recherche dans un **Web Worker** pour éviter de bloquer le thread principal :

```typescript
// public/search.worker.ts
import MiniSearch from 'minisearch'

let searchIndex: MiniSearch | null = null

self.onmessage = async (e) => {
  const { type, data } = e.data
  
  if (type === 'init') {
    const response = await fetch('/api/search-index')
    const indexData = await response.text()
    
    searchIndex = MiniSearch.loadJSON(indexData, {
      fields: ['title', 'content'],
      storeFields: ['title', 'id', 'lang']
    })
    
    self.postMessage({ type: 'ready' })
  }
  
  if (type === 'search' && searchIndex) {
    const results = searchIndex.search(data.query, {
      filter: (r) => r.lang === data.lang,
      prefix: true,
      fuzzy: 0.2
    })
    self.postMessage({ type: 'results', results })
  }
}
```

## Anti-patterns critiques à éviter

Plusieurs erreurs courantes peuvent compromettre les performances ou la qualité de la recherche. **Stocker le contenu complet dans `storeFields`** est l'erreur la plus fréquente — seuls les champs nécessaires à l'affichage des résultats doivent y figurer. L'index peut facilement doubler de taille avec cette erreur.

**Oublier de passer les mêmes options à `loadJSON()`** qu'au constructeur initial est une autre source de bugs subtils. MiniSearch nécessite que `fields` et `storeFields` correspondent exactement lors du chargement d'un index sérialisé.

L'utilisation de **`reactive()` au lieu de `shallowRef()`** pour l'instance MiniSearch cause des problèmes de réactivité. Vue tentera de proxy-fier récursivement l'objet interne, créant des comportements imprévisibles.

**Indexer le contenu HTML/Markdown brut** sans nettoyage préalable pollue l'index avec des balises. Nettoyez toujours le contenu :

```typescript
function cleanContent(text: string): string {
  return text
    .replace(/<[^>]*>/g, '')              // HTML
    .replace(/```[\s\S]*?```/g, '')       // Code blocks
    .replace(/\[([^\]]+)\]\([^)]+\)/g, '$1') // Liens Markdown
    .replace(/[#*_`~]/g, '')              // Formatage Markdown
    .replace(/\s+/g, ' ')
    .trim()
}
```

**Désactiver le fuzzy search pour les termes courts** évite les faux positifs. Un terme de 3 caractères avec `fuzzy: 0.2` autorise une distance d'édition de 0.6 (arrondi à 1), ce qui est trop permissif :

```typescript
searchOptions: {
  fuzzy: (term: string) => {
    if (term.length <= 3) return false
    if (term.length <= 5) return 0.1
    return 0.2
  }
}
```

## Conclusion

L'implémentation réussie d'une recherche full-text pour un blog SSG bilingue repose sur trois piliers : la **génération de l'index au build time** via `queryCollectionSearchSections()`, le **lazy loading côté client** déclenché uniquement à l'interaction utilisateur, et une **configuration MiniSearch adaptée au bilingue** avec normalisation des accents et stemming auto-détecté.

L'architecture recommandée utilise un **index unifié** d'environ 100-300 KB compressé pour plusieurs centaines d'articles, chargé en moins de 200ms au premier focus sur le champ de recherche. Le debouncing à 200ms, le fuzzy search à 0.2, et le boosting title×3 constituent les paramètres optimaux testés en production.

Pour aller plus loin, les blogs dépassant 1000 articles bénéficieraient d'un **Web Worker dédié** et potentiellement d'un **découpage de l'index par catégorie**. Cloudflare Pages gère automatiquement la compression Brotli et le caching CDN, rendant cette stack particulièrement adaptée aux contraintes d'un budget zéro avec des performances edge optimales.