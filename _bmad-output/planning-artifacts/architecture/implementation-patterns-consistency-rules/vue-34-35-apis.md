# Vue 3.4+ / 3.5+ APIs

**defineModel() — v-model simplifié (Vue 3.4+):**

```typescript
// Remplace props + emit pour v-model
const modelValue = defineModel<string>()
const count = defineModel<number>('count', { required: true })

// Utilisation dans le template
<input v-model="modelValue" />
```

**Générics pour composants réutilisables (Vue 3.3+):**

L'attribut `generic` permet de créer des composants type-safe qui s'adaptent à leur input :

```vue
<!-- components/BaseList.vue -->
<script setup lang="ts" generic="T extends { id: string | number }">
interface Props {
  items: T[]
  selected?: T
}

const props = defineProps<Props>()
const emit = defineEmits<{ select: [item: T] }>()

const isSelected = (item: T) => item.id === props.selected?.id

function handleSelect(item: T) {
  emit('select', item)
}
</script>

<template>
  <ul>
    <li
      v-for="item in items"
      :key="item.id"
      :class="{ 'bg-primary': isSelected(item) }"
      @click="handleSelect(item)"
    >
      <slot :item="item" />
    </li>
  </ul>
</template>
```

**Utilisation avec inférence automatique :**

```vue
<script setup lang="ts">
interface User {
  id: number
  name: string
  email: string
}

const users: User[] = [...]
const selectedUser = ref<User | undefined>()

// TypeScript infère que T = User
function handleUserSelect(user: User) {
  selectedUser.value = user
}
</script>

<template>
  <!-- T est inféré comme User, select émet User -->
  <BaseList
    :items="users"
    :selected="selectedUser"
    @select="handleUserSelect"
  >
    <template #default="{ item }">
      {{ item.name }} - {{ item.email }}  <!-- item est typé User -->
    </template>
  </BaseList>
</template>
```

**Contraintes multiples :**

```vue
<script setup lang="ts" generic="T extends BaseItem, K extends keyof T">
defineProps<{
  items: T[]
  labelKey: K
  valueKey: K
}>()
</script>
```

---

**Operateur `satisfies` (TypeScript 4.9+):**

`satisfies` valide qu'une valeur correspond à un type **sans élargir l'inférence**. Utile pour les configurations et objets littéraux.

```typescript
// ❌ Type assertion : perd l'inférence des clés littérales
const routes = {
  home: '/',
  about: '/about',
  blog: '/blog'
} as Record<string, string>

routes.home  // string (perd '/')
routes.typo  // ✅ Compile (pas d'erreur sur clé inexistante)

// ✅ satisfies : valide le type ET préserve l'inférence
const routes = {
  home: '/',
  about: '/about',
  blog: '/blog'
} satisfies Record<string, string>

routes.home  // '/' (littéral préservé)
routes.typo  // ❌ Erreur TypeScript : Property 'typo' does not exist
```

**Cas d'usage recommandés :**

| Cas | Exemple |
|-----|---------|
| **Configurations** | `nuxt.config.ts`, options de modules |
| **Objets de mapping** | Routes, traductions, constantes |
| **Directives Vue** | `vBindings satisfies DirectiveBinding` |
| **Props avec defaults** | Valider les valeurs par défaut |

```typescript
// Configuration de composant avec satisfies
const defaultProps = {
  variant: 'primary',
  size: 'md',
  disabled: false
} satisfies Partial<ButtonProps>

// Mapping de routes avec validation
const apiRoutes = {
  users: '/api/users',
  posts: '/api/posts',
  comments: '/api/comments'
} satisfies Record<string, `/${string}`>  // Valide que toutes les valeurs commencent par /
```

**Computed avec accès à l'ancienne valeur (Vue 3.4+):**

```typescript
const alwaysSmall = computed((previous) => {
  if (count.value <= 3) {
    return count.value
  }
  return previous  // Retourne la dernière valeur valide
})
```

**Reactive Props Destructure (Vue 3.5+ stable):**

```typescript
// Destructuration réactive directe des props
const { count = 0, title } = defineProps<{
  count?: number
  title: string
}>()

// ⚠️ Pour watch, wrapper en getter
watch(() => count, (newVal) => { ... })
```

**pause/resume pour watchers (Vue 3.5+):**

```typescript
const { pause, resume, stop } = watch(source, callback)

pause()   // Pause temporaire
resume()  // Reprendre
stop()    // Arrêt définitif
```

**onWatcherCleanup pour AbortController (Vue 3.5+):**

```typescript
import { watch, onWatcherCleanup } from 'vue'

watch(id, async (newId) => {
  const controller = new AbortController()

  onWatcherCleanup(() => controller.abort())

  const response = await fetch(`/api/${newId}`, {
    signal: controller.signal
  })
})
```

**effectScope pour gestion groupée des effets:**

```typescript
import { effectScope, watchEffect, onScopeDispose } from 'vue'

export function useComplexFeature() {
  const scope = effectScope()

  return scope.run(() => {
    const data = shallowRef([])

    watchEffect(() => {
      // Effet tracké automatiquement dans le scope
    })

    onScopeDispose(() => {
      console.log('Cleanup automatique de tous les effets')
    })

    return { data, stop: () => scope.stop() }
  })
}

// Utilisation - cleanup automatique
const { data, stop } = useComplexFeature()
onUnmounted(() => stop())
```

---

## Optimisation INP (Interaction to Next Paint)

L'INP mesure la latence de toutes les interactions utilisateur. L'objectif est de maintenir chaque interaction sous **200ms**. Vue 3 offre plusieurs APIs pour optimiser la réactivité.

### shallowRef pour grandes structures de données

Pour les structures > 1000 items, `shallowRef` évite l'overhead de la réactivité profonde :

```typescript
import { shallowRef, triggerRef } from 'vue'

// ✅ shallowRef pour listes volumineuses
const optimizedData = shallowRef({
  items: Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    details: { price: i * 10, active: true }
  }))
})

// Mise à jour : remplacer la valeur entière
function updateItem(index: number, newPrice: number) {
  optimizedData.value = {
    ...optimizedData.value,
    items: optimizedData.value.items.map((item, i) =>
      i === index ? { ...item, details: { ...item.details, price: newPrice } } : item
    )
  }
}

// Alternative : mutation directe + triggerRef
function updateItemDirect(index: number, newPrice: number) {
  optimizedData.value.items[index].details.price = newPrice
  triggerRef(optimizedData)  // Force la mise à jour du rendu
}
```

| API | Réactivité | Cas d'usage |
|-----|-----------|-------------|
| `ref()` | Profonde (récursive) | Structures < 100 items |
| `shallowRef()` | Surface uniquement | Listes > 1000 items, arbres complexes |
| `triggerRef()` | Force update | Après mutation directe d'un shallowRef |

### markRaw pour instances de librairies externes

Les instances de librairies tierces (Chart.js, editors, etc.) ne doivent **jamais** être réactives :

```typescript
import { ref, markRaw } from 'vue'
import Chart from 'chart.js/auto'

const chartInstance = ref<Chart | null>(null)

onMounted(() => {
  // ✅ markRaw empêche Vue de wrapper l'instance
  chartInstance.value = markRaw(new Chart(canvas, config))
})

onUnmounted(() => {
  chartInstance.value?.destroy()
})
```

**⚠️ Sans markRaw :** Vue tente de rendre l'instance réactive, causant des erreurs et une dégradation des performances.

### v-memo pour listes volumineuses (micro-optimisation)

`v-memo` est une **micro-optimisation** pour les listes de plus de **1000 items** avec composants enfants coûteux :

```vue
<template>
  <!-- v-memo : seuls les items dont le statut change sont re-rendus -->
  <div
    v-for="item in items"
    :key="item.id"
    v-memo="[item.id === selectedId]"
  >
    <p>ID: {{ item.id }} - selected: {{ item.id === selectedId }}</p>
    <ExpensiveChildComponent :data="item" />
  </div>
</template>
```

| Quand utiliser v-memo | Quand ne PAS utiliser |
|----------------------|----------------------|
| Listes > 1000 items | Listes < 100 items |
| Composants enfants lourds | Composants simples |
| Changements fréquents sur peu d'items | Changements globaux fréquents |

### v-once pour contenus statiques

`v-once` rend le contenu **une seule fois**, jamais mis à jour ensuite :

```vue
<template>
  <!-- ✅ Header statique - jamais re-rendu -->
  <header v-once>
    <h1>{{ siteTitle }}</h1>
    <p>{{ staticDescription }}</p>
  </header>

  <!-- ✅ Footer avec données de build -->
  <footer v-once>
    <p>Version {{ buildVersion }} - {{ buildDate }}</p>
  </footer>
</template>
```

### Event Delegation vs Fonctions Inline

Évitez les fonctions inline dans les templates (nouvelle fonction à chaque render) :

```vue
<template>
  <!-- ❌ Anti-pattern : nouvelle fonction créée à chaque render -->
  <button @click="() => handleClick(item.id)">Click</button>

  <!-- ✅ Event delegation avec data attributes -->
  <ul @click="handleItemClick">
    <li v-for="item in items" :key="item.id" :data-id="item.id">
      {{ item.name }}
    </li>
  </ul>

  <!-- ✅ Passive listeners pour scroll/touch -->
  <div @scroll.passive="onScroll" @touchmove.passive="onTouchMove">
    <!-- Contenu scrollable -->
  </div>
</template>

<script setup lang="ts">
function handleItemClick(event: Event) {
  const target = event.target as HTMLElement
  const listItem = target.closest('li')
  if (listItem) {
    const id = listItem.dataset.id
    // Traitement avec l'ID
  }
}
</script>
```

| Modifier | Effet | Cas d'usage |
|----------|-------|-------------|
| `.passive` | `addEventListener({ passive: true })` | scroll, touchmove, wheel |
| `.once` | Handler exécuté une seule fois | Initialisations |
| `.capture` | Phase de capture | Event delegation avancé |
