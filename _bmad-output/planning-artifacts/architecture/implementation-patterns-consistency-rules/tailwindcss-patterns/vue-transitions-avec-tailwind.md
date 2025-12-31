# Vue Transitions avec Tailwind

## Composant `<Transition>` avec classes Tailwind

Le composant `<Transition>` de Vue 3 s'intègre parfaitement avec les classes Tailwind via les props de classes personnalisées. Cette approche est idéale pour le SSG car les transitions CSS fonctionnent immédiatement sans hydratation :

```vue
<script setup lang="ts">
const isOpen = ref(false)
</script>

<template>
  <Transition
    enter-active-class="transition-all duration-300 ease-out"
    enter-from-class="opacity-0 scale-95 translate-y-4"
    enter-to-class="opacity-100 scale-100 translate-y-0"
    leave-active-class="transition-all duration-200 ease-in"
    leave-from-class="opacity-100 scale-100 translate-y-0"
    leave-to-class="opacity-0 scale-95 translate-y-4"
  >
    <div v-if="isOpen" class="modal-content">
      Contenu animé
    </div>
  </Transition>
</template>
```

## TransitionGroup pour listes animées

Pour `<TransitionGroup>`, ajouter `move-class` pour animer le repositionnement des éléments :

```vue
<TransitionGroup
  tag="ul"
  enter-active-class="transition-all duration-300"
  enter-from-class="opacity-0 translate-x-8"
  leave-active-class="transition-all duration-200"
  leave-to-class="opacity-0 translate-x-8"
  move-class="transition-all duration-300"
>
  <li v-for="item in items" :key="item.id">
    {{ item.name }}
  </li>
</TransitionGroup>
```

## Composable réutilisable useTransitions

```typescript
// composables/useTransitions.ts
export function useScaleFadeTransition(duration = 200) {
  return {
    enterActiveClass: `transition-all duration-${duration} ease-out`,
    enterFromClass: 'opacity-0 scale-95',
    enterToClass: 'opacity-100 scale-100',
    leaveActiveClass: `transition-all duration-${duration} ease-in`,
    leaveFromClass: 'opacity-100 scale-100',
    leaveToClass: 'opacity-0 scale-95'
  }
}

export function useSlideTransition(direction: 'left' | 'right' | 'up' | 'down' = 'right', duration = 300) {
  const translations = {
    left: 'translate-x-8',
    right: '-translate-x-8',
    up: 'translate-y-8',
    down: '-translate-y-8'
  }
  return {
    enterActiveClass: `transition-all duration-${duration} ease-out`,
    enterFromClass: `opacity-0 ${translations[direction]}`,
    enterToClass: 'opacity-100 translate-x-0 translate-y-0',
    leaveActiveClass: `transition-all duration-${duration} ease-in`,
    leaveFromClass: 'opacity-100 translate-x-0 translate-y-0',
    leaveToClass: `opacity-0 ${translations[direction]}`
  }
}
```

**Usage :**

```vue
<script setup>
import { useScaleFadeTransition } from '~/composables/useTransitions'
const transitionProps = useScaleFadeTransition(300)
</script>

<template>
  <Transition v-bind="transitionProps">
    <div v-if="show">Contenu</div>
  </Transition>
</template>
```

---
