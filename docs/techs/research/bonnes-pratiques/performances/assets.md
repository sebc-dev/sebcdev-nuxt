# Optimisation Nuxt 4.2.x SSG sur Cloudflare Pages : guide complet

Un blog Nuxt 4.2.x en mode SSG peut atteindre d'excellents scores Core Web Vitals avec une configuration optimale combinant tree shaking Vite/Rollup, TailwindCSS v4 JIT, compression Nitro et caching Cloudflare edge. Ce guide détaille chaque aspect technique pour un déploiement zero-cost performant.

La stack Nuxt 4 avec la nouvelle structure `app/` apporte des améliorations significatives : tree shaking natif des composables, intégration Vite 6, et support expérimental de Rolldown. TailwindCSS v4 représente une refonte architecturale majeure avec configuration CSS-native et détection automatique du contenu. Cloudflare Pages offre un hébergement edge gratuit avec Brotli natif niveau 4 et Early Hints automatiques.

---

## Tree shaking Vite/Rollup : configuration optimale pour SSG

Nuxt 4 intègre un tree shaking natif des composables via l'option `optimization.treeShake`, permettant d'éliminer automatiquement le code serveur du bundle client et inversement. Cette fonctionnalité, combinée aux presets Rollup, génère des bundles significativement plus légers.

### Configuration complète nuxt.config.ts

```typescript
export default defineNuxtConfig({
  compatibilityDate: '2025-01-01',
  srcDir: 'app',
  ssr: true,

  // Tree shaking natif Nuxt 4 pour composables
  optimization: {
    treeShake: {
      composables: {
        client: {
          vue: ['onServerPrefetch', 'onRenderTracked', 'onRenderTriggered'],
          '#app': ['definePayloadReducer', 'onPrehydrate']
        },
        server: {
          vue: ['onMounted', 'onUpdated', 'onUnmounted', 'onBeforeMount',
                'onBeforeUpdate', 'onBeforeUnmount', 'onActivated', 'onDeactivated'],
          '#app': ['definePayloadReviver']
        }
      }
    }
  },

  vite: {
    build: {
      target: 'esnext',
      cssCodeSplit: true,
      chunkSizeWarningLimit: 200,
      
      rollupOptions: {
        treeshake: {
          preset: 'recommended', // 'safest' | 'recommended' | 'smallest'
          moduleSideEffects: (id) => id.endsWith('.css'),
          propertyReadSideEffects: false,
          annotations: true
        },
        output: {
          minifyInternalExports: true,
          manualChunks: (id) => {
            if (id.includes('node_modules')) {
              if (id.includes('vue') || id.includes('@vue')) return 'vue-vendor'
              if (id.includes('@nuxt/content')) return 'content'
            }
          }
        }
      }
    },
    optimizeDeps: {
      include: ['lodash-es', '@vueuse/core']
    }
  }
})
```

Les trois presets Rollup offrent différents niveaux d'agressivité : **safest** préserve tous les side effects possibles, **recommended** (défaut) équilibre sécurité et taille, **smallest** élimine agressivement le code mort mais peut casser certaines dépendances. Pour un blog SSG, le preset `recommended` convient dans **95% des cas**.

### Bonnes pratiques imports ESM

Les imports nommés sont essentiels pour un tree shaking efficace. Évitez absolument les imports namespace (`import * as`) et les bibliothèques CommonJS :

```typescript
// ✅ Tree-shakeable
import { ref, computed } from 'vue'
import { debounce } from 'lodash-es'
import { Home, Settings } from 'lucide-vue-next'

// ❌ Anti-patterns bloquant le tree shaking
import * as Vue from 'vue'           // Namespace import
import lodash from 'lodash'          // CJS
import { icons } from 'lucide-vue-next'  // Objet entier
```

Pour les composants Vue, utilisez le prefix `Lazy` pour le code splitting automatique et les directives de lazy hydration (`hydrate-on-visible`, `hydrate-on-idle`) disponibles depuis Nuxt 3.16+.

---

## TailwindCSS v4 JIT : purging CSS automatique

TailwindCSS v4 représente une **refonte architecturale complète** : il n'y a plus de PurgeCSS car le JIT génère uniquement les classes utilisées. La configuration passe du JavaScript au **CSS natif** via la directive `@theme`.

### Différences majeures v3 → v4

| Aspect | TailwindCSS v3 | TailwindCSS v4 |
|--------|---------------|----------------|
| Config | `tailwind.config.js` | CSS natif `@theme {}` |
| JIT | Opt-in `mode: 'jit'` | Défaut et unique mode |
| Purging | PurgeCSS post-génération | Génération on-demand |
| Content | `content: []` obligatoire | Détection automatique |
| Import | `@tailwind base/utilities` | `@import "tailwindcss"` |
| Safelist | `safelist: []` | `@source inline()` |

### Configuration pour Nuxt 4 avec @tailwindcss/vite

```typescript
// nuxt.config.ts
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  css: ['~/assets/css/main.css'],
  vite: {
    plugins: [tailwindcss()]
  }
})
```

```css
/* assets/css/main.css */
@import "tailwindcss";

/* Sources additionnelles pour Nuxt Content */
@source "../content";

/* Thème personnalisé */
@theme {
  --font-sans: "Inter", system-ui, sans-serif;
  --color-primary-500: oklch(0.55 0.18 250);
  --color-primary-600: oklch(0.48 0.18 250);
}

/* Safelist pour contenu dynamique CMS */
@source inline("{sm:,md:,lg:,}grid-cols-{1,2,3,4}");
@source inline("{hover:,}text-primary-{500,600,700}");

/* Utilitaire personnalisé */
@utility prose-blog {
  max-width: 65ch;
  line-height: 1.75;
}
```

La détection automatique scanne tous les fichiers du projet (`app/components/`, `app/pages/`, etc.) en ignorant `node_modules/` et `.gitignore`. Pour le contenu Markdown avec Nuxt Content, ajoutez explicitement `@source "../content"`.

**Pattern critique** : évitez la concaténation dynamique de classes. Utilisez plutôt des objets conditionnels ou des mappings :

```vue
<!-- ❌ Non détectable -->
<div :class="`bg-${color}-500`">

<!-- ✅ Détectable -->
<div :class="{ 'bg-red-500': isRed, 'bg-blue-500': isBlue }">
```

---

## Compression Brotli Nitro et Cloudflare Pages

### Configuration Nitro

```typescript
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare_pages',
    
    compressPublicAssets: {
      brotli: true,  // Niveau 11 (maximum)
      gzip: true     // Fallback
    },
    
    prerender: {
      crawlLinks: true,
      routes: ['/'],
      autoSubfolderIndex: false // Compatibilité routage CF
    }
  }
})
```

**Point crucial** : Cloudflare Pages **ne sert pas les fichiers .br/.gz pré-compressés**. Le CDN décompresse le contenu origin et re-compresse en Brotli niveau 4. Les fichiers générés par Nitro restent utiles pour d'autres hébergeurs ou en cas d'évolution de Pages.

### Fichier _headers pour caching optimal

Placez ce fichier dans `public/` :

```
# Assets fingerprinted (hash dans filename) - cache infini
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

# Images et fonts statiques
/images/*
  Cache-Control: public, max-age=2592000, immutable
/fonts/*
  Cache-Control: public, max-age=31536000, immutable

# HTML - validation ETag obligatoire
/*.html
  Cache-Control: public, max-age=0, must-revalidate
/
  Cache-Control: public, max-age=0, must-revalidate

# Headers sécurité (toutes routes)
/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Referrer-Policy: strict-origin-when-cross-origin

# Early Hints manuels
/blog/*
  Link: </_nuxt/entry.js>; rel=modulepreload
```

### Paramètres Cloudflare Dashboard à désactiver

Ces fonctionnalités **cassent l'hydration Vue/Nuxt** :

- **Speed → Auto Minify** (JS, CSS, HTML) : **DÉSACTIVER**
- **Speed → Rocket Loader™** : **DÉSACTIVER**
- **Speed → Mirage** : **DÉSACTIVER**
- **Scrape Shield → Email Obfuscation** : **DÉSACTIVER**

Early Hints est automatiquement activé sur les domaines `*.pages.dev`. Pour les domaines personnalisés, activez-le manuellement dans Speed → Optimization.

---

## Bundle analysis : outils et seuils recommandés

### Méthodes d'analyse

```bash
# Commande intégrée Nuxt
npx nuxt analyze

# Avec flag environnement
ANALYZE=true npx nuxt build
```

```typescript
// Configuration dans nuxt.config.ts
import { visualizer } from 'rollup-plugin-visualizer'

export default defineNuxtConfig({
  build: {
    analyze: {
      template: 'treemap',
      gzipSize: true,
      brotliSize: true
    }
  },
  
  // Alternative avec rollup-plugin-visualizer
  vite: {
    plugins: [
      visualizer({
        filename: '.nuxt/stats.html',
        gzipSize: true,
        brotliSize: true,
        template: 'treemap'
      })
    ]
  }
})
```

### Seuils de taille pour blog SSG

| Métrique | Objectif | Acceptable | À optimiser |
|----------|----------|------------|-------------|
| **JS initial (gzip)** | < 100 KB | 100-200 KB | > 200 KB |
| **JS total (gzip)** | < 300 KB | 300-400 KB | > 400 KB |
| **CSS (gzip)** | < 50 KB | 50-100 KB | > 100 KB |
| **Chunk par route** | < 50 KB | 50-100 KB | > 100 KB |

Nuxt DevTools intègre également une vue Build → Analyze Build pour une analyse sans configuration additionnelle.

---

## Core Web Vitals : stratégies SSG spécifiques

### Seuils cibles (p75)

| Métrique | Bon | À améliorer | Mauvais |
|----------|-----|-------------|---------|
| **LCP** | ≤ 2.5s | 2.5-4.0s | > 4.0s |
| **INP** | ≤ 200ms | 200-500ms | > 500ms |
| **CLS** | ≤ 0.1 | 0.1-0.25 | > 0.25 |

### LCP : optimisation image hero et fonts

```vue
<template>
  <!-- Image LCP avec priorité maximale -->
  <NuxtImg
    src="/hero-banner.jpg"
    alt="Hero"
    width="1200"
    height="600"
    format="webp"
    preload
    loading="eager"
    fetchpriority="high"
    sizes="(max-width: 768px) 100vw, 1200px"
  />
</template>
```

```typescript
// nuxt.config.ts - Préchargement ressources critiques
export default defineNuxtConfig({
  app: {
    head: {
      link: [
        { rel: 'preload', as: 'image', href: '/hero-banner.webp', fetchpriority: 'high' },
        { rel: 'preconnect', href: 'https://fonts.bunny.net' }
      ]
    }
  },
  experimental: {
    inlineSSRStyles: true  // CSS critique inline
  }
})
```

### INP : lazy hydration et gestion événements

INP a remplacé FID depuis **mars 2024**. Il mesure la réactivité sur toute la session, pas seulement la première interaction.

```vue
<template>
  <!-- Hydratation différée selon visibilité/idle -->
  <LazyCommentSection hydrate-on-visible />
  <LazyNewsletter hydrate-on-idle />
  <LazyContactForm hydrate-on-interaction="focus" />
  
  <!-- Contenu statique - jamais hydraté -->
  <LazyArticleContent :hydrate="false" />
</template>

<script setup>
import debounce from 'lodash-es/debounce'

// Debounce opérations coûteuses
const handleSearch = debounce(async (query) => {
  await performSearch(query)
}, 300)

// Yield to main thread pour tâches lourdes
const handleHeavyTask = () => {
  updateUIImmediately()
  requestAnimationFrame(() => {
    setTimeout(() => performBackgroundTask(), 0)
  })
}
</script>
```

### CLS : dimensions et font-display

```css
/* Aspect ratio moderne pour images */
.image-container {
  aspect-ratio: 16 / 9;
  width: 100%;
}

/* Skeleton avec dimensions réservées */
.content-skeleton {
  aspect-ratio: 16 / 9;
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  animation: shimmer 1.5s infinite;
}

/* Font-display optimal */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter.woff2') format('woff2');
  font-display: optional; /* Meilleur pour CLS - swap si rapide, sinon fallback */
}
```

| font-display | Bloc | Swap | Usage recommandé |
|--------------|------|------|------------------|
| `swap` | ~0ms | Infini | Corps de texte (texte visible immédiatement) |
| `optional` | ~100ms | Aucun | **Meilleur CLS** - peut ignorer webfont |
| `fallback` | ~100ms | ~3s | Compromis corps de texte |

---

## Optimisation images avec @nuxt/image

Pour un hébergement zero-cost, utilisez le provider `ipxStatic` (automatique pour `nuxt generate`) qui optimise les images **au build**.

```typescript
export default defineNuxtConfig({
  modules: ['@nuxt/image'],
  
  image: {
    format: ['avif', 'webp'],  // Ordre de priorité
    quality: 80,
    screens: { sm: 640, md: 768, lg: 1024, xl: 1280 },
    densities: [1, 2],
    
    presets: {
      blogHero: {
        modifiers: { width: 1200, height: 630, fit: 'cover', quality: 85 }
      },
      thumbnail: {
        modifiers: { width: 400, height: 300, fit: 'cover', quality: 80 }
      }
    }
  }
})
```

```vue
<template>
  <!-- Picture element avec formats modernes et fallback -->
  <NuxtPicture
    src="/article-image.jpg"
    format="avif,webp"
    sizes="sm:100vw md:900px lg:1200px"
    :img-attrs="{ alt: 'Article', class: 'rounded-lg' }"
    densities="1,2"
  />
</template>
```

**Le provider Cloudflare Image Transformations** (gratuit jusqu'à 5000 transformations/mois) reste une option si vous dépassez la complexité du build-time optimization, mais `ipxStatic` suffit pour la majorité des blogs.

---

## Optimisation fonts avec @nuxt/fonts

```typescript
export default defineNuxtConfig({
  modules: ['@nuxt/fonts'],
  
  fonts: {
    defaults: {
      weights: [400, 500, 700],
      styles: ['normal'],
      subsets: ['latin']  // Subsetting critique : 139KB → 15KB
    },
    families: [
      { name: 'Inter', provider: 'bunny' },  // Alternative privacy-first à Google
      { name: 'JetBrains Mono', provider: 'bunny', weights: [400] }
    ],
    priority: ['bunny', 'google']
  }
})
```

Le module génère automatiquement des **fallback fonts métriques** via fontaine, réduisant drastiquement le CLS lors du chargement des webfonts. Les fonts sont self-hosted à `/_fonts/` pour éliminer les requêtes tierces.

**Subsetting** : limiter aux caractères latin réduit typiquement la taille de **80-90%** (139KB → 15KB pour une font complète).

---

## Configuration complète nuxt.config.ts

```typescript
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  compatibilityDate: '2025-01-01',
  srcDir: 'app',
  ssr: true,

  modules: ['@nuxt/image', '@nuxt/fonts'],

  css: ['~/assets/css/main.css'],

  // Tree shaking composables
  optimization: {
    treeShake: {
      composables: {
        client: { vue: ['onServerPrefetch'], '#app': ['definePayloadReducer'] },
        server: { vue: ['onMounted', 'onUpdated', 'onUnmounted'], '#app': ['definePayloadReviver'] }
      }
    }
  },

  // Nitro SSG + Compression
  nitro: {
    preset: 'cloudflare_pages',
    compressPublicAssets: { brotli: true, gzip: true },
    prerender: { crawlLinks: true, routes: ['/'], autoSubfolderIndex: false }
  },

  // Vite + TailwindCSS v4
  vite: {
    plugins: [tailwindcss()],
    build: {
      target: 'esnext',
      cssCodeSplit: true,
      chunkSizeWarningLimit: 200,
      rollupOptions: {
        treeshake: { preset: 'recommended', propertyReadSideEffects: false },
        output: {
          manualChunks: (id) => {
            if (id.includes('node_modules')) {
              if (id.includes('vue')) return 'vue-vendor'
            }
          }
        }
      }
    }
  },

  // Images
  image: {
    format: ['avif', 'webp'],
    quality: 80,
    densities: [1, 2]
  },

  // Fonts
  fonts: {
    defaults: { weights: [400, 700], subsets: ['latin'] },
    families: [{ name: 'Inter', provider: 'bunny' }]
  },

  // Performance expérimentale
  experimental: {
    inlineSSRStyles: true,
    payloadExtraction: true,
    defaults: { nuxtLink: { prefetchOn: 'interaction' } }
  },

  // Head
  app: {
    head: {
      htmlAttrs: { lang: 'fr' },
      link: [{ rel: 'preconnect', href: 'https://fonts.bunny.net' }]
    }
  }
})
```

---

## Checklist déploiement zero-cost

- ✅ `compressPublicAssets` avec Brotli + Gzip activés
- ✅ Fichier `_headers` avec cache immutable pour `/_nuxt/*`
- ✅ **Désactiver** Auto Minify, Rocket Loader, Mirage dans Cloudflare
- ✅ TailwindCSS v4 avec `@tailwindcss/vite` et configuration CSS-native
- ✅ Tree shaking preset `recommended` + imports ESM nommés
- ✅ `@nuxt/image` avec `ipxStatic` pour optimisation build-time
- ✅ `@nuxt/fonts` avec provider Bunny et subsetting latin
- ✅ Lazy hydration (`hydrate-on-visible`) pour composants non-critiques
- ✅ `font-display: optional` ou `swap` selon priorité CLS/visibilité
- ✅ Images LCP avec `preload`, `fetchpriority="high"`, dimensions explicites

Cette configuration génère un blog SSG avec des bundles JS < **100KB gzip**, un CSS TailwindCSS minimal, des images modernes (AVIF/WebP), et d'excellents scores Core Web Vitals depuis l'edge Cloudflare global.