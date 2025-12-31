# Accessibilité des Animations (prefers-reduced-motion)

## Variants Tailwind motion-safe / motion-reduce

TailwindCSS 4 active par défaut les variants `motion-safe:` et `motion-reduce:`, mappant sur la media query `prefers-color-scheme`. L'approche recommandée est l'**opt-in** : les animations ne s'activent que pour les utilisateurs n'ayant pas demandé leur réduction.

```html
<!-- Pattern opt-in : animation uniquement si autorisée -->
<button class="
  bg-indigo-600
  motion-safe:transition
  motion-safe:duration-200
  motion-safe:hover:scale-105
  hover:bg-indigo-700
">
  Action
</button>

<!-- Spinner avec fallback statique -->
<svg class="motion-safe:animate-spin motion-reduce:hidden h-5 w-5">
  <!-- Spinner animé -->
</svg>
<span class="hidden motion-reduce:inline">⏳</span>
```

## Détection JavaScript avec VueUse

Pour les animations JavaScript (GSAP, Motion Vue), utiliser `usePreferredReducedMotion` de VueUse :

```typescript
import { usePreferredReducedMotion } from '@vueuse/core'

const motionPreference = usePreferredReducedMotion()
const shouldAnimate = computed(() => motionPreference.value === 'no-preference')
const animationDuration = computed(() => shouldAnimate.value ? 300 : 0)
```

## Critères WCAG 2.2

| Critère | Exigence | Impact |
|---------|----------|--------|
| **2.2.2** | Pause/stop/hide pour animations >5s | Carousels, backgrounds animés |
| **2.3.1** | Pas plus de 3 flashs/seconde | Risque épilepsie |
| **2.3.3** | Animations déclenchées par interaction désactivables | Respect `prefers-reduced-motion` |

**Recommandation** : Respecter `prefers-reduced-motion` satisfait généralement ces trois exigences.

---
