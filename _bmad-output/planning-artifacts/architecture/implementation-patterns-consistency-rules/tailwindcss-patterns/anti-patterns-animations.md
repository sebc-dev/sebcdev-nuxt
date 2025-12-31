# Anti-patterns Animations

## ❌ Animer des propriétés de layout

```css
/* ❌ Déclenche des recalculs coûteux (reflow) */
.element {
  transition: width 300ms, height 300ms, margin 300ms;
}

/* ✅ Utiliser transform et opacity (accélérés GPU) */
.element {
  transition: transform 300ms, opacity 300ms;
}
```

| Propriété | Performance | Alternative |
|-----------|-------------|-------------|
| `width`, `height` | ❌ Reflow | `transform: scale()` |
| `margin`, `padding` | ❌ Reflow | `transform: translate()` |
| `top`, `left`, `right`, `bottom` | ❌ Reflow | `transform: translate()` |
| `transform`, `opacity` | ✅ Composite only | — |

## ❌ Conflits Vue Transition + CSS animations

```vue
<!-- ❌ Les deux systèmes entrent en conflit -->
<Transition enter-active-class="transition-all duration-300">
  <div class="animate-in fade-in">Conflit</div>
</Transition>

<!-- ✅ Utiliser un seul système -->
<Transition enter-active-class="transition-all duration-300 ease-out">
  <div>Un seul système</div>
</Transition>
```

## ❌ Oublier tw-animate-css

Les animations shadcn-vue (`animate-in`, `fade-in-0`, etc.) **échouent silencieusement** sans l'import :

```css
/* ❌ Animations shadcn-vue ne fonctionnent pas */
@import "tailwindcss";

/* ✅ Import obligatoire */
@import "tailwindcss";
@import "tw-animate-css";
```

## ❌ Memory leaks animations JavaScript

```typescript
// ❌ Animation non nettoyée
onMounted(() => {
  gsap.to('.element', { x: 100, duration: 1 })
})

// ✅ Cleanup dans onUnmounted
const animation = ref<gsap.core.Tween | null>(null)

onMounted(() => {
  animation.value = gsap.to('.element', { x: 100, duration: 1 })
})

onUnmounted(() => {
  animation.value?.kill()
})
```

## Pattern anti-flash SSG

Les animations peuvent déclencher un flash visuel au chargement SSG si elles s'exécutent avant hydratation. Différer avec `onMounted` :

```vue
<script setup>
const mounted = ref(false)
onMounted(() => mounted.value = true)
</script>

<template>
  <!-- Animation différée après hydratation -->
  <div :class="{ 'animate-in fade-in duration-500': mounted }">
    Contenu sans flash
  </div>

  <!-- Alternative : motion-safe pour respecter aussi les préférences -->
  <div :class="mounted && 'motion-safe:animate-in motion-safe:fade-in'">
    Contenu accessible
  </div>
</template>
```

**Pourquoi c'est nécessaire** : En SSG, le HTML statique contient les classes d'animation. Sans `onMounted`, l'animation se déclenche au parsing CSS, avant que Vue ne soit hydraté, causant un flash visuel.

---
