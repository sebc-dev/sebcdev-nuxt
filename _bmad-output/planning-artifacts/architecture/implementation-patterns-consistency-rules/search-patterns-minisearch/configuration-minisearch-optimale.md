# Configuration MiniSearch Optimale

## Normalisation des accents (crucial FR/EN)

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

## Stemmer Snowball (optionnel)

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

## Configuration complète

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

## Mécanismes de boosting avancés (MiniSearch 7.1.0+)

| Mécanisme | Fonction | Cas d'usage |
|-----------|----------|-------------|
| **Field boost** | `boost: { title: 3 }` | Prioriser certains champs |
| **Document boost** | `boostDocument(id, term, storedFields)` | Favoriser contenu récent, populaire |
| **Term boost** | `boostTerm(term, index, terms)` | Premier terme plus important |

**Note** : `boostDocument` et `boostTerm` requièrent MiniSearch ≥7.1.0. Pour utiliser `boostDocument`, les champs nécessaires (ex: `publishedAt`) doivent être dans `storeFields`.

## processTerm différencié : indexation vs recherche

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
