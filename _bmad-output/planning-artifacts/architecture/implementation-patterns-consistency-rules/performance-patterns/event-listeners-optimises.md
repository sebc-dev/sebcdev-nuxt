# Event Listeners Optimisés

## Passive Listeners

Les événements `scroll`, `touchmove`, `wheel` bloquent le rendering si le handler peut appeler `preventDefault()`. Le flag `passive: true` indique qu'on ne le fera pas.

```typescript
import { useEventListener } from '@vueuse/core'

// ✅ CORRECT : passive pour événements continus
useEventListener(window, 'scroll', onScroll, { passive: true })
useEventListener(document, 'touchmove', onTouchMove, { passive: true })
useEventListener(document, 'wheel', onWheel, { passive: true })

// ✅ Dans les templates Vue avec modifier
// <div @scroll.passive="onScroll">
```

## Cleanup automatique avec VueUse

```typescript
// ❌ INCORRECT : pas de cleanup = memory leak
onMounted(() => {
  window.addEventListener('resize', handleResize)
})
// Si composant unmount → listener reste attaché

// ✅ CORRECT : VueUse gère le cleanup automatiquement
import { useEventListener } from '@vueuse/core'

useEventListener(window, 'resize', handleResize)
// Automatiquement nettoyé à l'unmount du composant
```

## Pattern combiné scroll optimisé

```typescript
import { useEventListener, useThrottleFn } from '@vueuse/core'

const scrollY = ref(0)
const scrollDirection = ref<'up' | 'down' | null>(null)
let lastScrollY = 0

const updateScrollInfo = useThrottleFn(() => {
  const currentY = window.scrollY
  scrollY.value = currentY
  scrollDirection.value = currentY > lastScrollY ? 'down' : 'up'
  lastScrollY = currentY
}, 100)

useEventListener(window, 'scroll', updateScrollInfo, { passive: true })
```

---
