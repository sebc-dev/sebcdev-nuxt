# Performance Patterns

Patterns d'optimisation des performances pour les Core Web Vitals (LCP, INP, CLS) et la réactivité utilisateur.

---

## Lazy Hydration Strategy

### Stratégie de décision hydratation

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

### Exemple complet directives hydratation

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

### Anti-patterns hydratation à éviter

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

### defineAsyncComponent avec stratégies Vue 3.5

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

## LCP (Largest Contentful Paint)

### Diagnostic des sous-parties LCP

Le LCP se décompose en 4 phases. Identifier la phase problématique permet de cibler les optimisations :

| Phase | Cible | Problèmes courants |
|-------|-------|-------------------|
| **TTFB** (Time to First Byte) | <40% du LCP | Serveur lent, redirections, pas de CDN |
| **Resource Load Delay** | <10% | Image découverte tard, JS-dependent |
| **Resource Load Duration** | ~40% | Image trop lourde, pas de CDN |
| **Element Render Delay** | <10% | CSS/JS bloquant, main thread occupé |

**Comment identifier ces phases :**
1. Chrome DevTools → Performance → Live Metrics → Survol du métrique LCP
2. Utiliser le plugin `web-vitals` avec attribution (voir ci-dessous)

### Plugin web-vitals avec attribution détaillée

Plugin client pour monitoring des Core Web Vitals en développement et optionnellement en production :

```typescript
// plugins/web-vitals.client.ts
import { onLCP, onINP, onCLS } from 'web-vitals/attribution'

export default defineNuxtPlugin(() => {
  const logMetric = (metric: any) => {
    const data = {
      name: metric.name,
      value: metric.value,
      rating: metric.rating,  // 'good' | 'needs-improvement' | 'poor'
      delta: metric.delta,
      id: metric.id,
      attribution: metric.attribution,
      url: window.location.href
    }

    // Debug en développement uniquement
    if (import.meta.dev) {
      if (metric.name === 'LCP') {
        console.group('🎯 LCP Attribution')
        console.log('Value:', metric.value, 'ms')
        console.log('Rating:', metric.rating)
        console.log('Element:', metric.attribution.element)
        console.log('Resource Load Delay:', metric.attribution.resourceLoadDelay, 'ms')
        console.log('Resource Load Duration:', metric.attribution.resourceLoadDuration, 'ms')
        console.log('Element Render Delay:', metric.attribution.elementRenderDelay, 'ms')
        console.groupEnd()
      }

      if (metric.name === 'INP') {
        console.group('⚡ INP Attribution')
        console.log('Value:', metric.value, 'ms')
        console.log('Target:', metric.attribution.interactionTarget)
        console.log('Input Delay:', metric.attribution.inputDelay, 'ms')
        console.log('Processing Duration:', metric.attribution.processingDuration, 'ms')
        console.log('Presentation Delay:', metric.attribution.presentationDelay, 'ms')
        console.groupEnd()
      }

      if (metric.name === 'CLS') {
        console.group('📐 CLS Attribution')
        console.log('Value:', metric.value)
        console.log('Largest Shift Target:', metric.attribution.largestShiftTarget)
        console.log('Largest Shift Time:', metric.attribution.largestShiftTime, 'ms')
        console.groupEnd()
      }
    }
  }

  onLCP(logMetric)
  onINP(logMetric)
  onCLS(logMetric)
})
```

**Installation :**

```bash
pnpm add web-vitals
```

**Note SSG** : Pour collecter les métriques en production, un service externe est nécessaire (Cloudflare Web Analytics est gratuit et intégré si hébergé sur CF).

### Seuils Core Web Vitals

| Métrique | Bon | À améliorer | Mauvais |
|----------|-----|-------------|---------|
| **LCP** | ≤2.5s | 2.5s - 4s | >4s |
| **INP** | ≤200ms | 200ms - 500ms | >500ms |
| **CLS** | ≤0.1 | 0.1 - 0.25 | >0.25 |

### Checklist LCP Nuxt 4 + Cloudflare Pages

```markdown
## Configuration initiale
- [ ] `@nuxt/image` configuré avec formats `['avif', 'webp']`
- [ ] `@nuxt/fonts` avec provider local pour fonts self-hosted
- [ ] `@nuxtjs/critters` pour extraction CSS critique
- [ ] TailwindCSS 4.x via `@tailwindcss/vite` plugin
- [ ] `nitro.preset: 'cloudflare_pages'` ou `cloudflare_pages_static`

## Image LCP (1 par page)
- [ ] Utiliser `<NuxtPicture>` avec `format="avif,webp"`
- [ ] Ajouter `:preload="{ fetchPriority: 'high' }"`
- [ ] Définir `loading="eager"`
- [ ] Spécifier `width` et `height` explicites
- [ ] Configurer `sizes` pour responsive

## Images below-the-fold
- [ ] `loading="lazy"` sur toutes les images non-LCP
- [ ] `fetchpriority="low"` pour images décoratives
- [ ] Pas de `preload`

## Fonts
- [ ] Self-hosting WOFF2 dans `public/fonts/`
- [ ] Preload de la police principale avec `crossorigin="anonymous"`
- [ ] `font-display: optional` pour zéro CLS
- [ ] `line-height` explicite sur `body`
- [ ] Subsets limités (`latin`, `latin-ext`)

## Cloudflare Pages
- [ ] Fichier `public/_headers` créé
- [ ] Cache immutable pour `/_nuxt/*`
- [ ] `X-Robots-Tag: noindex` sur `*.pages.dev`
- [ ] Early Hints Link headers configurés

## Monitoring
- [ ] Plugin `web-vitals.client.ts` en place (dev)
- [ ] Tests PageSpeed Insights mobile < 2.5s LCP
- [ ] Lighthouse CI configuré avec assertions Core Web Vitals

## Validation build SSG
- [ ] `pnpm run generate` sans erreurs
- [ ] Images optimisées dans `.output/public/_ipx/`
- [ ] Taille bundle CSS < 50KB gzipped
- [ ] Pas de console errors Core Web Vitals
```

---

## CLS (Cumulative Layout Shift)

### Configuration NuxtImg pour CLS zéro

La clé fondamentale : **toujours spécifier `width` et `height`** sur chaque `<NuxtImg>`. Ces attributs permettent au navigateur de calculer l'aspect-ratio et réserver l'espace avant le chargement.

```vue
<template>
  <!-- Image LCP (above-the-fold) : preload + eager -->
  <NuxtImg
    src="/hero-banner.jpg"
    width="1920"
    height="1080"
    preload
    loading="eager"
    fetch-priority="high"
    format="webp"
    sizes="100vw"
    alt="Hero banner"
  />

  <!-- Image below-the-fold : lazy + placeholder -->
  <NuxtImg
    src="/product.jpg"
    width="800"
    height="600"
    loading="lazy"
    placeholder
    format="webp"
    sizes="sm:100vw md:50vw lg:400px"
    alt="Product image"
  />
</template>
```

**Configuration nuxt.config.ts optimale :**

```typescript
image: {
  provider: 'ipx',  // Génération au build pour SSG
  quality: 80,
  format: ['webp', 'avif'],

  // Breakpoints alignés TailwindCSS 4
  screens: {
    xs: 320, sm: 640, md: 768,
    lg: 1024, xl: 1280, '2xl': 1536
  },

  densities: [1, 2],  // Support Retina

  presets: {
    hero: {
      modifiers: { format: 'webp', quality: 90, fit: 'cover' }
    },
    thumbnail: {
      modifiers: { format: 'webp', quality: 75, width: 300, height: 200 }
    },
    articleCover: {
      modifiers: { format: 'webp', quality: 80, width: 800, height: 450 }
    }
  }
}
```

### LQIP et placeholders blur-up

Le prop `:placeholder` génère automatiquement une version minuscule floutée :

```vue
<template>
  <!-- Placeholder automatique 10x10 -->
  <NuxtImg src="/photo.jpg" placeholder width="800" height="600" />

  <!-- Placeholder personnalisé [width, height, quality, blur] -->
  <NuxtImg src="/photo.jpg" :placeholder="[50, 25, 75, 5]" width="800" height="600" />
</template>
```

**Pour BlurHash/ThumbHash (~30 bytes inline) :**

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@unlazy/nuxt'],
  unlazy: { ssr: true, placeholderSize: 32 }
})
```

### Composant VideoEmbed sans CLS

Pattern complet pour embeds vidéo avec placeholder cliquable :

```vue
<!-- components/VideoEmbed.vue -->
<script setup lang="ts">
interface Props {
  src: string
  aspectRatio?: '16/9' | '4/3' | '21/9' | '9/16'
  title?: string
}

const props = withDefaults(defineProps<Props>(), {
  aspectRatio: '16/9',
  title: 'Video'
})

const showIframe = ref(false)
const isLoaded = ref(false)

const aspectClass = computed(() => ({
  '16/9': 'aspect-video',
  '4/3': 'aspect-[4/3]',
  '21/9': 'aspect-[21/9]',
  '9/16': 'aspect-[9/16]'
})[props.aspectRatio])
</script>

<template>
  <div :class="[aspectClass, 'relative w-full bg-muted rounded-lg overflow-hidden']">
    <!-- Placeholder cliquable -->
    <button
      v-if="!showIframe"
      class="absolute inset-0 flex items-center justify-center group"
      @click="showIframe = true"
      :aria-label="`Charger ${title}`"
    >
      <div class="w-16 h-16 bg-red-600 rounded-full flex items-center justify-center group-hover:bg-red-700 transition-colors">
        <svg class="w-6 h-6 text-white ml-1" fill="currentColor" viewBox="0 0 24 24">
          <path d="M8 5v14l11-7z"/>
        </svg>
      </div>
    </button>

    <!-- Skeleton pendant chargement -->
    <div v-if="showIframe && !isLoaded" class="absolute inset-0 animate-pulse bg-muted" />

    <!-- Iframe -->
    <iframe
      v-if="showIframe"
      :src="src"
      :title="title"
      loading="lazy"
      class="absolute inset-0 w-full h-full"
      allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope"
      allowfullscreen
      @load="isLoaded = true"
    />
  </div>
</template>
```

### Skeleton screens et réservation d'espace

**Composant SkeletonBox réutilisable :**

```vue
<!-- components/SkeletonBox.vue -->
<template>
  <div
    :class="['bg-muted rounded animate-pulse', className]"
    :style="{ width, height, minHeight }"
  />
</template>

<script setup lang="ts">
defineProps<{
  width?: string
  height?: string
  minHeight?: string
  className?: string
}>()
</script>
```

**Pattern avec réservation d'espace exacte :**

```vue
<template>
  <!-- Container avec dimensions fixes = zéro CLS -->
  <div class="min-h-[320px]">
    <template v-if="pending">
      <SkeletonBox width="100%" height="200px" />
      <SkeletonBox width="70%" height="24px" class="mt-4" />
      <SkeletonBox width="90%" height="16px" class="mt-2" />
    </template>

    <template v-else>
      <NuxtImg :src="data.image" width="800" height="400" class="w-full h-auto" />
      <h2>{{ data.title }}</h2>
      <p>{{ data.description }}</p>
    </template>
  </div>
</template>
```

**Réservation pour embeds tiers (Twitter, Instagram) :**

```vue
<template>
  <!-- min-height estimé pour Twitter embed (~250px) -->
  <div class="max-w-lg min-h-[250px] w-full">
    <ClientOnly>
      <blockquote class="twitter-tweet" data-dnt="true">
        <a :href="tweetUrl"></a>
      </blockquote>
    </ClientOnly>
  </div>
</template>
```

### CSS contain pour isolation du layout

```css
/* Isoler les widgets tiers */
.embed-container {
  contain: layout paint;
  min-height: 250px;
}

/* Sections offscreen pour performance */
.below-fold-section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;
}
```

| Propriété | Effet | Cas d'usage |
|-----------|-------|-------------|
| `contain: layout` | Isole les recalculs de layout | Widgets tiers, embeds |
| `contain: paint` | Isole les repaint | Contenu dynamique |
| `content-visibility: auto` | Render à la demande | Sections longues below-fold |
| `contain-intrinsic-size` | Taille estimée pour réservation | Avec content-visibility |

### Pattern accordion sans CLS

Animation `grid-template-rows` au lieu de `height` (compositor-safe) :

```vue
<template>
  <div class="accordion">
    <button @click="isOpen = !isOpen">Toggle</button>
    <div class="content-wrapper" :class="{ expanded: isOpen }">
      <div class="content">
        {{ content }}
      </div>
    </div>
  </div>
</template>

<style>
.content-wrapper {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 0.3s ease;
}

.content-wrapper.expanded {
  grid-template-rows: 1fr;
}

.content {
  overflow: hidden;
}
</style>
```

### Debugging CLS en console

Script à coller dans DevTools pour identifier les sources de layout-shift :

```javascript
// Coller dans la console DevTools
let cls = 0;
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    if (!entry.hadRecentInput) {
      cls += entry.value;
      console.log('Layout Shift:', {
        value: entry.value.toFixed(4),
        cumulativeCLS: cls.toFixed(4),
        sources: entry.sources?.map(s => s.node?.tagName || 'unknown')
      });
    }
  }
}).observe({ type: 'layout-shift', buffered: true });
```

### Anti-patterns CLS critiques

| ❌ Anti-pattern | ✅ Correction |
|----------------|---------------|
| `<NuxtImg>` sans width/height | Toujours spécifier les dimensions |
| `loading="lazy"` sur image LCP | `loading="eager"` + `preload` |
| `font-display: block` | `font-display: swap` ou `optional` |
| Animation `width`/`height` | Utiliser `transform: scale()` |
| Pas de `min-height` sur embeds | Réserver l'espace avec CSS |
| Embeds tiers sans `<ClientOnly>` | Wrapper avec `<ClientOnly>` |
| `v-if` sans réservation espace | `min-height` sur container |

### Checklist CLS < 0.1

```markdown
## Images
- [ ] Toutes les `<NuxtImg>` ont `width` et `height`
- [ ] Images LCP avec `preload`, `loading="eager"`, `fetch-priority="high"`
- [ ] Format moderne (`format="webp"`)
- [ ] `placeholder` activé pour images below-fold
- [ ] Presets configurés pour tailles communes

## Fonts
- [ ] `font-display: optional` ou fallback métrique
- [ ] Preload de 1-2 fonts critiques maximum
- [ ] Subsets limités (`latin`, `latin-ext`)

## Contenu dynamique
- [ ] Skeletons avec dimensions exactes
- [ ] `min-height` sur containers avec v-if
- [ ] `<ClientOnly>` pour widgets tiers

## Embeds
- [ ] `aspect-ratio` ou `min-height` sur tous les embeds
- [ ] Placeholder cliquable avant chargement iframe
- [ ] `contain: layout paint` pour isolation

## Animations
- [ ] Uniquement `transform` et `opacity`
- [ ] Pattern grid-template-rows pour accordions
- [ ] Pas de `width`/`height` animés

## Validation
- [ ] Layout Shift Regions activé dans DevTools
- [ ] Script debugging CLS sans shifts > 0.01
- [ ] Lighthouse CLS < 0.1
```

---

## INP (Interaction to Next Paint)

### Debounce et Throttle avec VueUse

### Règle d'or

| Technique | Comportement | Cas d'usage |
|-----------|--------------|-------------|
| **Debounce** | Attend que l'utilisateur arrête | Recherche, validation, autocomplétion |
| **Throttle** | Max 1 exécution par intervalle | Scroll, resize, mousemove |

### Debounce pour inputs utilisateur

```typescript
import { useDebounceFn } from '@vueuse/core'

const searchQuery = ref('')
const results = ref([])

// Debounce : attend 300ms après la dernière frappe
// maxWait : force exécution après 1s max (même si l'utilisateur tape encore)
const debouncedSearch = useDebounceFn(async (query: string) => {
  if (query.length < 2) {
    results.value = []
    return
  }
  results.value = await fetchSearchResults(query)
}, 300, { maxWait: 1000 })

// Déclenché à chaque frappe
watch(searchQuery, (query) => {
  debouncedSearch(query)
})
```

**Options avancées :**

| Option | Effet | Valeur recommandée |
|--------|-------|-------------------|
| `delay` | Délai avant exécution | 300ms (search), 500ms (validation) |
| `maxWait` | Force exécution après délai max | 1000ms (évite attente infinie) |

### Throttle pour événements continus

```typescript
import { useThrottleFn, useEventListener } from '@vueuse/core'

// Throttle : max 1 exécution par 100ms
const throttledScrollHandler = useThrottleFn(() => {
  // Calculs de position, lazy loading, etc.
  updateScrollPosition()
}, 100)

// useEventListener avec passive: true pour performances scroll
useEventListener(window, 'scroll', throttledScrollHandler, { passive: true })
```

### Pattern combiné pour search avec UI feedback

```vue
<script setup lang="ts">
import { useDebounceFn, refDebounced } from '@vueuse/core'

const query = ref('')
const queryDebounced = refDebounced(query, 300)  // Pour affichage
const isSearching = ref(false)
const results = ref([])

// Debounce la fonction de recherche
const performSearch = useDebounceFn(async (q: string) => {
  if (q.length < 2) {
    results.value = []
    return
  }

  isSearching.value = true
  try {
    results.value = await fetchSearchResults(q)
  } finally {
    isSearching.value = false
  }
}, 300, { maxWait: 1000 })

watch(query, performSearch)
</script>

<template>
  <input v-model="query" placeholder="Rechercher..." />
  <p v-if="isSearching">Recherche en cours...</p>
  <p v-else-if="queryDebounced && !results.length">Aucun résultat</p>
  <ul v-else>
    <li v-for="result in results" :key="result.id">{{ result.title }}</li>
  </ul>
</template>
```

---

## Scheduling et Yield

### Le problème

Les tâches JavaScript longues (> 50ms) bloquent le main thread et dégradent l'INP. Les patterns "yield" et "scheduling" découpent les tâches pour permettre aux interactions utilisateur de s'intercaler.

### Support navigateurs (Décembre 2025)

| API | Chrome | Edge | Firefox | Safari |
|-----|--------|------|---------|--------|
| `requestIdleCallback` | 47+ | 79+ | 55+ | ❌ (flag) |
| `scheduler.postTask` | 94+ | 94+ | 101+ | ❌ |
| `scheduler.yield` | 129+ | 129+ | 142+ | ❌ |

### Composable useScheduler (avec fallbacks universels)

```typescript
// composables/useScheduler.ts
export function useScheduler() {
  const idleCallbackId = ref<number | null>(null)

  /**
   * Exécute une tâche pendant les idle periods du navigateur.
   * Idéal pour analytics, prefetch, warm cache.
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
   * Schedule avec priorité (Scheduler API).
   * - 'user-blocking': critique, exécution immédiate
   * - 'user-visible': important mais différable
   * - 'background': non-critique
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
   * Yield au main thread avec continuation prioritaire.
   * Utilise scheduler.yield() si disponible, fallback sur setTimeout.
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

### Pattern traitement par chunks avec yield

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

### Pattern event handler optimisé

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

### Composable useYieldingProcessor (avec annulation)

```typescript
// composables/useYieldingProcessor.ts
export function useYieldingProcessor<T>() {
  const isProcessing = ref(false)
  const progress = ref(0)
  const controller = ref<AbortController | null>(null)

  async function process(
    items: T[],
    processor: (item: T) => void,
    chunkSize: number = 5
  ): Promise<void> {
    controller.value = new AbortController()
    isProcessing.value = true
    progress.value = 0

    try {
      for (let i = 0; i < items.length; i++) {
        if (controller.value.signal.aborted) break

        processor(items[i])
        progress.value = Math.round((i / items.length) * 100)

        if (i % chunkSize === 0 && i > 0) {
          await yieldToMain()
        }
      }
    } finally {
      isProcessing.value = false
      progress.value = 100
    }
  }

  function cancel() {
    controller.value?.abort()
  }

  return { process, cancel, isProcessing, progress }
}

async function yieldToMain(): Promise<void> {
  if ((globalThis as any).scheduler?.yield) {
    return (globalThis as any).scheduler.yield()
  }
  return new Promise(resolve => setTimeout(resolve, 0))
}
```

---

## Event Listeners Optimisés

### Passive Listeners

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

### Cleanup automatique avec VueUse

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

### Pattern combiné scroll optimisé

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

## Optimisation des Listes

### Event Delegation pour listes

Au lieu d'attacher un handler à chaque élément de liste, utiliser un seul handler délégué :

```vue
<script setup lang="ts">
// Un seul handler délégué au lieu de N handlers
const handleListClick = (event: MouseEvent) => {
  const listItem = (event.target as HTMLElement).closest('[data-item-id]')
  if (!listItem) return

  const itemId = listItem.dataset.itemId
  if (itemId) {
    selectItem(parseInt(itemId))
  }
}
</script>

<template>
  <!-- ✅ BIEN : 1 seul listener pour toute la liste -->
  <ul @click="handleListClick">
    <li
      v-for="item in items"
      :key="item.id"
      :data-item-id="item.id"
      class="cursor-pointer hover:bg-muted"
    >
      {{ item.name }}
    </li>
  </ul>
</template>
```

**Avantages :**
- Réduit la mémoire (1 listener vs N listeners)
- Meilleur INP (moins de handlers à attacher/détacher)
- Fonctionne automatiquement avec les éléments ajoutés dynamiquement

| Nombre d'items | Handlers directs | Event delegation |
|----------------|------------------|------------------|
| 50 | 50 listeners | 1 listener |
| 500 | 500 listeners | 1 listener |
| 5000 | 5000 listeners | 1 listener |

### v-memo pour listes larges (1000+ items)

`v-memo` évite le re-render des items dont les dépendances n'ont pas changé :

```vue
<template>
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

**Fonctionnement :** Le template de l'item n'est re-rendu que si une valeur dans le tableau `v-memo` change.

| Scénario | Sans v-memo | Avec v-memo |
|----------|-------------|-------------|
| Sélection item dans liste 1000 | Re-render 1000 items | Re-render 2 items (ancien + nouveau sélectionné) |

**Quand utiliser :**
- Listes de 1000+ items avec sélection
- Composants item complexes (plusieurs enfants)
- Listes avec filtrage/tri côté client

**Quand éviter :**
- Listes < 100 items (overhead de v-memo > bénéfice)
- Items simples (texte uniquement)
- Virtualisation déjà en place (vue-virtual-scroller)

### Passive Listeners pour listes scrollables

Pour les listes avec scroll interne, toujours utiliser `.passive` :

```vue
<template>
  <!-- ✅ Passive scroll (ne bloque jamais le scroll) -->
  <div
    @scroll.passive="handleScroll"
    class="overflow-auto max-h-96"
  >
    <div v-for="item in items" :key="item.id">
      {{ item.name }}
    </div>
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

---

## Code Splitting et Chunks Vite

### Configuration manualChunks pour vendor splitting

Séparer les vendors en chunks distincts améliore le caching lors des mises à jour :

```typescript
// nuxt.config.ts
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
```

**Tailles cibles :** chunks de **100-250 KB** (30-80 KB gzippé), warning Vite à 500 KB.

| Chunk | Contenu | Mise à jour |
|-------|---------|-------------|
| `vendor-core` | Vue, Pinia | Rare (version majeure) |
| `vendor-ui` | Reka UI, shadcn | Occasionnel |
| `vendor-utils` | VueUse, clsx | Rare |
| `vendor` | Autres dépendances | Variable |

**Avantage caching :** Quand le code applicatif change, seuls les chunks concernés sont invalidés. Les vendors restent en cache.

### Configuration Rollup Tree Shaking

Configurer Rollup pour un tree shaking plus agressif tout en préservant les fichiers CSS :

```typescript
// nuxt.config.ts
vite: {
  build: {
    target: 'esnext',
    cssCodeSplit: true,
    rollupOptions: {
      treeshake: {
        preset: 'recommended',  // 'safest' | 'recommended' | 'smallest'
        moduleSideEffects: (id) => id.endsWith('.css'),  // Préserve les imports CSS
        propertyReadSideEffects: false,  // Élimination plus agressive
        annotations: true  // Respecte /* #__PURE__ */ et @__NO_SIDE_EFFECTS__
      },
      output: {
        minifyInternalExports: true,
        // ... manualChunks config
      }
    }
  },
  // Pre-bundle des dépendances pour accélérer le dev server
  optimizeDeps: {
    include: ['lodash-es', '@vueuse/core']
  }
}
```

**Presets Rollup treeshake :**

| Preset | Comportement | Cas d'usage |
|--------|--------------|-------------|
| `safest` | Conserve tous les side effects possibles | Dépendances legacy problématiques |
| `recommended` | Équilibre sécurité/taille (**défaut**) | ✅ **95% des projets** |
| `smallest` | Élimination agressive | Projets sans dépendances CJS |

**Options clés :**

| Option | Effet | Recommandation |
|--------|-------|----------------|
| `moduleSideEffects: (id) => id.endsWith('.css')` | Préserve les imports CSS purs | ✅ Obligatoire |
| `propertyReadSideEffects: false` | Élimine les lectures de propriétés inutilisées | ✅ Recommandé |
| `minifyInternalExports: true` | Réduit les noms d'exports internes | ✅ Recommandé |

**⚠️ Attention :** Si une dépendance casse avec `recommended`, tester d'abord avec `safest` avant de la signaler comme problématique.

### Prefetching stratégique NuxtLink

Contrôler le prefetching pour éviter la surcharge réseau :

```vue
<template>
  <!-- Default : prefetch à la visibilité (recommandé pour navigation principale) -->
  <NuxtLink to="/about">À propos</NuxtLink>

  <!-- Désactivé pour pages lourdes ou rarement visitées -->
  <NuxtLink to="/heavy-dashboard" :prefetch="false">Dashboard</NuxtLink>

  <!-- Prefetch au hover uniquement (économise bande passante) -->
  <NuxtLink to="/contact" prefetch-on="interaction">Contact</NuxtLink>
</template>
```

**Configuration globale dans nuxt.config.ts :**

```typescript
experimental: {
  defaults: {
    nuxtLink: {
      prefetchOn: {
        visibility: true,    // Prefetch quand le lien est visible
        interaction: false   // Pas de prefetch au hover (économie)
      }
    }
  }
}
```

| Stratégie | Comportement | Cas d'usage |
|-----------|--------------|-------------|
| `visibility: true` | Prefetch dès que visible | Navigation principale, liens fréquents |
| `interaction: true` | Prefetch au hover/focus | Liens secondaires |
| `prefetch: false` | Pas de prefetch | Pages lourdes, liens rares |

---

## Anti-patterns à éviter

### ❌ Debounce sans maxWait

```typescript
// ❌ Risque d'attente infinie si utilisateur tape constamment
const search = useDebounceFn(async (q) => {
  await fetchResults(q)
}, 300)

// ✅ maxWait garantit une exécution périodique
const search = useDebounceFn(async (q) => {
  await fetchResults(q)
}, 300, { maxWait: 1000 })
```

### ❌ Throttle trop agressif

```typescript
// ❌ 10ms = trop fréquent, inutile
const throttled = useThrottleFn(handler, 10)

// ✅ 100ms = bon équilibre performance/réactivité
const throttled = useThrottleFn(handler, 100)
```

### ❌ Tâches longues sans yield

```typescript
// ❌ Bloque le main thread pendant le traitement
function processAllItems(items: Item[]) {
  items.forEach(item => {
    heavyProcessing(item)
  })
}

// ✅ Yield périodique pour préserver l'INP
async function processAllItems(items: Item[]) {
  for (let i = 0; i < items.length; i++) {
    heavyProcessing(items[i])
    if (i % 5 === 0) await yieldToMain()
  }
}
```

### ❌ Event listeners sans passive

```typescript
// ❌ Bloque potentiellement le scroll
window.addEventListener('scroll', onScroll)

// ✅ Passive = ne bloque jamais le scroll
window.addEventListener('scroll', onScroll, { passive: true })
```

---

## Checklist Performance INP

```markdown
## Hydratation lazy
- [ ] Identifier composants below-fold → ajouter `hydrate-on-visible`
- [ ] Identifier composants interactifs non-critiques → ajouter `hydrate-on-interaction`
- [ ] Convertir composants statiques (footer, sidebar) en `.server.vue` ou `hydrate-never`
- [ ] Vérifier absence de props spreading (`v-bind="obj"`) sur composants lazy
- [ ] Composants critiques above-fold : PAS de prefix `Lazy`

## Inputs et formulaires
- [ ] Debounce sur recherche/autocomplétion (300ms + maxWait 1000ms)
- [ ] Throttle sur validation temps réel (500ms)

## Événements continus
- [ ] Throttle sur scroll/resize (100ms)
- [ ] Passive listeners pour scroll/touch/wheel (`.passive` modifier)

## Optimisation des listes
- [ ] Listes > 50 items : implémenter event delegation
- [ ] Listes > 1000 items : ajouter `v-memo`
- [ ] Listes scrollables : `.passive` sur scroll handler

## Tâches longues
- [ ] Utiliser `scheduleIdle()` pour analytics/tracking
- [ ] Yield toutes les ~50ms dans boucles longues
- [ ] Séparer UI critique du travail non-critique dans handlers
- [ ] AbortController pour annulation si nécessaire
- [ ] Indicateur de progression pour tâches > 1s

## Code Splitting
- [ ] Configurer `manualChunks` Vite pour vendor splitting
- [ ] NuxtLink : `prefetch-on="interaction"` pour liens secondaires
- [ ] NuxtLink : `:prefetch="false"` pour pages lourdes

## Cleanup
- [ ] useEventListener pour cleanup automatique
- [ ] AbortController pour fetch dans watchers
- [ ] onUnmounted pour cancelIdleCallback

## Mesure et validation
- [ ] Vérifier INP < 200ms dans DevTools Performance
- [ ] Tester avec CPU 4x throttling (simulation mobile)
- [ ] Analyser chunks avec `npx nuxi analyze`
- [ ] Plugin web-vitals avec attribution en développement
```
