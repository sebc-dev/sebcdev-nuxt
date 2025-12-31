# Guide Vue 3 Composition API pour Nuxt 4.2.X

La Composition API avec `<script setup lang="ts">` représente le standard obligatoire pour tout projet Nuxt 4 moderne. Ce guide détaille les conventions établies en décembre 2025, optimisées pour la génération de code avec Claude Code, le déploiement Cloudflare Pages et l'intégration Nuxt Content 3.

---

## 1. `<script setup lang="ts">` : le standard obligatoire

La syntaxe `<script setup lang="ts">` combine trois avantages décisifs : **code 30% plus concis** que l'Options API, **meilleure performance runtime** grâce à la compilation directe dans le même scope, et **inférence TypeScript supérieure** réduisant le travail du language server.

### Pourquoi c'est le standard Nuxt 4

Vue 3 et Nuxt 4 recommandent exclusivement cette approche. Le futur **Vapor Mode** (Vue 3.6+) ne supportera que `<script setup>` avec Composition API, promettant des performances comparables à Solid.js. Le bundle size diminue significativement grâce au tree-shaking optimal des imports nommés.

### Structure de fichier recommandée

L'ordre des éléments dans `<script setup>` suit une convention stricte que Claude Code doit respecter :

```vue
<script setup lang="ts">
// 1. IMPORTS EXTERNES
import { ref, computed, watch, onMounted } from 'vue'

// 2. IMPORTS COMPOSANTS
import BaseButton from '@/components/base/BaseButton.vue'

// 3. IMPORTS COMPOSABLES
import { useBlogPost } from '@/composables/useBlogPost'

// 4. IMPORTS TYPES
import type { BlogPost, ApiResponse } from '@/types'

// 5. TYPES LOCAUX
interface Props {
  postId: number
  mode?: 'view' | 'edit'
}

// 6. PROPS & EMITS
const props = withDefaults(defineProps<Props>(), {
  mode: 'view'
})

const emit = defineEmits<{
  save: [data: BlogPost]
  cancel: []
}>()

// 7. COMPOSABLES
const { post, isLoading, refresh } = useBlogPost(() => props.postId)
const route = useRoute()

// 8. STATE RÉACTIF
const isEditing = ref(false)
const formData = ref<BlogPost | null>(null)

// 9. COMPUTED
const canSave = computed(() => !isLoading.value && formData.value !== null)
const isEditMode = computed(() => props.mode === 'edit')

// 10. WATCHERS
watch(() => props.postId, () => refresh(), { immediate: true })

// 11. MÉTHODES
function handleSave(): void {
  if (!canSave.value || !formData.value) return
  emit('save', formData.value)
}

// 12. LIFECYCLE HOOKS
onMounted(() => {
  console.log('Component mounted')
})

// 13. EXPOSE (optionnel)
defineExpose({ refresh })
</script>

<template>
  <div class="blog-post">
    <slot name="header" />
    <BaseButton @click="handleSave" :disabled="!canSave">
      Sauvegarder
    </BaseButton>
  </div>
</template>

<style scoped>
/* Styles scopés */
</style>
```

### Patterns TypeScript essentiels Vue 3.4+

**Props typées avec valeurs par défaut** — utiliser `interface` (extensible) plutôt que `type` :

```typescript
interface Props {
  user: UserModel
  items?: ItemModel[]
  config?: Partial<ConfigOptions>
}

const props = withDefaults(defineProps<Props>(), {
  items: () => [],  // Fonction pour éviter mutations partagées
  config: () => ({ theme: 'light' })
})
```

**Emits typés avec syntaxe tuple** (Vue 3.3+) :

```typescript
const emit = defineEmits<{
  change: [id: number]
  update: [value: string, timestamp: Date]
  'item-selected': [item: ItemModel]
}>()
```

**defineModel pour v-model simplifié** (Vue 3.4+) :

```typescript
const modelValue = defineModel<string>()
const count = defineModel<number>('count', { required: true })
```

**Generics pour composants réutilisables** :

```vue
<script setup lang="ts" generic="T extends BaseItem">
import type { BaseItem } from '@/types'

defineProps<{
  items: T[]
  labelKey: keyof T
}>()
</script>
```

### Optimisation pour Claude Code

Pour garantir une génération cohérente, inclure ces règles dans le contexte :

| Type | Convention | Exemple |
|------|------------|---------|
| Composants | PascalCase | `UserProfile.vue` |
| Composables | camelCase + `use` | `useBlogPost.ts` |
| Refs boolean | `is`, `has`, `can` | `isLoading`, `hasError` |
| Handlers | `handle` + Event | `handleSubmit` |
| Async functions | verbe action | `fetchData`, `saveUser` |

---

## 2. Composables : conventions et patterns avancés

Les composables constituent le cœur de la réutilisation logique en Vue 3. La convention **préfixe `use`** est obligatoire et documentée officiellement.

### Convention de nommage officielle

```typescript
// ✅ Convention recommandée - camelCase avec préfixe "use"
export function useMouse() { ... }
export function useBlogPosts() { ... }

// ❌ À éviter
export function mouseTracker() { ... }  // Pas de préfixe "use"
export function UseMouse() { ... }      // PascalCase incorrect
```

Dans Nuxt 4, les fichiers `composables/useFoo.ts` sont auto-importés. L'export nommé `useFoo` ou l'export par défaut sont tous deux supportés.

### Pattern de retour : objets avec refs destructurables

La convention officielle Vue.js exige de **retourner un objet plain contenant des refs**, jamais un objet réactif :

```typescript
// ✅ RECOMMANDÉ - Objet plain avec refs
export function useCounter(initialValue = 0) {
  const count = shallowRef(initialValue)
  
  const increment = () => count.value++
  const decrement = () => count.value--
  
  return { count: readonly(count), increment, decrement }
}

// Utilisation - conserve la réactivité
const { count, increment } = useCounter()
```

**Pourquoi éviter reactive() pour le retour** : la destructuration perd la connexion réactive :

```typescript
// ❌ PROBLÈME - Perd la réactivité à la destructuration
export function useMouse() {
  const state = reactive({ x: 0, y: 0 })
  return state
}
const { x, y } = useMouse() // x et y ne sont plus réactifs!
```

### MaybeRefOrGetter pour APIs flexibles

Depuis Vue 3.3, les types natifs `MaybeRef<T>` et `MaybeRefOrGetter<T>` remplacent les versions VueUse (dépréciées). La fonction `toValue()` normalise tous les formats d'entrée :

```typescript
import { 
  toValue, 
  watchEffect, 
  type MaybeRefOrGetter 
} from 'vue'

export function useFetch<T>(url: MaybeRefOrGetter<string>) {
  const data = shallowRef<T | null>(null)
  const error = shallowRef<Error | null>(null)
  const isLoading = shallowRef(false)

  watchEffect(async () => {
    isLoading.value = true
    error.value = null
    
    try {
      const response = await fetch(toValue(url))  // Tracking automatique
      data.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e : new Error(String(e))
    } finally {
      isLoading.value = false
    }
  })

  return { 
    data: readonly(data), 
    error: readonly(error), 
    isLoading: readonly(isLoading) 
  }
}
```

**Toutes ces utilisations deviennent valides** :

```typescript
useFetch('/api/posts')                    // Valeur brute
useFetch(ref('/api/posts'))               // Ref
useFetch(() => `/api/posts/${props.id}`)  // Getter réactif aux props
```

### Pattern typé complet avec interface de retour

```typescript
import { shallowRef, readonly, type Ref, type MaybeRefOrGetter, toValue } from 'vue'

export interface UseAsyncStateReturn<T> {
  data: Readonly<Ref<T | null>>
  error: Readonly<Ref<Error | null>>
  isLoading: Readonly<Ref<boolean>>
  execute: () => Promise<void>
}

export interface UseAsyncStateOptions {
  immediate?: boolean
}

export function useAsyncState<T>(
  asyncFn: () => Promise<T>,
  options: UseAsyncStateOptions = {}
): UseAsyncStateReturn<T> {
  const { immediate = true } = options
  
  const data = shallowRef<T | null>(null)
  const error = shallowRef<Error | null>(null)
  const isLoading = shallowRef(false)
  
  const execute = async () => {
    isLoading.value = true
    error.value = null
    
    try {
      data.value = await asyncFn()
    } catch (e) {
      error.value = e instanceof Error ? e : new Error(String(e))
    } finally {
      isLoading.value = false
    }
  }
  
  if (immediate) execute()
  
  return {
    data: readonly(data),
    error: readonly(error),
    isLoading: readonly(isLoading),
    execute
  }
}
```

### Composables spécifiques Nuxt Content 3

**Composable pour articles de blog** :

```typescript
// composables/useBlogPosts.ts
export interface BlogPost {
  _path: string
  title: string
  description: string
  date: string
  tags: string[]
}

export function useBlogPosts() {
  const posts = shallowRef<BlogPost[]>([])
  const isLoading = shallowRef(false)

  const fetchPosts = async (options: { limit?: number; tag?: string } = {}) => {
    isLoading.value = true
    
    try {
      let query = queryCollection('blog')
        .select('_path', 'title', 'description', 'date', 'tags')
        .order('date', 'DESC')
      
      if (options.tag) {
        query = query.where('tags', 'LIKE', `%${options.tag}%`)
      }
      
      if (options.limit) {
        query = query.limit(options.limit)
      }
      
      posts.value = await query.all()
    } finally {
      isLoading.value = false
    }
  }

  return {
    posts: readonly(posts),
    isLoading: readonly(isLoading),
    fetchPosts
  }
}
```

**Composable avec navigation article précédent/suivant** :

```typescript
// composables/useBlogPost.ts
export function useBlogPost(path: MaybeRefOrGetter<string>) {
  const { data: post, pending } = useAsyncData(
    `post-${toValue(path)}`,
    () => queryCollection('blog').path(toValue(path)).first()
  )
  
  const { data: surroundings } = useAsyncData(
    `surroundings-${toValue(path)}`,
    () => queryCollectionItemSurroundings('blog', toValue(path), {
      fields: ['_path', 'title']
    })
  )
  
  const prev = computed(() => surroundings.value?.[0] ?? null)
  const next = computed(() => surroundings.value?.[1] ?? null)
  
  return { post, isLoading: pending, prev, next }
}
```

---

## 3. Réactivité avancée : performance avec grandes listes

La gestion de **10k+ items** nécessite des stratégies spécifiques pour éviter les goulots d'étranglement de performance.

### Quand utiliser ref, shallowRef, reactive

| Critère | `ref()` | `shallowRef()` | `reactive()` |
|---------|---------|----------------|--------------|
| Types supportés | Tous | Tous | Objets uniquement |
| Réactivité profonde | ✅ Oui | ❌ Non | ✅ Oui |
| Réassignation | ✅ Oui | ✅ Oui | ❌ Non |
| **Performance 10k+ items** | 🔴 Lente | 🟢 Rapide | 🔴 Lente |

**Benchmarks Vue 3.5** : `ref()` sur 100k items nécessite ~1000ms d'initialisation contre ~2ms pour `shallowRef()`. Le nouveau système de réactivité apporte une **réduction mémoire de 56%**.

### Pattern shallowRef pour grandes listes

```typescript
import { shallowRef, triggerRef, computed } from 'vue'

// 1. Utiliser shallowRef pour le stockage massif
const items = shallowRef<Item[]>([])  // 10k+ items

// 2. Mise à jour par remplacement immutable
function addItem(newItem: Item): void {
  items.value = [...items.value, newItem]
}

function updateItem(index: number, updates: Partial<Item>): void {
  items.value = [
    ...items.value.slice(0, index),
    { ...items.value[index], ...updates },
    ...items.value.slice(index + 1)
  ]
}

// 3. Alternative : mutation + trigger manuel
function quickMutate(index: number, value: string): void {
  items.value[index].name = value
  triggerRef(items)  // Force la mise à jour
}

// 4. Computed pour filtrage/transformation
const filteredItems = computed(() => 
  items.value.filter(item => item.active)
)
```

### Optimisations computed Vue 3.4+

Depuis Vue 3.4, les computed utilisent un **cache intelligent** : ils ne déclenchent des effets que si leur valeur change réellement.

```typescript
const count = ref(0)
const isEven = computed(() => count.value % 2 === 0)

watchEffect(() => console.log(isEven.value))  // "true"

count.value = 2  // ❌ Pas de log (reste true)
count.value = 4  // ❌ Pas de log (reste true)  
count.value = 1  // ✅ Log "false" (valeur change)
```

**Accès à l'ancienne valeur** (Vue 3.4+) :

```typescript
const alwaysSmall = computed((previous) => {
  if (count.value <= 3) {
    return count.value
  }
  return previous  // Retourne la dernière valeur valide
})
```

**Décomposer les computed massifs** pour une invalidation granulaire :

```typescript
// ❌ Computed massif - toute modification invalide tout
const massiveComputed = computed(() => 
  items.value.filter(...).map(...).sort(...).reduce(...)
)

// ✅ Computed décomposés - invalidation ciblée
const filteredItems = computed(() => items.value.filter(...))
const mappedItems = computed(() => filteredItems.value.map(...))
const sortedItems = computed(() => mappedItems.value.sort(...))
```

### effectScope pour gestion groupée des effets

```typescript
import { effectScope, watchEffect, onScopeDispose, onUnmounted } from 'vue'

export function useComplexFeature() {
  const scope = effectScope()
  
  return scope.run(() => {
    const data = shallowRef([])
    
    watchEffect(() => {
      // Effet tracké
    })
    
    onScopeDispose(() => {
      console.log('Cleanup automatique')
    })
    
    return { data, scope }
  })
}

// Dans le composant
const { data, scope } = useComplexFeature()
onUnmounted(() => scope.stop())  // Arrête tous les effets
```

### Nouvelles APIs Vue 3.5+

**Reactive Props Destructure** (stable) :

```typescript
// Vue 3.5+ - Destructuration réactive directe
const { count = 0 } = defineProps<{ count?: number }>()

// ⚠️ Pour watch, wrapper en getter
watch(() => count, (newVal) => { ... })
```

**pause/resume pour watchers** :

```typescript
const { pause, resume, stop } = watch(source, callback)
pause()   // Pause temporaire
resume()  // Reprendre
stop()    // Arrêt définitif
```

**onWatcherCleanup pour AbortController** :

```typescript
import { watch, onWatcherCleanup } from 'vue'

watch(id, (newId) => {
  const controller = new AbortController()
  fetch(`/api/${newId}`, { signal: controller.signal })
  
  onWatcherCleanup(() => controller.abort())
})
```

### Virtualisation pour 10k+ items

Pour les listes massives, la virtualisation est indispensable :

```vue
<script setup lang="ts">
import { RecycleScroller } from 'vue-virtual-scroller'
</script>

<template>
  <RecycleScroller 
    :items="sortedItems" 
    :item-size="60" 
    key-field="id"
  >
    <template #default="{ item }">
      <div v-memo="[item.id, item.updated]">
        {{ item.name }}
      </div>
    </template>
  </RecycleScroller>
</template>
```

**Alternatives recommandées** : `@tanstack/vue-virtual`, Nuxt UI `<UScrollArea>` avec `virtualize`.

---

## 4. Auto-imports Nuxt 4 : configuration optimale

Nuxt 4 auto-importe automatiquement les utilities Vue, les composables et les composants, éliminant le boilerplate d'imports.

### Vue utilities auto-importés par défaut

**Reactivity APIs** : `ref`, `reactive`, `computed`, `readonly`, `shallowRef`, `shallowReactive`, `toRef`, `toRefs`, `toRaw`, `unref`, `isRef`, `isReactive`, `isReadonly`, `isProxy`

**Lifecycle Hooks** : `onMounted`, `onUpdated`, `onUnmounted`, `onBeforeMount`, `onBeforeUpdate`, `onBeforeUnmount`, `onErrorCaptured`, `onActivated`, `onDeactivated`, `onServerPrefetch`

**Watchers** : `watch`, `watchEffect`, `watchPostEffect`, `watchSyncEffect`

**Nuxt Composables** : `useFetch`, `useAsyncData`, `useState`, `useCookie`, `useRoute`, `useRouter`, `useHead`, `useSeoMeta`, `useRuntimeConfig`, `navigateTo`

### Configuration nuxt.config.ts recommandée

```typescript
export default defineNuxtConfig({
  compatibilityDate: '2024-11-01',
  
  // Auto-imports
  imports: {
    scan: true,
    dirs: [
      'composables',
      'composables/**',  // Scan récursif
      'utils',
      'stores'
    ]
  },

  // Composants
  components: {
    dirs: [
      { path: '~/components/global', global: true },
      { path: '~/components/content', global: true },  // Pour MDC
      '~/components'
    ]
  },

  // TypeScript
  typescript: {
    strict: true,
    typeCheck: 'build'
  },

  // Nitro pour Cloudflare Pages
  nitro: {
    preset: 'cloudflare-pages',
    prerender: {
      crawlLinks: true,
      routes: ['/']
    }
  },

  // Modules
  modules: ['@nuxt/content'],

  // Route rules
  routeRules: {
    '/blog/**': { isr: 3600 }  // ISR 1h pour le blog
  }
})
```

### Structure de dossiers Nuxt 4

```
project-root/
├── app/                        # srcDir par défaut Nuxt 4
│   ├── components/
│   │   ├── global/             # Composants globaux
│   │   ├── content/            # Composants MDC
│   │   └── ui/
│   ├── composables/            # Auto-importés
│   │   ├── index.ts            # Re-exports recommandés
│   │   ├── useBlogPosts.ts
│   │   └── useAuth.ts
│   ├── utils/                  # Auto-importés
│   ├── pages/
│   ├── layouts/
│   └── app.vue
├── content/                    # Fichiers Markdown
│   └── blog/
├── server/
│   └── utils/                  # Auto-importés côté serveur
├── shared/                     # Partagé app + server
│   └── utils/
├── content.config.ts
├── nuxt.config.ts
└── tsconfig.json
```

### Configuration TypeScript Nuxt 4

**tsconfig.json** avec Project References :

```json
{
  "files": [],
  "references": [
    { "path": "./.nuxt/tsconfig.app.json" },
    { "path": "./.nuxt/tsconfig.server.json" },
    { "path": "./.nuxt/tsconfig.shared.json" },
    { "path": "./.nuxt/tsconfig.node.json" }
  ]
}
```

Les types sont générés dans `.nuxt/imports.d.ts` et `.nuxt/types/components.d.ts` automatiquement via `npx nuxt prepare`.

### Import explicite avec #imports

Pour clarté ou résolution de conflits :

```typescript
import { ref, computed } from '#imports'
```

### Bonnes pratiques pour éviter les conflits

```typescript
// ❌ Noms génériques risqués
export const data = ref([])
export const error = ref(null)

// ✅ Nommage explicite
export const useBlogData = () => { ... }
export const useApiError = () => { ... }
```

**Règle critique** : les composables Nuxt doivent être appelés DANS la fonction composable, jamais au niveau module :

```typescript
// ✅ CORRECT
export const useMyFeature = () => {
  const config = useRuntimeConfig()  // Dans la fonction
  return { config }
}

// ❌ ERREUR - "Nuxt instance unavailable"
const config = useRuntimeConfig()  // Hors fonction
export const useMyFeature = () => { ... }
```

### Configuration Nuxt Content 3

**content.config.ts** :

```typescript
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { z } from 'zod'

export default defineContentConfig({
  collections: {
    blog: defineCollection({
      type: 'page',
      source: 'blog/**/*.md',
      schema: z.object({
        title: z.string(),
        description: z.string().optional(),
        date: z.date(),
        tags: z.array(z.string()).optional()
      })
    })
  }
})
```

**Composables auto-importés** par Nuxt Content 3 : `queryCollection()`, `queryCollectionNavigation()`, `queryCollectionItemSurroundings()`, `queryCollectionSearchSections()`

---

## Résumé des bonnes pratiques pour Claude Code

### À TOUJOURS FAIRE

- Utiliser `<script setup lang="ts">` exclusivement
- Suivre l'ordre : imports → types → props → composables → refs → computed → watch → functions → lifecycle
- Préfixer les composables avec `use` et retourner des objets avec refs
- Utiliser `shallowRef()` pour les listes 100+ items
- Décomposer les computed massifs en computed granulaires
- Typer les props avec `interface`, les emits avec syntax tuple
- Utiliser `MaybeRefOrGetter<T>` + `toValue()` pour APIs flexibles

### À ÉVITER

- Mélanger Options API et Composition API
- Utiliser `reactive()` pour le retour des composables
- Créer des refs profondes pour grandes collections de données
- Ignorer le typage des emits et des template refs
- Appeler des composables Nuxt hors du contexte setup

Ce guide constitue la référence pour garantir une génération de code Vue 3 / Nuxt 4.2.X cohérente, performante et maintenable avec Claude Code.