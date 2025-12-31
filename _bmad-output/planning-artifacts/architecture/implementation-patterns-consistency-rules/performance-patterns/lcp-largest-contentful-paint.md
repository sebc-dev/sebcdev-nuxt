# LCP (Largest Contentful Paint)

## Diagnostic des sous-parties LCP

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

## Plugin web-vitals avec attribution détaillée

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

## Seuils Core Web Vitals

| Métrique | Bon | À améliorer | Mauvais |
|----------|-----|-------------|---------|
| **LCP** | ≤2.5s | 2.5s - 4s | >4s |
| **INP** | ≤200ms | 200ms - 500ms | >500ms |
| **CLS** | ≤0.1 | 0.1 - 0.25 | >0.25 |

## Checklist LCP Nuxt 4 + Cloudflare Pages

```markdown
# Configuration initiale
- [ ] `@nuxt/image` configuré avec formats `['avif', 'webp']`
- [ ] `@nuxt/fonts` avec provider local pour fonts self-hosted
- [ ] `@nuxtjs/critters` pour extraction CSS critique
- [ ] TailwindCSS 4.x via `@tailwindcss/vite` plugin
- [ ] `nitro.preset: 'cloudflare_pages'` ou `cloudflare_pages_static`

# Image LCP (1 par page)
- [ ] Utiliser `<NuxtPicture>` avec `format="avif,webp"`
- [ ] Ajouter `:preload="{ fetchPriority: 'high' }"`
- [ ] Définir `loading="eager"`
- [ ] Spécifier `width` et `height` explicites
- [ ] Configurer `sizes` pour responsive

# Images below-the-fold
- [ ] `loading="lazy"` sur toutes les images non-LCP
- [ ] `fetchpriority="low"` pour images décoratives
- [ ] Pas de `preload`

# Fonts
- [ ] Self-hosting WOFF2 dans `public/fonts/`
- [ ] Preload de la police principale avec `crossorigin="anonymous"`
- [ ] `font-display: optional` pour zéro CLS
- [ ] `line-height` explicite sur `body`
- [ ] Subsets limités (`latin`, `latin-ext`)

# Cloudflare Pages
- [ ] Fichier `public/_headers` créé
- [ ] Cache immutable pour `/_nuxt/*`
- [ ] `X-Robots-Tag: noindex` sur `*.pages.dev`
- [ ] Early Hints Link headers configurés

# Monitoring
- [ ] Plugin `web-vitals.client.ts` en place (dev)
- [ ] Tests PageSpeed Insights mobile < 2.5s LCP
- [ ] Lighthouse CI configuré avec assertions Core Web Vitals

# Validation build SSG
- [ ] `pnpm run generate` sans erreurs
- [ ] Images optimisées dans `.output/public/_ipx/`
- [ ] Taille bundle CSS < 50KB gzipped
- [ ] Pas de console errors Core Web Vitals
```

---
