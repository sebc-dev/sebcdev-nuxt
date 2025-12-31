# Lazy Hydration Strategy

## Stratégie de décision hydratation

Table de mapping pour choisir la stratégie d'hydratation appropriée selon le cas d'usage :

| Cas d'usage | Stratégie recommandée | Directive |
|-------------|----------------------|-----------|
| Footer, mentions légales | Jamais hydrater | `hydrate-never` ou `.server.vue` |
| Syntax highlighting, markdown processing | Island serveur | `.server.vue` |
| Composant critique above-fold | Hydratation normale | Pas de `Lazy` prefix |
| Composant above-fold non-critique | Idle time | `hydrate-on-idle` |
| Contenu below-fold | Visibilité viewport | `hydrate-on-visible` |
| Formulaires, modals, dropdowns | Interaction utilisateur | `hydrate-on-interaction` |
| Navigation mobile-only | Media query | `hydrate-on-media-query="(max-width: 768px)"` |
| Promo banner différée | Délai fixe | `:hydrate-after="3000"` |
| Fonctionnalité conditionnelle | Condition booléenne | `:hydrate-when="userIsPremium"` |

## Exemple complet directives hydratation

```vue
<template>
  <!-- Hydrate quand visible dans le viewport -->
  <LazyCommentsSection
    hydrate-on-visible
    :hydrate-on-visible="{ rootMargin: '100px' }"
  />

  <!-- Hydrate pendant les idle periods du navigateur -->
  <LazyAnalyticsWidget :hydrate-on-idle="2000" />

  <!-- Hydrate au premier engagement utilisateur -->
  <LazyNewsletterForm hydrate-on-interaction />
  <LazyShareButtons :hydrate-on-interaction="['click', 'touchstart']" />

  <!-- Hydrate selon media query (responsive) -->
  <LazyMobileNav hydrate-on-media-query="(max-width: 768px)" />

  <!-- Hydrate après délai fixe -->
  <LazyPromoBanner :hydrate-after="3000" />

  <!-- Hydrate conditionnellement -->
  <LazyPremiumFeature :hydrate-when="userIsPremium" />

  <!-- Ne jamais hydrater - contenu statique pur -->
  <LazyFooter hydrate-never />
</template>
```

L'événement `@hydrated` permet de déclencher des actions post-hydratation :

```vue
<LazyHeavyComponent hydrate-on-visible @hydrated="onComponentReady" />
```

## Anti-patterns hydratation à éviter

```vue
<!-- ❌ MAL : Lazy hydration sur contenu critique above-fold -->
<LazyHeroButton hydrate-on-visible />

<!-- ❌ MAL : hydrate-never sur composant interactif -->
<LazyContactForm hydrate-never />

<!-- ❌ MAL : Spreading de props (force hydratation immédiate) -->
<LazyMyComponent v-bind="propsObject" hydrate-on-visible />

<!-- ✅ BIEN : Props explicites préservent l'hydratation lazy -->
<LazyMyComponent :title="title" :data="data" hydrate-on-visible />
```

| Anti-pattern | Problème | Solution |
|--------------|----------|----------|
| `v-bind="obj"` sur Lazy | Force hydratation immédiate | Props explicites |
| `hydrate-never` sur interactif | Composant non-fonctionnel | `hydrate-on-interaction` |
| Lazy sur contenu critique | Délai perçu, mauvais UX | Pas de prefix `Lazy` |
| `hydrate-on-visible` above-fold | Hydratation tardive inutile | `hydrate-on-idle` ou normal |

## defineAsyncComponent avec stratégies Vue 3.5

API programmatique pour contrôler l'hydratation des composants chargés dynamiquement :

```typescript
import {
  defineAsyncComponent,
  hydrateOnIdle,
  hydrateOnVisible,
  hydrateOnInteraction
} from 'vue'

// Composant lourd hydraté pendant idle
const HeavyChart = defineAsyncComponent({
  loader: () => import('./components/Chart.vue'),
  loadingComponent: ChartSkeleton,
  delay: 200,
  timeout: 10000,
  hydrate: hydrateOnIdle(2000)
})

// Composant below-fold avec IntersectionObserver
const RelatedPosts = defineAsyncComponent({
  loader: () => import('./components/RelatedPosts.vue'),
  hydrate: hydrateOnVisible({ rootMargin: '200px' })
})

// Widget interactif hydraté au hover/click
const ShareDialog = defineAsyncComponent({
  loader: () => import('./components/ShareDialog.vue'),
  hydrate: hydrateOnInteraction(['mouseover', 'click', 'focus'])
})
```

**Quand utiliser :** Pour les composants importés dynamiquement dans le code (pas dans les templates). Les directives `hydrate-on-*` sont préférées pour les composants templates.

---
