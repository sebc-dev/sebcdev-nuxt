# Guide complet CLS et Core Web Vitals pour Nuxt 4.2.x SSG

**Un CLS inférieur à 0.1 est atteignable** avec votre stack Nuxt 4.2.x + TailwindCSS 4.1.x + Cloudflare Pages en appliquant systématiquement les techniques documentées ici. L'optimisation repose sur quatre piliers : dimensionnement explicite des images, réservation d'espace pour le contenu dynamique, gestion optimale des fonts, et animations compositor-safe. Ce guide fournit des configurations concrètes et des anti-patterns critiques à éviter pour atteindre un score CLS excellent en production SSG.

---

## Configuration NuxtImg pour un CLS zéro

La clé fondamentale est de **toujours spécifier `width` et `height`** sur chaque composant `<NuxtImg>`. Ces attributs HTML permettent au navigateur de calculer l'aspect-ratio et réserver l'espace avant même le chargement du CSS.

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

### Configuration nuxt.config.ts optimale pour SSG

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/image', '@nuxt/fonts'],
  
  // Critique : désactiver inline styles pour éviter CLS CSS
  features: {
    inlineStyles: false
  },

  image: {
    provider: 'ipx', // Génération au build pour SSG
    quality: 80,
    format: ['webp', 'avif'],
    
    // Breakpoints alignés TailwindCSS 4
    screens: {
      xs: 320, sm: 640, md: 768, 
      lg: 1024, xl: 1280, '2xl': 1536
    },
    
    densities: [1, 2], // Support Retina
    
    presets: {
      hero: {
        modifiers: { format: 'webp', quality: 90, fit: 'cover' }
      },
      thumbnail: {
        modifiers: { format: 'webp', quality: 75, width: 300, height: 200 }
      }
    }
  },

  nitro: {
    preset: 'cloudflare-pages-static',
    prerender: { crawlLinks: true }
  }
})
```

### LQIP et placeholders blur-up

Le prop `:placeholder` génère automatiquement une version minuscule floutée de l'image :

```vue
<template>
  <!-- Placeholder automatique 10x10 -->
  <NuxtImg src="/photo.jpg" placeholder width="800" height="600" />

  <!-- Placeholder personnalisé [width, height, quality, blur] -->
  <NuxtImg src="/photo.jpg" :placeholder="[50, 25, 75, 5]" width="800" height="600" />
</template>
```

Pour des performances optimales avec **BlurHash** ou **ThumbHash** (inline ~30 bytes), utilisez `@unlazy/nuxt` :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@unlazy/nuxt'],
  unlazy: { ssr: true, placeholderSize: 32 }
})
```

---

## Aspect-ratio CSS moderne avec TailwindCSS 4.1.x

En **décembre 2025**, `aspect-ratio` CSS bénéficie d'un support navigateur de **95.33%**. Aucun fallback padding-bottom n'est nécessaire. TailwindCSS 4 propose une syntaxe CSS-native élégante.

### Classes natives et configuration @theme

```css
/* assets/css/tailwind.css */
@import "tailwindcss";

@theme {
  /* Ratios personnalisés pour votre blog */
  --aspect-retro: 4 / 3;
  --aspect-cinema: 21 / 9;
  --aspect-instagram-feed: 4 / 5;
  --aspect-story: 9 / 16;
  
  /* Fonts et couleurs oklch */
  --font-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;
  --color-primary: oklch(0.6 0.15 250);
}
```

```html
<!-- Utilisation directe des classes -->
<img class="aspect-video w-full object-cover" src="/cover.jpg" />
<div class="aspect-square bg-gray-100">Carré</div>
<iframe class="aspect-[21/9] w-full" src="https://youtube.com/embed/..."></iframe>

<!-- Ratios custom depuis @theme -->
<img class="aspect-retro w-full" src="/photo.jpg" />
<div class="aspect-story max-w-sm mx-auto">Story format</div>
```

### Composant VideoEmbed sans CLS

Ce composant réserve l'espace avant le chargement de l'iframe et propose un placeholder interactif :

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
  <div :class="[aspectClass, 'relative w-full bg-gray-200 dark:bg-gray-800 rounded-lg overflow-hidden']">
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
    <div v-if="showIframe && !isLoaded" class="absolute inset-0 animate-pulse bg-gray-300" />

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

---

## Optimisation typographie et fonts

Les fonts mal configurées sont une **cause majeure de CLS invisible** dans Lighthouse mais visible par les utilisateurs (FOUT - Flash of Unstyled Text).

### Stratégie font-display recommandée

| Valeur | Usage | Impact CLS |
|--------|-------|------------|
| `swap` | Headings/branding | ⚠️ Peut causer CLS |
| `optional` | Body text | ✅ **Zéro CLS garanti** |
| `fallback` | Compromis | Swap limité à ~3s |

### Configuration @nuxt/fonts + TailwindCSS 4

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/fonts'],
  
  fonts: {
    families: [
      { 
        name: 'Inter', 
        provider: 'google',
        weights: ['100..900'] // Variable font
      }
    ],
    defaults: {
      weights: [400, 500, 600, 700],
      subsets: ['latin']
    }
  },
  
  app: {
    head: {
      link: [
        {
          rel: 'preload',
          href: '/_fonts/inter-latin-400-normal.woff2',
          as: 'font',
          type: 'font/woff2',
          crossorigin: 'anonymous'
        }
      ]
    }
  }
})
```

### @font-face avec fallback métrique (CLS = 0)

Pour éliminer complètement le CLS des fonts, utilisez des **fallbacks métriquement ajustés** :

```css
/* assets/css/fonts.css */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter-var.woff2') format('woff2');
  font-weight: 100 900;
  font-display: swap;
}

/* Fallback métrique généré par Capsize/Fontaine */
@font-face {
  font-family: 'Inter Fallback';
  src: local('Arial');
  ascent-override: 90.49%;
  descent-override: 22.56%;
  line-gap-override: 0%;
  size-adjust: 107.64%;
}

@theme {
  --font-sans: 'Inter', 'Inter Fallback', ui-sans-serif, system-ui, sans-serif;
}
```

### Subsetting fonts avec Glyphhanger

Réduisez **96% du poids** en ne conservant que les caractères Latin :

```bash
# Installation
npm install -g glyphhanger
pip install fonttools brotli

# Subset Latin uniquement
glyphhanger --LATIN --subset=Inter.ttf --formats=woff2

# Résultat : Inter 765KB → ~15KB
```

---

## Skeleton screens et réservation d'espace

### Composant Skeleton réutilisable

```vue
<!-- components/SkeletonBox.vue -->
<template>
  <div 
    :class="['bg-gray-200 dark:bg-gray-700 rounded animate-pulse', className]"
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

### Pattern avec réservation d'espace exacte

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

### Réservation pour embeds tiers (Twitter, Instagram)

```vue
<template>
  <!-- min-height estimé pour Twitter embed -->
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

---

## Animations et transitions sans CLS

### Propriétés safe vs dangereuses

| ✅ **Safe (Compositor)** | ❌ **Éviter (Layout trigger)** |
|--------------------------|-------------------------------|
| `transform: translate()` | `top`, `left`, `margin` |
| `transform: scale()` | `width`, `height` |
| `opacity` | `padding`, `border-width` |
| `filter` | `font-size` |

### Pattern accordion sans CLS

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

---

## Configuration Cloudflare Pages

### Early Hints (103) automatiques

Cloudflare convertit automatiquement vos `<link rel="preload">` en Early Hints. Configurez également via `_headers` :

```
# public/_headers

/*
  Link: </_fonts/inter-latin-400-normal.woff2>; rel=preload; as=font; type=font/woff2; crossorigin

/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

/fonts/*
  Cache-Control: public, max-age=31536000, immutable
  Access-Control-Allow-Origin: *

/*
  Cache-Control: public, max-age=0, must-revalidate
```

---

## Mesure et debugging CLS

### Outils essentiels

- **PageSpeed Insights** : données Lab + Field (CrUX)
- **Chrome DevTools** : Performance panel → piste "Layout Shifts"
- **Layout Shift Regions** : DevTools → Rendering → cocher l'option

### Script de debugging console

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

### Monitoring RUM avec web-vitals

```typescript
// plugins/web-vitals.client.ts
import { onCLS, onINP, onLCP } from 'web-vitals/attribution'

onCLS((metric) => {
  console.log('CLS:', metric.value, metric.attribution?.largestShiftTarget)
  // Envoyer à votre analytics
})
```

---

## Anti-patterns critiques à éviter

| ❌ Anti-pattern | ✅ Correction |
|----------------|---------------|
| `<NuxtImg>` sans width/height | Toujours spécifier les dimensions |
| `loading="lazy"` sur image LCP | `loading="eager"` + `preload` |
| `font-display: block` | `font-display: swap` ou `optional` |
| Animation `width`/`height` | Utiliser `transform: scale()` |
| Pas de `min-height` sur embeds | Réserver l'espace avec CSS |
| `inlineStyles: true` (défaut) | `inlineStyles: false` |

---

## Checklist finale CLS < 0.1

- [ ] Toutes les `<NuxtImg>` ont `width` et `height`
- [ ] Images LCP avec `preload`, `loading="eager"`, `fetch-priority="high"`
- [ ] Format moderne (`format="webp"`)
- [ ] `placeholder` activé pour images below-fold
- [ ] `inlineStyles: false` dans nuxt.config.ts
- [ ] Fonts avec `font-display: optional` ou fallback métrique
- [ ] Preload de 1-2 fonts critiques maximum
- [ ] Embeds avec `aspect-ratio` ou `min-height`
- [ ] Animations uniquement sur `transform` et `opacity`
- [ ] Headers cache Cloudflare configurés
- [ ] Test Lighthouse et Layout Shift Regions validés

---

## Conclusion

L'atteinte d'un score CLS excellent repose sur une discipline systématique : **dimensionner explicitement tous les éléments visuels**, utiliser les placeholders LQIP pour l'UX, et confiner les éléments dynamiques avec `contain: layout`. La combinaison `@nuxt/fonts` + fallbacks métriques élimine le CLS typographique, tandis que TailwindCSS 4 simplifie la gestion des aspect-ratios. Cloudflare Pages amplifie ces optimisations avec Early Hints automatiques. En suivant cette approche, un CLS de **0.05 ou moins** est réaliste pour un blog SSG bien optimisé.