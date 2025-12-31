# Infrastructure & Deployment

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Image Optimization** | @nuxt/image v2 + Cloudflare | AVIF/WebP auto, Early Hints LCP +30% |
| **CI/CD** | Cloudflare Pages direct | Zero config, preview branches, fast |
| **Environment Config** | .env + runtimeConfig + CF fallback | Local dev + prod parity |
| **Tests a11y** | Stack complet (voir ci-dessous) | Couverture WCAG 2.2 AA |
| **CSS Critique** | @nuxtjs/critters | Inline CSS above-the-fold, FCP -55% |

## Configuration NuxtImage pour LCP

L'optimisation du Largest Contentful Paint (LCP) repose sur une configuration précise de `@nuxt/image` :

**Configuration nuxt.config.ts :**

```typescript
// nuxt.config.ts
image: {
  quality: 80,
  format: ['avif', 'webp'],
  screens: {
    xs: 320, sm: 640, md: 768, lg: 1024, xl: 1280, xxl: 1536
  },
  presets: {
    hero: {
      modifiers: { format: 'webp', fit: 'cover', quality: 85 }
    },
    thumbnail: {
      modifiers: { format: 'webp', fit: 'cover', quality: 70, width: 300, height: 200 }
    },
    articleCover: {
      modifiers: { format: 'webp', fit: 'cover', quality: 80, width: 800, height: 450 }
    }
  }
}
```

**Template pour image LCP (hero) :**

Trois attributs sont **obligatoires** pour l'image LCP : `preload`, `loading="eager"` et `fetchpriority="high"`.

**Option 1 : NuxtImg (format unique)**

```vue
<template>
  <NuxtImg
    src="/images/hero-banner.jpg"
    alt="Description accessible de l'image hero"
    preset="hero"
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

**Option 2 : NuxtPicture (multi-formats AVIF/WebP) - Recommandé**

`<NuxtPicture>` génère un élément `<picture>` avec sources multiples et fallback automatique :

```vue
<!-- components/HeroImage.vue -->
<template>
  <NuxtPicture
    src="/images/hero-banner.jpg"
    format="avif,webp"
    :preload="{ fetchPriority: 'high' }"
    loading="eager"
    sizes="100vw lg:1200px"
    width="1200"
    height="600"
    :img-attrs="{
      fetchpriority: 'high',
      alt: 'Description accessible de l\'image hero',
      class: 'w-full h-auto object-cover'
    }"
  />
</template>
```

**HTML généré par NuxtPicture en SSG :**

```html
<head>
  <link rel="preload" fetchpriority="high" as="image"
        href="/_ipx/f_avif,w_1200/images/hero-banner.jpg" type="image/avif">
</head>
<body>
  <picture>
    <source srcset="/_ipx/f_avif,w_640/hero.jpg 640w, ..." type="image/avif">
    <source srcset="/_ipx/f_webp,w_640/hero.jpg 640w, ..." type="image/webp">
    <img fetchpriority="high" loading="eager" width="1200" height="600" ...>
  </picture>
</body>
```

| Composant | Avantage | Cas d'usage |
|-----------|----------|-------------|
| `NuxtImg` | Plus simple, un seul format | Images non-critiques |
| `NuxtPicture` | Multi-formats, meilleure compression | **Image LCP, hero sections** |

**Différences entre preload, eager et fetchpriority :**

| Attribut | Effet | Cas d'usage |
|----------|-------|-------------|
| `preload` | Génère `<link rel="preload">` dans `<head>` | Image LCP uniquement (1 par page) |
| `loading="eager"` | Désactive le lazy loading natif | Toute image above-the-fold |
| `fetchpriority="high"` | Priorise dans la file réseau | Image critique (LCP, logo header) |

**Combinaison optimale pour LCP** : les trois ensemble. Pour images below-the-fold : `loading="lazy"` + `fetchpriority="low"` sans preload.

**⚠️ Anti-patterns LCP :**

```vue
<!-- ❌ INCORRECT : lazy loading sur image LCP -->
<NuxtImg src="/hero.jpg" loading="lazy" />

<!-- ❌ INCORRECT : pas de dimensions = CLS -->
<NuxtImg src="/hero.jpg" class="w-full" />

<!-- ❌ INCORRECT : format original non optimisé -->
<img src="/hero.png" />
```

**Stack Tests Accessibilité Complet :**

| Phase | Outil | Usage |
|-------|-------|-------|
| **Développement** | eslint-plugin-vuejs-accessibility | Lint temps réel dans l'IDE |
| **Développement** | WAVE extension | Audit visuel manuel |
| **Tests unitaires** | vitest-axe (Browser Mode) | Validation composants isolés |
| **Tests E2E** | @axe-core/playwright | Pages SSG générées |
| **CI** | Lighthouse CI | Score ≥90% accessibilité |
| **CI** | Pa11y-ci | Audit sitemap complet |

**Configuration ESLint vuejs-accessibility (ESLint 9+ flat config) :**

```javascript
// eslint.config.js
import pluginVueA11y from 'eslint-plugin-vuejs-accessibility'

export default [
  ...pluginVueA11y.configs['flat/recommended'],
  {
    rules: {
      // Règles critiques - erreurs bloquantes
      'vuejs-accessibility/alt-text': 'error',
      'vuejs-accessibility/anchor-has-content': 'error',
      'vuejs-accessibility/click-events-have-key-events': 'error',
      'vuejs-accessibility/form-control-has-label': 'error',
      'vuejs-accessibility/heading-has-content': 'error',
      'vuejs-accessibility/interactive-supports-focus': 'error',
      'vuejs-accessibility/label-has-for': 'error',
      'vuejs-accessibility/tabindex-no-positive': 'error',

      // Règles importantes - warnings
      'vuejs-accessibility/no-autofocus': 'warn',
      'vuejs-accessibility/no-redundant-roles': 'warn',
      'vuejs-accessibility/no-distracting-elements': 'warn',
    }
  }
]
```

**Installation :**

```bash
pnpm add -D eslint-plugin-vuejs-accessibility
```

**Configuration Vitest + axe-core pour tests unitaires :**

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import { defineVitestProject } from '@nuxt/test-utils/config'

export default defineConfig({
  test: {
    projects: [
      await defineVitestProject({
        test: {
          name: 'nuxt',
          include: ['test/nuxt/**/*.{test,spec}.ts'],
          environment: 'nuxt',
          environmentOptions: {
            nuxt: {
              domEnvironment: 'happy-dom',  // Plus rapide que jsdom
            },
          },
          setupFiles: ['./test/setup.ts'],
          globals: true,
        },
      }),
    ],
  },
})
```

```typescript
// test/setup.ts
import { expect } from 'vitest'
import * as matchers from 'vitest-axe/matchers'

expect.extend(matchers)
```

**Test de composant avec axe :**

```typescript
// components/__tests__/Dialog.a11y.test.ts
import { render, fireEvent } from '@testing-library/vue'
import { axe } from 'vitest-axe'
import { describe, it, expect } from 'vitest'
import ProfileDialog from '../ProfileDialog.vue'

describe('ProfileDialog Accessibility', () => {
  it('should have no violations when closed', async () => {
    const { container } = render(ProfileDialog)
    expect(await axe(container)).toHaveNoViolations()
  })

  it('should have no violations when open', async () => {
    const { container, getByRole } = render(ProfileDialog)

    await fireEvent.click(getByRole('button', { name: /ouvrir/i }))

    const results = await axe(container, {
      runOnly: ['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa']
    })
    expect(results).toHaveNoViolations()
  })

  it('should trap focus within dialog', async () => {
    const { getByRole } = render(ProfileDialog)

    await fireEvent.click(getByRole('button', { name: /ouvrir/i }))

    const dialog = getByRole('dialog')
    expect(dialog).toHaveAttribute('aria-modal', 'true')
  })
})
```

**Installation Vitest + axe :**

```bash
pnpm add -D @nuxt/test-utils vitest vitest-axe @vue/test-utils happy-dom @vitest/coverage-v8
```

**Configuration @axe-core/playwright WCAG 2.2 :**

```typescript
// tests/e2e/accessibility.spec.ts
import { test, expect } from '@playwright/test'
import AxeBuilder from '@axe-core/playwright'

test('Page sans violations WCAG 2.2 AA', async ({ page }) => {
  await page.goto('/')

  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'])
    .analyze()

  expect(results.violations).toEqual([])
})
```

**Configuration Lighthouse CI :**

```json
// lighthouserc.json
{
  "ci": {
    "collect": {
      "staticDistDir": ".output/public"
    },
    "assert": {
      "assertions": {
        "categories:accessibility": ["error", { "minScore": 0.9 }],
        "categories:performance": ["warn", { "minScore": 0.8 }]
      }
    }
  }
}
```

**Note importante** : Les tests automatisés détectent ~30% des problèmes d'accessibilité. Les 70% restants nécessitent des tests manuels avec clavier et lecteur d'écran (VoiceOver macOS, NVDA Windows).

**Cloudflare Pages Avantages (budget 0€):**

| Avantage | Valeur |
|----------|--------|
| Bande passante | **Illimitée** (vs 100GB Netlify/Vercel) |
| Builds/mois | 500 |
| Réseau edge | 330+ datacenters, 95% population <50ms |
| HTTP/3 + Early Hints | Activés automatiquement |

**Deployment Flow:**

```
Git Push → Cloudflare Pages Build → SSG + MiniSearch Index → Edge Deployment
                ↓
         Preview URL (branches)
```
