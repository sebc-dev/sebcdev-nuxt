# Optimisation Core Web Vitals pour Nuxt 4 SSG en 2025

L'optimisation des Core Web Vitals pour un blog statique Nuxt 4.2.x repose sur trois piliers fondamentaux : le préchargement intelligent des ressources critiques pour le LCP, la gestion rigoureuse de la réactivité Vue 3 pour l'INP, et la réservation systématique d'espace pour éviter le CLS. Avec le stack technique ciblé (TailwindCSS 4.1.x, Cloudflare Pages, Nuxt Content 3.10+), atteindre **LCP ≤ 2.5s**, **INP ≤ 200ms** et **CLS ≤ 0.1** est réalisable en appliquant les configurations et patterns documentés ci-dessous. L'INP ayant officiellement remplacé le FID comme métrique Core Web Vital le **12 mars 2024**, les optimisations de réactivité Vue 3 deviennent particulièrement critiques.

---

## Maîtriser le LCP avec Nuxt 4 et TailwindCSS 4

Le Largest Contentful Paint mesure le temps nécessaire pour afficher le plus grand élément visible dans le viewport. Pour un blog, c'est généralement l'**image hero** ou le **titre principal**. La stratégie optimale combine préchargement agressif des ressources critiques et configuration méticuleuse de NuxtImage.

### Configuration NuxtImage pour images LCP

Le module `@nuxt/image` doit être configuré avec les formats modernes AVIF/WebP et des presets réutilisables :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/image', '@nuxt/fonts', 'nuxt-vitalizer'],
  
  image: {
    quality: 80,
    format: ['avif', 'webp'],
    screens: {
      xs: 320, sm: 640, md: 768, lg: 1024, xl: 1280, xxl: 1536
    },
    presets: {
      hero: { modifiers: { format: 'webp', fit: 'cover', quality: 85 } },
      thumbnail: { modifiers: { format: 'webp', fit: 'cover', quality: 70, width: 300, height: 200 } }
    }
  }
})
```

Pour l'image LCP elle-même, trois attributs sont **obligatoires** : `preload`, `loading="eager"` et `fetchpriority="high"` :

```vue
<template>
  <NuxtImg
    src="/images/hero-banner.jpg"
    alt="Hero banner"
    format="webp"
    width="1200"
    height="600"
    sizes="100vw sm:100vw md:100vw lg:1200px"
    preload
    loading="eager"
    fetchpriority="high"
    class="w-full h-auto"
  />
</template>
```

### Gestion du CSS critique avec TailwindCSS 4.1.x

TailwindCSS 4 utilise désormais une configuration CSS-native avec le plugin Vite `@tailwindcss/vite`. Un problème connu est que le fichier `entry.*.css` peut bloquer le rendu. Le module `nuxt-vitalizer` offre un workaround efficace :

```typescript
// nuxt.config.ts
import tailwindcss from "@tailwindcss/vite"

export default defineNuxtConfig({
  css: ['~/assets/css/main.css'],
  
  vite: {
    plugins: [tailwindcss()],
  },
  
  vitalizer: {
    disablePrefetchLinks: 'dynamicImports',
    disableStylesheets: 'entry',  // Supprime le CSS render-blocking
  },
  
  features: {
    inlineStyles: true,  // Requis pour disableStylesheets
  },
})
```

La configuration `@theme` de TailwindCSS 4 avec oklch s'intègre ainsi :

```css
/* assets/css/main.css */
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.6 0.2 250);
  --color-secondary: oklch(0.7 0.15 200);
  --font-sans: 'Inter', system-ui, sans-serif;
}

@layer base {
  html { font-family: var(--font-sans); }
}
```

### Préchargement des fonts avec @nuxt/fonts

Le module `@nuxt/fonts` gère automatiquement le self-hosting et génère les fallback metrics pour réduire le CLS :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/fonts'],
  fonts: {
    families: [
      { name: 'Inter', provider: 'google' },
    ],
  },
})
```

Pour un preload manuel avec `useHead()`, **utilisez l'import Vite** pour résoudre le chemin avec hash :

```vue
<script setup lang="ts">
import fontUrl from '~/assets/fonts/Inter-Regular.woff2?url'

useHead({
  link: [{
    rel: 'preload',
    href: fontUrl,
    as: 'font',
    type: 'font/woff2',
    crossorigin: 'anonymous'
  }]
})
</script>
```

---

## Optimiser l'INP avec Vue 3 Composition API

L'INP mesure la latence de toutes les interactions utilisateur, composée de trois phases : **Input Delay** (temps avant traitement), **Processing Duration** (exécution des handlers), et **Presentation Delay** (temps jusqu'au rendu). L'objectif est de maintenir chaque interaction sous **200ms**.

### Réactivité optimisée avec shallowRef et markRaw

Pour les grandes structures de données, `shallowRef` évite l'overhead de la réactivité profonde :

```vue
<script setup lang="ts">
import { shallowRef, markRaw, triggerRef } from 'vue'
import Chart from 'chart.js'

// ✅ shallowRef pour listes > 1000 items
const optimizedData = shallowRef({
  items: Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    details: { price: i * 10, active: true }
  }))
})

// ✅ markRaw pour instances de librairies externes
const chartInstance = ref<Chart | null>(null)

onMounted(() => {
  chartInstance.value = markRaw(new Chart(canvas, config))
})

// Pour déclencher une mise à jour avec shallowRef
function updateItem(index: number, newPrice: number) {
  optimizedData.value = {
    ...optimizedData.value,
    items: optimizedData.value.items.map((item, i) => 
      i === index ? { ...item, details: { ...item.details, price: newPrice } } : item
    )
  }
}
</script>
```

### Éviter les re-renders avec v-memo et v-once

Le directive `v-memo` est une micro-optimisation pour les listes de plus de **1000 items** :

```vue
<template>
  <!-- v-memo : seuls les items dont le statut "selected" change sont re-rendus -->
  <div 
    v-for="item in items" 
    :key="item.id"
    v-memo="[item.id === selectedId]"
  >
    <p>ID: {{ item.id }} - selected: {{ item.id === selectedId }}</p>
    <ExpensiveChildComponent :data="item" />
  </div>
  
  <!-- v-once : rendu une seule fois, jamais mis à jour -->
  <header v-once>
    <h1>{{ siteTitle }}</h1>
  </header>
</template>
```

### Debouncing et throttling avec VueUse

Pour les recherches en temps réel et les handlers de scroll, VueUse offre des primitives optimisées :

```vue
<script setup lang="ts">
import { useDebounceFn, useThrottleFn, useEventListener } from '@vueuse/core'

// Debounce : attend que l'utilisateur arrête de taper
const debouncedSearch = useDebounceFn(async (query: string) => {
  results.value = await fetchSearchResults(query)
}, 300, { maxWait: 1000 }) // Force exécution après 1s max

// Throttle : max 1 exécution par 100ms (scroll, resize)
const throttledScrollHandler = useThrottleFn(() => {
  updateScrollPosition()
}, 100)

useEventListener(window, 'scroll', throttledScrollHandler, { passive: true })
</script>
```

**Règle d'or** : Utilisez `debounce` pour les inputs utilisateur (recherche, validation), et `throttle` pour les événements continus (scroll, resize, mousemove).

### Yield to main thread pour tâches longues

Pour les traitements lourds, le pattern de yield permet aux interactions utilisateur de s'intercaler :

```typescript
async function processLongTask(items: any[]) {
  for (let i = 0; i < items.length; i++) {
    processItem(items[i])
    
    if (i % 5 === 0) {
      // scheduler.yield() avec fallback
      if ('scheduler' in window && 'yield' in (window as any).scheduler) {
        await (window as any).scheduler.yield()
      } else {
        await new Promise(resolve => setTimeout(resolve, 0))
      }
    }
  }
}
```

### Event handlers optimisés

Évitez les fonctions inline dans les templates et préférez les **passive event listeners** :

```vue
<template>
  <!-- ❌ Anti-pattern : nouvelle fonction créée à chaque render -->
  <button @click="() => handleClick(item.id)">Click</button>
  
  <!-- ✅ Event delegation avec data attributes -->
  <ul @click="handleItemClick">
    <li v-for="item in items" :key="item.id" :data-id="item.id">
      {{ item.name }}
    </li>
  </ul>
  
  <!-- ✅ Passive listeners pour scroll/touch -->
  <div @scroll.passive="onScroll" @touchmove.passive="onTouchMove">
    <!-- Contenu scrollable -->
  </div>
</template>

<script setup lang="ts">
function handleItemClick(event: Event) {
  const target = event.target as HTMLElement
  const listItem = target.closest('li')
  if (listItem) {
    const id = listItem.dataset.id
    // Traitement
  }
}
</script>
```

---

## Éliminer le CLS avec réservation d'espace systématique

Le Cumulative Layout Shift mesure l'instabilité visuelle de la page. Les trois sources principales sont les **images sans dimensions** (~60%), les **fonts web** (~25%), et le **contenu dynamique** (~15%).

### Images avec aspect-ratio TailwindCSS 4

TailwindCSS 4.1.x inclut nativement les utilitaires `aspect-ratio` (plus besoin de plugin) :

```vue
<template>
  <!-- Ratios prédéfinis -->
  <img class="aspect-video w-full" src="..." />       <!-- 16:9 -->
  <img class="aspect-square w-full" src="..." />     <!-- 1:1 -->
  <img class="aspect-[4/3] w-full" src="..." />      <!-- Custom -->
  
  <!-- Conteneur pour images lazy-loaded -->
  <div class="relative aspect-video overflow-hidden">
    <NuxtImg
      src="/images/article.jpg"
      width="800"
      height="450"
      loading="lazy"
      class="absolute inset-0 w-full h-full object-cover"
    />
  </div>
</template>
```

**Règle critique** : Tous les `<img>` doivent avoir les attributs `width` et `height`, même avec des dimensions responsive en CSS.

### Stratégie fonts optimale pour CLS zéro

Pour un CLS de **0**, la meilleure option est `font-display: optional` (la font n'est utilisée que si disponible à temps). Pour un compromis avec font visible, utilisez `font-display: swap` **avec size-adjust** :

```css
/* Font principale */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter.woff2') format('woff2');
  font-display: swap;
}

/* Fallback ajusté pour correspondre aux métriques d'Inter */
@font-face {
  font-family: 'Inter Fallback';
  src: local('Arial');
  size-adjust: 107.40%;
  ascent-override: 90.20%;
  descent-override: 22.48%;
  line-gap-override: 0%;
}

body {
  font-family: 'Inter', 'Inter Fallback', sans-serif;
}
```

Le module `@nuxtjs/fontaine` automatise ce calcul. Les outils [Fallback Font Generator](https://screenspan.net/fallback) et [Capsize](https://github.com/seek-oss/capsize) permettent de calculer ces métriques manuellement.

### ClientOnly avec fallback obligatoire

Les composants `<ClientOnly>` causent du CLS s'ils n'ont pas de placeholder côté serveur :

```vue
<template>
  <!-- ❌ CLS garanti -->
  <ClientOnly>
    <DynamicWidget />
  </ClientOnly>
  
  <!-- ✅ Placeholder avec dimensions identiques -->
  <ClientOnly>
    <DynamicWidget />
    <template #fallback>
      <div class="aspect-video bg-gray-100 animate-pulse"></div>
    </template>
  </ClientOnly>
</template>
```

### Animations sans CLS

Seules les propriétés `transform` et `opacity` sont **safe** pour les animations (pas de reflow) :

```css
/* ❌ CAUSE CLS */
.bad-animation {
  transition: width 0.3s, height 0.3s, margin 0.3s;
}

/* ✅ SAFE - GPU-accelerated */
.good-animation {
  transition: transform 0.3s, opacity 0.3s;
}

.button:hover {
  transform: scale(1.05);
}
```

---

## Configuration Cloudflare Pages pour performance maximale

### Fichier _headers optimisé

Placez ce fichier dans `/public/_headers` pour Nuxt 4 :

```plaintext
# Headers globaux - Sécurité
/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Referrer-Policy: strict-origin-when-cross-origin

# HTML statiques - Cache court avec revalidation
/*.html
  Cache-Control: public, max-age=0, s-maxage=3600, stale-while-revalidate=86400

/
  Cache-Control: public, max-age=0, s-maxage=3600, stale-while-revalidate=86400

# Assets JS/CSS hashés - Cache agressif immutable
/_nuxt/*.js
  Cache-Control: public, max-age=31536000, immutable

/_nuxt/*.css
  Cache-Control: public, max-age=31536000, immutable

# Images statiques
/_nuxt/*.webp
  Cache-Control: public, max-age=31536000, immutable

/_nuxt/*.avif
  Cache-Control: public, max-age=31536000, immutable

# Fonts - CORS requis
/fonts/*
  Cache-Control: public, max-age=31536000, immutable
  Access-Control-Allow-Origin: *

# Early Hints (103) - Preload ressources critiques
/
  Link: </_nuxt/entry.css>; rel=preload; as=style
  Link: </fonts/inter.woff2>; rel=preload; as=font; type=font/woff2; crossorigin

# Bloquer indexation preview deployments
https://:project.pages.dev/*
  X-Robots-Tag: noindex
```

### Configuration Nitro pour SSG Cloudflare

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare-pages-static',
    compressPublicAssets: {
      brotli: true,
      gzip: true,
    },
    prerender: {
      crawlLinks: true,
      routes: ['/'],
      failOnError: false,
    },
  },
  
  vite: {
    build: {
      rollupOptions: {
        output: {
          assetFileNames: '_nuxt/[name].[hash][extname]',
          chunkFileNames: '_nuxt/[name].[hash].js',
          entryFileNames: '_nuxt/[name].[hash].js',
        },
      },
    },
  },
})
```

### Paramètres Cloudflare Dashboard

| Paramètre | État | Raison |
|-----------|------|--------|
| HTTP/3 (QUIC) | ✅ ON | Connexions plus rapides |
| Early Hints | ✅ ON | Réduit LCP jusqu'à 30% |
| Brotli | ✅ ON | Meilleure compression |
| **Rocket Loader™** | ❌ OFF | **Interfère avec l'hydration Vue** |
| **Auto Minify** | ❌ OFF | Nuxt minifie déjà en build |
| **Mirage** | ❌ OFF | Conflits avec lazy loading Nuxt |

---

## Monitoring et CI avec Lighthouse CI

### Configuration lighthouserc.js pour Core Web Vitals

```javascript
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      staticDistDir: './.output/public',
      numberOfRuns: 3,
      url: ['http://localhost/', 'http://localhost/blog/'],
      settings: {
        chromeFlags: '--no-sandbox --headless',
        preset: 'desktop',
      }
    },
    assert: {
      assertions: {
        'largest-contentful-paint': ['error', { maxNumericValue: 2500 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        'total-blocking-time': ['error', { maxNumericValue: 200 }],
        'categories:performance': ['error', { minScore: 0.9 }],
      }
    },
    upload: {
      target: 'temporary-public-storage'
    }
  }
};
```

### GitHub Actions workflow

```yaml
# .github/workflows/lighthouse-ci.yml
name: Lighthouse CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      
      - run: npm ci
      - run: npm run generate
      
      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v12
        with:
          configPath: './lighthouserc.js'
          uploadArtifacts: true
          temporaryPublicStorage: true
          runs: 3
```

### Web-vitals pour debugging et RUM

```typescript
// plugins/web-vitals-debug.client.ts
import { onCLS, onINP, onLCP } from 'web-vitals/attribution'

export default defineNuxtPlugin(() => {
  if (process.dev) {
    onLCP((metric) => {
      console.group('🎯 LCP Attribution')
      console.log('Value:', metric.value, 'ms')
      console.log('Element:', metric.attribution.element)
      console.log('Resource Load Delay:', metric.attribution.resourceLoadDelay, 'ms')
      console.groupEnd()
    })

    onINP((metric) => {
      console.group('⚡ INP Attribution')
      console.log('Value:', metric.value, 'ms')
      console.log('Target:', metric.attribution.interactionTarget)
      console.log('Input Delay:', metric.attribution.inputDelay, 'ms')
      console.log('Processing Duration:', metric.attribution.processingDuration, 'ms')
      console.groupEnd()
    })

    onCLS((metric) => {
      console.group('📐 CLS Attribution')
      console.log('Value:', metric.value)
      console.log('Largest Shift Target:', metric.attribution.largestShiftTarget)
      console.groupEnd()
    })
  }
})
```

**Stack RUM gratuite recommandée** : Cloudflare Web Analytics (si hébergé sur Cloudflare) + Google Search Console + CrUX API pour les données field.

---

## Configuration nuxt.config.ts complète

Voici la configuration consolidée intégrant toutes les optimisations :

```typescript
// nuxt.config.ts
import tailwindcss from "@tailwindcss/vite"

export default defineNuxtConfig({
  compatibilityDate: '2024-12-30',
  ssr: true,

  modules: [
    '@nuxt/image',
    '@nuxt/fonts',
    '@nuxt/content',
    'nuxt-vitalizer',
  ],

  app: {
    head: {
      htmlAttrs: { lang: 'fr' },
      meta: [
        { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      ],
    },
  },

  css: ['~/assets/css/main.css'],

  vite: {
    plugins: [tailwindcss()],
    build: {
      cssCodeSplit: true,
      rollupOptions: {
        output: {
          assetFileNames: '_nuxt/[name].[hash][extname]',
          chunkFileNames: '_nuxt/[name].[hash].js',
          entryFileNames: '_nuxt/[name].[hash].js',
        }
      }
    },
  },

  image: {
    quality: 80,
    format: ['avif', 'webp'],
    screens: { xs: 320, sm: 640, md: 768, lg: 1024, xl: 1280, xxl: 1536 },
    presets: {
      hero: { modifiers: { format: 'webp', fit: 'cover', quality: 85 } },
    }
  },

  fonts: {
    families: [{ name: 'Inter', provider: 'google' }],
  },

  vitalizer: {
    disablePrefetchLinks: 'dynamicImports',
    disableStylesheets: 'entry',
  },

  features: {
    inlineStyles: true,
  },

  experimental: {
    payloadExtraction: true,
    defaults: {
      nuxtLink: { prefetchOn: 'interaction' },
    },
  },

  nitro: {
    preset: 'cloudflare-pages-static',
    compressPublicAssets: { brotli: true, gzip: true },
    prerender: { crawlLinks: true, routes: ['/'] },
  },

  routeRules: {
    '/_nuxt/**': { 
      headers: { 'Cache-Control': 'public, max-age=31536000, immutable' }
    },
  },
})
```

---

## Anti-patterns critiques à éviter

Les erreurs les plus courantes qui dégradent les Core Web Vitals incluent l'utilisation de `ref()` sur des structures de données massives au lieu de `shallowRef()`, l'oubli des attributs `width` et `height` sur les images, l'activation de Rocket Loader sur Cloudflare avec Nuxt, et l'utilisation de `font-display: swap` sans configuration de fallback metrics. 

Pour les images LCP, ne jamais utiliser `loading="lazy"` ou omettre `fetchpriority="high"`. Pour les handlers d'événements, éviter les fonctions inline dans les templates Vue qui créent de nouvelles instances à chaque render. Pour le CSS, ne pas désactiver `cssCodeSplit` de Vite qui créerait un bundle CSS monolithique.

## Conclusion

L'optimisation des Core Web Vitals pour Nuxt 4 SSG avec TailwindCSS 4 et Cloudflare Pages repose sur une approche systématique : **identifier et précharger l'élément LCP** (image hero avec `preload`, `eager`, `fetchpriority="high"`), **réduire la complexité des handlers Vue** (shallowRef, debounce, event delegation), et **réserver l'espace de chaque élément dynamique** (aspect-ratio, dimensions explicites, font metrics override). La configuration Cloudflare Pages avec Early Hints activé et Rocket Loader désactivé complète cette stack performante. Le monitoring via Lighthouse CI en CI/CD et web-vitals en RUM permet de détecter les régressions avant qu'elles n'impactent les utilisateurs réels.