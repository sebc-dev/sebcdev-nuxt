# Performance Monitoring

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Dev diagnostics** | `@nuxt/hints` | Core Web Vitals temps réel, diff hydratation serveur/client |
| **Prod tracking** | `@nuxtjs/web-vitals` | Tracking continu, analytics intégrées |
| **Bundle analysis** | `npx nuxi analyze` | Visualisation interactive du bundle (intégré) |
| **Métrique clé** | **INP** (remplace FID depuis mars 2024) | Interaction to Next Paint, cible ≤200ms |

**Core Web Vitals cibles:**

| Métrique | Seuil | Impact hydratation |
|----------|-------|-------------------|
| **LCP** (Largest Contentful Paint) | ≤2.5s | Images, fonts |
| **INP** (Interaction to Next Paint) | ≤200ms | Hydratation lazy critique |
| **CLS** (Cumulative Layout Shift) | ≤0.1 | Fallbacks dimensionnés |

**Configuration développement:**

```typescript
// nuxt.config.ts - modules dev performance
modules: [
  // ... autres modules
  '@nuxt/hints',  // Diagnostics Core Web Vitals en dev
],

// Production uniquement
$production: {
  modules: ['@nuxtjs/web-vitals'],
},

webVitals: {
  provider: 'log',  // ou 'ga' pour Google Analytics
  debug: process.env.NODE_ENV === 'development',
}
```

## Lighthouse CI Configuration

**lighthouserc.js avec assertions Core Web Vitals :**

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
        // Core Web Vitals - seuils stricts
        'largest-contentful-paint': ['error', { maxNumericValue: 2500 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        'total-blocking-time': ['error', { maxNumericValue: 200 }],

        // Score global
        'categories:performance': ['error', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 0.9 }],
      }
    },
    upload: {
      target: 'temporary-public-storage'
    }
  }
};
```

**GitHub Actions workflow :**

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
      - uses: pnpm/action-setup@v4
        with:
          version: 10
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm run generate

      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v12
        with:
          configPath: './lighthouserc.js'
          uploadArtifacts: true
          temporaryPublicStorage: true
          runs: 3
```

## Plugin web-vitals Debug (Développement)

Plugin client pour attribution détaillée des Core Web Vitals en développement :

```typescript
// plugins/web-vitals-debug.client.ts
import { onCLS, onINP, onLCP } from 'web-vitals/attribution'

export default defineNuxtPlugin(() => {
  if (import.meta.dev) {
    onLCP((metric) => {
      console.group('🎯 LCP Attribution')
      console.log('Value:', metric.value, 'ms')
      console.log('Element:', metric.attribution.element)
      console.log('Resource Load Delay:', metric.attribution.resourceLoadDelay, 'ms')
      console.log('Resource Load Duration:', metric.attribution.resourceLoadDuration, 'ms')
      console.groupEnd()
    })

    onINP((metric) => {
      console.group('⚡ INP Attribution')
      console.log('Value:', metric.value, 'ms')
      console.log('Target:', metric.attribution.interactionTarget)
      console.log('Input Delay:', metric.attribution.inputDelay, 'ms')
      console.log('Processing Duration:', metric.attribution.processingDuration, 'ms')
      console.log('Presentation Delay:', metric.attribution.presentationDelay, 'ms')
      console.groupEnd()
    })

    onCLS((metric) => {
      console.group('📐 CLS Attribution')
      console.log('Value:', metric.value)
      console.log('Largest Shift Target:', metric.attribution.largestShiftTarget)
      console.log('Largest Shift Time:', metric.attribution.largestShiftTime, 'ms')
      console.groupEnd()
    })
  }
})
```

**Installation :**

```bash
pnpm add web-vitals
```

**Stack RUM gratuite recommandée :**
- Cloudflare Web Analytics (intégré si hébergé sur CF)
- Google Search Console (données CrUX)
- CrUX API pour données field
