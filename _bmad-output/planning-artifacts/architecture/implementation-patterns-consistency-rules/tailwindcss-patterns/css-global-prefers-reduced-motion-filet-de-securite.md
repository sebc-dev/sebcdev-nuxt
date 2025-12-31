# CSS Global prefers-reduced-motion (Filet de Sécurité)

En complément des variants `motion-safe:` et `motion-reduce:` de Tailwind, ce CSS global agit comme **filet de sécurité** pour les animations non gérées individuellement.

## Configuration dans main.css

```css
/* app/assets/css/main.css */
@import "tailwindcss";

@layer base {
  /* Filet de sécurité global pour reduced motion */
  @media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
      scroll-behavior: auto !important;
    }
  }
}
```

## Pourquoi ce pattern est nécessaire

| Sans filet de sécurité | Avec filet de sécurité |
|------------------------|------------------------|
| Chaque animation doit être gérée individuellement | Fallback automatique pour animations oubliées |
| CSS tiers peut ignorer les préférences | Tout le CSS est couvert |
| Risque d'accessibilité si un dev oublie | Accessibilité garantie par défaut |

## Combinaison avec variants Tailwind

```html
<!-- Approach recommandée : opt-in explicite -->
<button class="
  bg-primary
  motion-safe:transition
  motion-safe:duration-200
  motion-safe:hover:scale-105
">
  Bouton animé si autorisé
</button>

<!-- Le filet de sécurité capture les oublis -->
<div class="animate-spin">
  <!-- Si motion-reduce: pas demandé explicitement,
       le filet de sécurité l'arrête quand même -->
</div>
```

## Composable VueUse pour animations JavaScript

```typescript
// composables/useReducedMotion.ts
import { useMediaQuery } from '@vueuse/core'

export function useReducedMotion() {
  const prefersReducedMotion = useMediaQuery('(prefers-reduced-motion: reduce)')

  const transitionDuration = computed(() =>
    prefersReducedMotion.value ? 0 : 300
  )

  const shouldAnimate = computed(() => !prefersReducedMotion.value)

  return {
    prefersReducedMotion,
    transitionDuration,
    shouldAnimate
  }
}
```

```vue
<script setup>
import { useReducedMotion } from '~/composables/useReducedMotion'

const { shouldAnimate, transitionDuration } = useReducedMotion()
</script>

<template>
  <Transition :duration="transitionDuration">
    <div v-if="show && shouldAnimate">
      Contenu animé conditionnellement
    </div>
    <div v-else-if="show">
      Contenu sans animation
    </div>
  </Transition>
</template>
```

---
