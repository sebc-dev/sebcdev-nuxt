# CLS (Cumulative Layout Shift)

## Configuration NuxtImg pour CLS zéro

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

## LQIP et placeholders blur-up

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

## Composant VideoEmbed sans CLS

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

## Skeleton screens et réservation d'espace

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

## CSS contain pour isolation du layout

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

## Pattern accordion sans CLS

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

## Debugging CLS en console

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

## Anti-patterns CLS critiques

| ❌ Anti-pattern | ✅ Correction |
|----------------|---------------|
| `<NuxtImg>` sans width/height | Toujours spécifier les dimensions |
| `loading="lazy"` sur image LCP | `loading="eager"` + `preload` |
| `font-display: block` | `font-display: swap` ou `optional` |
| Animation `width`/`height` | Utiliser `transform: scale()` |
| Pas de `min-height` sur embeds | Réserver l'espace avec CSS |
| Embeds tiers sans `<ClientOnly>` | Wrapper avec `<ClientOnly>` |
| `v-if` sans réservation espace | `min-height` sur container |

## Checklist CLS < 0.1

```markdown
# Images
- [ ] Toutes les `<NuxtImg>` ont `width` et `height`
- [ ] Images LCP avec `preload`, `loading="eager"`, `fetch-priority="high"`
- [ ] Format moderne (`format="webp"`)
- [ ] `placeholder` activé pour images below-fold
- [ ] Presets configurés pour tailles communes

# Fonts
- [ ] `font-display: optional` ou fallback métrique
- [ ] Preload de 1-2 fonts critiques maximum
- [ ] Subsets limités (`latin`, `latin-ext`)

# Contenu dynamique
- [ ] Skeletons avec dimensions exactes
- [ ] `min-height` sur containers avec v-if
- [ ] `<ClientOnly>` pour widgets tiers

# Embeds
- [ ] `aspect-ratio` ou `min-height` sur tous les embeds
- [ ] Placeholder cliquable avant chargement iframe
- [ ] `contain: layout paint` pour isolation

# Animations
- [ ] Uniquement `transform` et `opacity`
- [ ] Pattern grid-template-rows pour accordions
- [ ] Pas de `width`/`height` animés

# Validation
- [ ] Layout Shift Regions activé dans DevTools
- [ ] Script debugging CLS sans shifts > 0.01
- [ ] Lighthouse CLS < 0.1
```

---
