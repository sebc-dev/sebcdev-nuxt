# Anti-patterns

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

## Exemples détaillés

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
