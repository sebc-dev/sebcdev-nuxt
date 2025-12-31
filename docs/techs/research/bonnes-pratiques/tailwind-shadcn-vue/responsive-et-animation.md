# Responsive Design et Animations avec TailwindCSS 4 et shadcn-vue pour Nuxt 4 SSG

TailwindCSS 4.1.x révolutionne la configuration responsive avec une approche **CSS-first** via `@theme`, remplaçant le traditionnel `tailwind.config.js`. Pour un projet Nuxt 4 SSG déployé sur Cloudflare Pages, cette architecture offre des performances exceptionnelles : builds complets **3.8× plus rapides** et builds incrémentaux en microsecondes. Les animations shadcn-vue s'appuient sur Reka UI 2.7.0 et **tw-animate-css**, tandis que l'accessibilité `prefers-reduced-motion` est désormais activée par défaut via les variants `motion-safe:` et `motion-reduce:`.

## Mobile-first et breakpoints dans TailwindCSS 4

La philosophie mobile-first reste inchangée : les classes non-préfixées s'appliquent à toutes les tailles d'écran, les préfixes (`md:`, `lg:`) activent les styles à partir du breakpoint spécifié. La syntaxe CSS-native dans `@theme` remplace la configuration JavaScript :

```css
@import "tailwindcss";

@theme {
  /* Ajouter des breakpoints personnalisés */
  --breakpoint-xs: 20rem;      /* 320px */
  --breakpoint-3xl: 120rem;    /* 1920px */
  
  /* Remplacer un breakpoint existant */
  --breakpoint-sm: 30rem;      /* 480px au lieu de 640px */
}
```

Les **breakpoints par défaut** utilisent désormais `rem` : `sm` (**40rem**/640px), `md` (**48rem**/768px), `lg` (**64rem**/1024px), `xl` (**80rem**/1280px), `2xl` (**96rem**/1536px). Les variants `max-*` sont intégrés nativement (`max-md:hidden` pour cibler uniquement les écrans sous 768px).

Les **container queries** représentent l'évolution majeure de v4, intégrées au core sans plugin. La syntaxe utilise `@container` sur le parent et des variants `@sm`, `@md`, `@lg` sur les enfants :

```html
<div class="@container">
  <article class="flex flex-col @md:flex-row @lg:gap-8">
    <!-- S'adapte à la taille du conteneur, pas du viewport -->
  </article>
</div>
```

Cette approche permet de créer des composants réutilisables qui s'adaptent à leur contexte plutôt qu'à la fenêtre. Les variants container incluent `@3xs` (16rem) jusqu'à `@7xl` (80rem), avec support des noms (`@container/sidebar`) et des valeurs arbitraires (`@min-[475px]:`).

## Typographie fluide avec clamp() et @theme

L'implémentation de la typographie fluide dans TailwindCSS 4 passe par la définition de tokens personnalisés utilisant `clamp()`. Cette fonction CSS accepte trois paramètres : minimum, valeur préférée (généralement en `vw`), et maximum :

```css
@theme {
  /* Échelle typographique fluide complète */
  --text-fluid-body: clamp(1rem, 0.913rem + 0.4348vw, 1.2rem);
  --text-fluid-body--line-height: 1.6;
  
  --text-fluid-h1: clamp(2.4883rem, 1.9432rem + 2.7258vw, 2.986rem);
  --text-fluid-h1--line-height: 1.15;
  --text-fluid-h1--font-weight: 700;
  --text-fluid-h1--letter-spacing: -0.02em;
  
  --text-fluid-display: clamp(3rem, 2rem + 5vw, 6rem);
  --text-fluid-display--line-height: 1.05;
  
  /* Espacements fluides associés */
  --spacing-fluid-md: clamp(1rem, 0.5rem + 2.5vw, 2rem);
  --spacing-fluid-lg: clamp(2rem, 1rem + 5vw, 4rem);
}
```

L'utilisation devient alors simple et cohérente : `<h1 class="text-fluid-h1">`. Pour les cas ponctuels, la syntaxe arbitraire reste disponible : `text-[clamp(1.5rem,4vw,3rem)]`. Tailwind v4 n'inclut pas de support natif pour la typographie fluide, mais le plugin **fluid-tailwind** offre une syntaxe élégante `~text-lg/2xl` — noter cependant des problèmes de compatibilité v4 en cours de résolution. L'approche `@theme` avec tokens personnalisés reste la plus fiable pour Nuxt 4 SSG.

## Accessibilité et prefers-reduced-motion

Les variants **`motion-safe:`** et **`motion-reduce:`** sont activés par défaut dans TailwindCSS 4, mappant directement sur la media query `prefers-reduced-motion`. L'approche recommandée est l'opt-in : les animations ne s'activent que pour les utilisateurs n'ayant pas demandé leur réduction :

```html
<button class="
  bg-indigo-600 
  motion-safe:transition 
  motion-safe:duration-200 
  motion-safe:hover:scale-105
  hover:bg-indigo-700
">
  Action
</button>

<svg class="motion-safe:animate-spin motion-reduce:hidden h-5 w-5">
  <!-- Spinner avec fallback statique -->
</svg>
<span class="hidden motion-reduce:inline">⏳</span>
```

Pour les animations JavaScript, le composable `usePreferredReducedMotion` de VueUse détecte la préférence en temps réel :

```typescript
import { usePreferredReducedMotion } from '@vueuse/core'

const motionPreference = usePreferredReducedMotion()
const shouldAnimate = computed(() => motionPreference.value === 'no-preference')
const animationDuration = computed(() => shouldAnimate.value ? 300 : 0)
```

Les critères **WCAG 2.2** pertinents incluent le 2.2.2 (pause/stop/hide pour animations >5s), le 2.3.1 (pas plus de 3 flashs/seconde), et le 2.3.3 (animations déclenchées par interaction désactivables). Respecter `prefers-reduced-motion` satisfait généralement ces exigences.

## Vue Transitions intégrées avec Tailwind

Le composant `<Transition>` de Vue 3 s'intègre parfaitement avec les classes Tailwind via les props de classes personnalisées. Cette approche est idéale pour le SSG car les transitions CSS fonctionnent immédiatement sans hydratation :

```vue
<script setup lang="ts">
import { ref } from 'vue'
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

Pour `<TransitionGroup>`, ajouter `move-class="transition-all duration-300"` anime le repositionnement des éléments. Le mode `out-in` évite les chevauchements visuels lors des transitions de pages Nuxt :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  app: {
    pageTransition: { name: 'page', mode: 'out-in' },
    layoutTransition: { name: 'layout', mode: 'out-in' }
  }
})
```

Un composable réutilisable simplifie la création de transitions cohérentes :

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
```

## Animations shadcn-vue et Reka UI 2.7.0

shadcn-vue utilise **tw-animate-css** pour Tailwind v4, combiné aux attributs `data-state` de Reka UI. L'installation est obligatoire pour les animations des Dialog, Sheet, Dropdown :

```css
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&:is(.dark *));
```

Les composants utilisent les sélecteurs `data-[state=open]` et `data-[state=closed]` avec les utilitaires `animate-in`, `animate-out`, `fade-in`, `zoom-in-95`, `slide-in-from-right` :

```html
<DialogContent class="
  data-[state=open]:animate-in 
  data-[state=open]:fade-in-0 
  data-[state=open]:zoom-in-95
  data-[state=closed]:animate-out 
  data-[state=closed]:fade-out-0 
  data-[state=closed]:zoom-out-95
  data-[state=closed]:duration-200 
  data-[state=open]:duration-300
">
```

Reka UI 2.7.0 expose des **variables CSS pour les dimensions** (`--reka-accordion-content-height`), permettant des animations de hauteur fluides. La prop `forceMount` maintient l'élément dans le DOM pendant les animations de sortie, nécessaire pour Vue `<Transition>` ou Motion Vue.

Pour les transitions de thème avec **oklch**, animer les propriétés CSS custom offre des dégradés perceptuellement uniformes :

```css
body {
  transition: background-color 0.3s ease, color 0.3s ease;
  background-color: oklch(var(--bg-lightness) 0.02 var(--hue));
}
```

## Anti-patterns critiques à éviter

Plusieurs erreurs compromettent fréquemment les performances et l'accessibilité des animations en contexte SSG :

- **Animer des propriétés de layout** (`width`, `height`, `margin`) déclenche des recalculs coûteux. Préférer `transform` et `opacity`, accélérés GPU
- **Conflits Vue Transition + CSS animations** : utiliser un seul système, pas les deux simultanément sur le même élément
- **Oublier tw-animate-css** : les animations shadcn-vue échouent silencieusement sans cette dépendance
- **Ignorer `prefers-reduced-motion`** : affecte **70+ millions** de personnes avec troubles vestibulaires
- **Memory leaks** : nettoyer les animations JavaScript (GSAP, Motion Vue) dans `onUnmounted`
- **Flash au chargement SSG** : différer les animations après hydratation avec `onMounted`

```vue
<script setup>
const mounted = ref(false)
onMounted(() => mounted.value = true)
</script>

<template>
  <!-- Évite le flash d'animation avant hydratation -->
  <div :class="{ 'animate-in fade-in': mounted }">
    Contenu
  </div>
</template>
```

Pour Cloudflare Pages, utiliser `nuxt generate` avec le preset `static` ou `cloudflare_pages` pour SSR edge. Les animations CSS pures fonctionnent immédiatement, tandis que les animations JavaScript nécessitent l'hydratation complète. Privilégier `transform` et `opacity` garantit **60fps** sur mobile.

## Conclusion

L'écosystème TailwindCSS 4 + shadcn-vue + Nuxt 4 SSG offre une base solide pour des interfaces performantes et accessibles. Les points clés à retenir : configurer exclusivement via `@theme` CSS-native, utiliser les container queries pour les composants réutilisables, implémenter `motion-safe:` systématiquement pour l'accessibilité, et s'appuyer sur tw-animate-css pour les animations shadcn-vue. Les tokens `clamp()` personnalisés pour la typographie fluide évitent les dépendances externes fragiles. Enfin, tester avec `prefers-reduced-motion: reduce` activé dans les paramètres système révèle les oublis d'accessibilité avant mise en production.