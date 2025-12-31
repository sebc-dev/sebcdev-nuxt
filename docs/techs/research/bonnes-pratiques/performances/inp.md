# Optimisation INP pour Nuxt 4.2.x SSG : Guide technique complet

L'Interaction to Next Paint (INP) est devenu Core Web Vital en mars 2024, avec un seuil optimal de **≤200ms**. Pour un blog Nuxt 4.2.x déployé en SSG sur Cloudflare Pages, l'optimisation passe par quatre axes majeurs : lazy hydration intelligente, code splitting stratégique, scheduling des tâches non-critiques, et optimisation des event handlers. Ce guide fournit des patterns immédiatement applicables avec la syntaxe actuelle de décembre 2025.

---

## Lazy hydration : la clé de l'INP en SSG

Nuxt 4 intègre nativement les stratégies d'hydratation lazy de Vue 3.5, permettant de différer l'hydratation des composants selon leur criticité. Cette approche réduit drastiquement le blocage du main thread lors du chargement initial.

### Directives d'hydratation disponibles

Le préfixe `Lazy` combiné aux directives `hydrate-on-*` contrôle précisément quand chaque composant s'hydrate :

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

L'événement `@hydrated` permet de déclencher des actions post-hydratation, utile pour l'analytics ou l'initialisation différée.

### Nuxt Islands pour les composants serveur-only

Les Islands éliminent totalement le JavaScript client pour les composants purement statiques ou utilisant des librairies lourdes côté serveur :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  experimental: {
    componentIslands: {
      selectiveClient: true  // Permet nuxt-client dans les islands
    }
  }
})
```

```vue
<!-- components/SyntaxHighlighter.server.vue -->
<script setup lang="ts">
import shiki from 'shiki'
// Librairie lourde exécutée uniquement côté serveur
const highlighter = await shiki.getHighlighter({ theme: 'nord' })
const html = highlighter.codeToHtml(props.code, { lang: props.lang })
</script>

<template>
  <div v-html="html" />
</template>
```

### Stratégie de décision hydratation

| Cas d'usage | Stratégie recommandée |
|-------------|----------------------|
| Footer, mentions légales | `hydrate-never` ou `.server.vue` |
| Syntax highlighting, markdown processing | `.server.vue` (Island) |
| Composant critique above-fold | Hydratation normale (pas de Lazy) |
| Composant above-fold non-critique | `hydrate-on-idle` |
| Contenu below-fold | `hydrate-on-visible` |
| Formulaires, modals, dropdowns | `hydrate-on-interaction` |
| Navigation mobile-only | `hydrate-on-media-query` |

### Anti-patterns à éviter absolument

```vue
<!-- ❌ MAL : Lazy hydration sur contenu critique above-fold -->
<LazyHeroButton hydrate-on-visible />

<!-- ❌ MAL : hydrate-never sur composant interactif -->
<LazyContactForm hydrate-never />

<!-- ❌ MAL : Spreading de props (force hydratation immédiate) -->
<LazyMyComponent v-bind="propsObject" hydrate-on-visible />

<!-- ✅ BIEN : Props explicites -->
<LazyMyComponent :title="title" :data="data" hydrate-on-visible />
```

---

## Code splitting et lazy loading optimisés

### defineAsyncComponent avec stratégies d'hydratation Vue 3.5

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

### Configuration chunks Vite optimale pour SSG

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true,
  
  routeRules: {
    '/': { prerender: true },
    '/blog/**': { prerender: true },
    '/tags/**': { prerender: true }
  },
  
  experimental: {
    defaults: {
      nuxtLink: {
        prefetchOn: {
          visibility: true,
          interaction: false  // Évite prefetch excessif
        }
      }
    }
  },
  
  vite: {
    build: {
      cssCodeSplit: true,
      rollupOptions: {
        output: {
          manualChunks(id) {
            if (id.includes('node_modules')) {
              // Core framework séparé
              if (id.includes('vue') || id.includes('pinia')) {
                return 'vendor-core'
              }
              // UI components (shadcn-vue, Reka UI)
              if (id.includes('reka-ui') || id.includes('radix-vue')) {
                return 'vendor-ui'
              }
              // Utilitaires
              if (id.includes('@vueuse') || id.includes('clsx')) {
                return 'vendor-utils'
              }
              return 'vendor'
            }
          },
          // Nommage hash-only pour éviter blocage adblockers
          chunkFileNames: '_nuxt/[hash].js',
          entryFileNames: '_nuxt/[hash].js',
          assetFileNames: '_nuxt/[hash].[ext]'
        }
      }
    }
  }
})
```

**Tailles cibles** : chunks de **100-250 KB** (30-80 KB gzippé), warning Vite à 500 KB.

### Prefetching stratégique NuxtLink

```vue
<template>
  <!-- Default : prefetch à la visibilité -->
  <NuxtLink to="/about">À propos</NuxtLink>
  
  <!-- Désactivé pour pages lourdes -->
  <NuxtLink to="/heavy-dashboard" :prefetch="false">Dashboard</NuxtLink>
  
  <!-- Prefetch au hover uniquement -->
  <NuxtLink to="/contact" prefetch-on="interaction">Contact</NuxtLink>
</template>
```

---

## Scheduling et yield : différer les tâches non-critiques

### Support navigateurs décembre 2025

| API | Chrome | Edge | Firefox | Safari |
|-----|--------|------|---------|--------|
| `requestIdleCallback` | 47+ | 79+ | 55+ | ❌ (flag) |
| `scheduler.postTask` | 94+ | 94+ | 101+ | ❌ |
| `scheduler.yield` | 129+ | 129+ | 142+ | ❌ |

### Composable universel avec fallbacks

```typescript
// composables/useScheduler.ts
export function useScheduler() {
  const idleCallbackId = ref<number | null>(null)
  
  /**
   * Exécute une tâche pendant les idle periods
   */
  function scheduleIdle(
    callback: () => void, 
    options: { timeout?: number } = { timeout: 2000 }
  ) {
    if (idleCallbackId.value !== null) {
      cancelIdleCallback(idleCallbackId.value)
    }
    
    if ('requestIdleCallback' in window) {
      idleCallbackId.value = requestIdleCallback(callback, options)
    } else {
      // Fallback Safari
      setTimeout(callback, 1)
    }
  }
  
  /**
   * Schedule avec priorité (Scheduler API)
   */
  async function scheduleTask(
    callback: () => any,
    priority: 'user-blocking' | 'user-visible' | 'background' = 'background'
  ): Promise<any> {
    if ('scheduler' in globalThis && 'postTask' in (globalThis as any).scheduler) {
      return (globalThis as any).scheduler.postTask(callback, { priority })
    }
    
    // Fallback requestIdleCallback pour background
    if (priority === 'background' && 'requestIdleCallback' in window) {
      return new Promise(resolve => {
        requestIdleCallback(() => resolve(callback()))
      })
    }
    
    // Fallback final setTimeout
    return new Promise(resolve => {
      setTimeout(() => resolve(callback()), priority === 'user-blocking' ? 0 : 1)
    })
  }
  
  /**
   * Yield au main thread avec continuation prioritaire
   */
  function yieldToMain(): Promise<void> {
    if ((globalThis as any).scheduler?.yield) {
      return (globalThis as any).scheduler.yield()
    }
    return new Promise(resolve => setTimeout(resolve, 0))
  }
  
  onUnmounted(() => {
    if (idleCallbackId.value !== null) {
      cancelIdleCallback(idleCallbackId.value)
    }
  })
  
  return { scheduleIdle, scheduleTask, yieldToMain }
}
```

### Pattern de traitement par chunks avec yield

```typescript
// composables/useChunkedProcessor.ts
export function useChunkedProcessor<T, R>(
  processor: (item: T) => R,
  options: { chunkSize?: number; yieldInterval?: number } = {}
) {
  const { chunkSize = 10, yieldInterval = 50 } = options
  const isProcessing = ref(false)
  const progress = ref(0)
  
  async function processItems(items: T[]): Promise<R[]> {
    isProcessing.value = true
    progress.value = 0
    
    const results: R[] = []
    let lastYield = performance.now()
    
    for (let i = 0; i < items.length; i++) {
      results.push(processor(items[i]))
      progress.value = ((i + 1) / items.length) * 100
      
      // Yield toutes les ~50ms pour rester sous le seuil long task
      if (performance.now() - lastYield > yieldInterval) {
        await ((globalThis as any).scheduler?.yield?.() ?? 
               new Promise(r => setTimeout(r, 0)))
        lastYield = performance.now()
      }
    }
    
    isProcessing.value = false
    return results
  }
  
  return { processItems, isProcessing, progress }
}
```

### Utilisation dans les lifecycle hooks

```vue
<script setup lang="ts">
const { scheduleIdle, scheduleTask, yieldToMain } = useScheduler()

onMounted(async () => {
  // 1. Critique : initialisation UI immédiate
  initializeCriticalUI()
  
  // 2. Yield avant travail non-critique
  await yieldToMain()
  
  // 3. Travail user-visible mais différable
  await scheduleTask(async () => {
    await fetchSecondaryData()
  }, 'user-visible')
  
  // 4. Background : analytics, prefetch
  scheduleIdle(() => {
    initAnalytics()
    prefetchRelatedContent()
    warmImageCache()
  }, { timeout: 5000 })
})
</script>
```

---

## Event handlers performants

### Pattern critique : travail minimal dans les handlers

```vue
<script setup lang="ts">
const { yieldToMain } = useScheduler()

// ❌ MAL : Tout synchrone
const badHandler = () => {
  heavyComputation()
  updateAnalytics()
  saveToLocalStorage()
  items.value = result
}

// ✅ BIEN : UI d'abord, puis yield et defer
const goodHandler = async () => {
  // 1. Update UI critique immédiatement
  isLoading.value = true
  
  // 2. Yield pour permettre le paint
  await yieldToMain()
  
  // 3. Travail non-critique après paint
  requestAnimationFrame(() => {
    setTimeout(() => {
      updateAnalytics()
      saveToLocalStorage()
    }, 0)
  })
}
</script>
```

### Debounce et throttle avec VueUse

```vue
<script setup lang="ts">
import { useDebounceFn, useThrottleFn } from '@vueuse/core'

// Search : debounce 300ms avec maxWait 2s
const debouncedSearch = useDebounceFn(
  async (query: string) => {
    if (query.length < 2) return
    searchResults.value = await fetchResults(query)
  },
  300,
  { maxWait: 2000 }
)

// Scroll : throttle 100ms avec passive listener
const throttledScroll = useThrottleFn(() => {
  scrollPosition.value = window.scrollY
  isHeaderVisible.value = scrollPosition.value < 200
}, 100)

onMounted(() => {
  window.addEventListener('scroll', throttledScroll, { passive: true })
})
</script>

<template>
  <input @input="e => debouncedSearch(e.target.value)" />
</template>
```

**Valeurs recommandées** :
- Search input : **300-500ms** debounce
- Validation formulaire : **250-400ms** debounce
- Scroll/resize : **50-100ms** throttle
- Clicks : **aucun délai**

### Passive listeners pour scroll et touch

```vue
<template>
  <!-- ✅ Passive scroll (ne peut pas preventDefault) -->
  <div @scroll.passive="handleScroll" class="overflow-auto">
    <div v-for="item in items" :key="item.id">{{ item.name }}</div>
  </div>
  
  <!-- ✅ Passive touch pour mobile -->
  <div 
    @touchstart.passive="handleTouchStart"
    @touchmove.passive="handleTouchMove"
  >
    Contenu swipeable
  </div>
</template>
```

### Event delegation pour listes

```vue
<script setup lang="ts">
// Un seul handler délégué au lieu de N handlers
const handleListClick = (event: MouseEvent) => {
  const listItem = (event.target as HTMLElement).closest('[data-item-id]')
  if (!listItem) return
  
  const itemId = parseInt(listItem.dataset.itemId!)
  selectItem(itemId)
}
</script>

<template>
  <ul @click="handleListClick">
    <li 
      v-for="item in items" 
      :key="item.id"
      :data-item-id="item.id"
    >
      {{ item.name }}
    </li>
  </ul>
</template>
```

### v-memo pour listes larges (1000+ items)

```vue
<template>
  <!-- Re-render uniquement quand selection change -->
  <div 
    v-for="item in items" 
    :key="item.id"
    v-memo="[item.id === selectedId]"
    :class="{ 'bg-primary': item.id === selectedId }"
    @click="selectedId = item.id"
  >
    <h3>{{ item.title }}</h3>
    <p>{{ item.description }}</p>
  </div>
</template>
```

---

## Mesure et monitoring INP

### Intégration web-vitals

```typescript
// plugins/web-vitals.client.ts
import { onINP, onLCP, onCLS } from 'web-vitals/attribution'

export default defineNuxtPlugin(() => {
  const sendMetric = (metric: any) => {
    const body = JSON.stringify({
      name: metric.name,
      value: metric.value,
      rating: metric.rating,
      page: window.location.pathname,
      // Attribution pour debug
      attribution: metric.attribution ? {
        interactionTarget: metric.attribution.interactionTarget,
        inputDelay: metric.attribution.inputDelay,
        processingDuration: metric.attribution.processingDuration,
        presentationDelay: metric.attribution.presentationDelay
      } : undefined
    })
    
    // sendBeacon pour fiabilité au page unload
    if (navigator.sendBeacon) {
      navigator.sendBeacon('/api/analytics', body)
    }
  }
  
  onINP(sendMetric)
  onLCP(sendMetric)
  onCLS(sendMetric)
})
```

### Debug console pour développement

```typescript
// plugins/inp-debug.client.ts
export default defineNuxtPlugin(() => {
  if (import.meta.env.DEV) {
    import('web-vitals/attribution').then(({ onINP }) => {
      onINP((metric) => {
        const rating = metric.rating === 'good' ? '🟢' : 
                       metric.rating === 'needs-improvement' ? '🟡' : '🔴'
        
        console.log(`${rating} INP: ${metric.value.toFixed(0)}ms`)
        
        if (metric.attribution) {
          console.table({
            'Input Delay': `${metric.attribution.inputDelay.toFixed(0)}ms`,
            'Processing': `${metric.attribution.processingDuration.toFixed(0)}ms`,
            'Presentation': `${metric.attribution.presentationDelay.toFixed(0)}ms`,
            'Target': metric.attribution.interactionTarget
          })
        }
      })
    })
  }
})
```

---

## Checklist actionnable pour Claude Code

### Configuration initiale

- [ ] Activer `componentIslands` dans nuxt.config.ts
- [ ] Configurer `manualChunks` Vite pour vendor splitting
- [ ] Installer `web-vitals` et configurer plugin client
- [ ] Installer `scheduler-polyfill` pour support Safari

### Composants existants à auditer

- [ ] Identifier composants below-fold → ajouter `hydrate-on-visible`
- [ ] Identifier composants interactifs non-critiques → ajouter `hydrate-on-interaction`
- [ ] Convertir composants statiques (footer, sidebar) en `.server.vue` ou `hydrate-never`
- [ ] Vérifier absence de props spreading sur composants lazy

### Event handlers à optimiser

- [ ] Search inputs : debounce 300-500ms avec VueUse
- [ ] Scroll handlers : throttle 50-100ms + `.passive`
- [ ] Listes > 50 items : implémenter event delegation
- [ ] Listes > 1000 items : ajouter `v-memo`

### Code patterns à appliquer

- [ ] Utiliser `scheduleIdle` pour analytics/tracking
- [ ] Implémenter yield toutes les ~50ms dans boucles longues
- [ ] Séparer UI critique du travail non-critique dans handlers
- [ ] Batch DOM reads avant DOM writes

### Mesure et validation

- [ ] Vérifier INP < 200ms dans DevTools Performance
- [ ] Tester avec CPU 4x throttling (simulation mobile)
- [ ] Analyser chunks avec `npx nuxi analyze`
- [ ] Monitorer CrUX data via PageSpeed Insights

---

## Configuration nuxt.config.ts complète

```typescript
// nuxt.config.ts - Configuration optimisée INP
export default defineNuxtConfig({
  ssr: true,
  
  experimental: {
    componentIslands: { selectiveClient: true },
    defaults: {
      nuxtLink: {
        prefetchOn: { visibility: true, interaction: false }
      }
    }
  },
  
  routeRules: {
    '/': { prerender: true },
    '/blog/**': { prerender: true },
    '/tags/**': { prerender: true }
  },
  
  vite: {
    build: {
      cssCodeSplit: true,
      rollupOptions: {
        output: {
          manualChunks(id) {
            if (id.includes('node_modules')) {
              if (id.includes('vue') || id.includes('pinia')) return 'vendor-core'
              if (id.includes('reka-ui') || id.includes('radix-vue')) return 'vendor-ui'
              if (id.includes('@vueuse')) return 'vendor-utils'
              return 'vendor'
            }
          },
          chunkFileNames: '_nuxt/[hash].js',
          entryFileNames: '_nuxt/[hash].js',
          assetFileNames: '_nuxt/[hash].[ext]'
        }
      }
    }
  },
  
  nitro: {
    preset: 'cloudflare-pages-static'
  }
})
```

L'optimisation INP pour un blog Nuxt SSG repose sur une stratégie d'hydratation progressive où seuls les composants critiques s'hydratent immédiatement. Combinée au scheduling intelligent des tâches non-critiques et à l'optimisation des event handlers, cette approche permet d'atteindre systématiquement le seuil de **200ms** requis par Google pour les Core Web Vitals.