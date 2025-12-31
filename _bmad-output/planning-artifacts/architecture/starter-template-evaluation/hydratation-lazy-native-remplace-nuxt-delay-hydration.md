# Hydratation Lazy Native (remplace nuxt-delay-hydration)

```vue
<template>
  <!-- Hydrate quand visible dans le viewport -->
  <LazyExpensiveComponent hydrate-on-visible />
  
  <!-- Hydrate pendant l'idle time du navigateur -->
  <LazyHeavyComponent hydrate-on-idle />
  
  <!-- Hydrate après un délai fixe (2s) -->
  <LazyDelayedComponent :hydrate-after="2000" />
  
  <!-- Hydrate sur interaction (hover, click, focus) -->
  <LazyInteractiveComponent hydrate-on-interaction />
</template>
```
