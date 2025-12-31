# Alternative : Pagefind (>500 pages)

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

## Installation et configuration

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

## Composant Vue Pagefind

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

## Multilingue automatique

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

## Avantages Pagefind pour SSG

| Aspect | Comportement |
|--------|-------------|
| **Build time** | Index généré automatiquement post-build |
| **Network** | Charge uniquement les chunks nécessaires (~100KB max) |
| **Highlighting** | Excerpts avec `<mark>` inclus par défaut |
| **i18n** | Index séparés par langue automatique |
| **Maintenance** | Aucune configuration à maintenir |

**Note** : Pagefind offre moins de contrôle sur le boosting et le stemming multilingue que MiniSearch, mais son intégration "zero-config" le rend idéal pour les blogs à forte croissance.

---
