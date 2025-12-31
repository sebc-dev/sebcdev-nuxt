# Core Architectural Decisions

## Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Content structure: By language with separate collections (`articles_fr`, `articles_en`)
- Frontmatter schema: English constants, auto-calculated readingTime
- Component architecture: Structured folders with custom Prose components
- SEO/GEO: Auto-generated llms.txt, Schema.org via @unhead

**Important Decisions (Shape Architecture):**
- State management: Vue composables only (no Pinia)
- Image optimization: @nuxt/image + Cloudflare provider
- CI/CD: Cloudflare Pages direct Git integration

**Deferred Decisions (Post-MVP):**
- Newsletter integration
- Comments system
- Advanced analytics (heatmaps)

## Content Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **File Organization** | By language (`content/fr/`, `content/en/`) | Natural fit with @nuxtjs/i18n, clear separation |
| **Collections** | Separate per language (`articles_fr`, `articles_en`) | Performance D1, typage précis, code auto-documenté |
| **Frontmatter Constants** | English (`ai`, `tutorial`, `beginner`) | Consistency, i18n-agnostic |
| **Reading Time** | Auto-calculated (200 words/min) | Less maintenance, always accurate |

**Frontmatter Schema:**

```yaml
title: string
description: string
slug: string
pillar: 'ai' | 'engineering' | 'ux'
category: 'news' | 'tutorial' | 'deep-dive' | 'case-study' | 'retrospective'
level: 'all' | 'beginner' | 'intermediate' | 'advanced'
tags: string[]
publishedAt: date
updatedAt: date
image: string
draft: boolean
translationKey: string          # Optionnel - Lie les traductions FR/EN d'un même article
```

**Champ `translationKey` (i18n Content) :**

Le champ `translationKey` permet de lier les versions française et anglaise d'un même article pour :
- Afficher le switcher de langue sur les pages article
- Générer les balises `hreflang` alternate correctes
- Proposer "Lire en français/anglais" automatiquement

```yaml
# content/fr/guide-nuxt-4.md
---
title: "Guide complet Nuxt 4"
translationKey: "nuxt-4-complete-guide"
---

# content/en/nuxt-4-guide.md
---
title: "Complete Nuxt 4 Guide"
translationKey: "nuxt-4-complete-guide"  # Même clé = même article
---
```

**Requête pour trouver la traduction :**

```typescript
// Trouver la version EN d'un article FR
const { data: translation } = await useAsyncData(
  `translation-${article.translationKey}`,
  () => queryCollection('articles_en')
    .where('translationKey', '=', article.translationKey)
    .first()
)
```

**Configuration Collections (content.config.ts) :**

```typescript
import { defineContentConfig, defineCollection, asSitemapCollection } from '@nuxt/content'
import { z } from 'zod/v4'  // ⚠️ Zod 4 : utiliser 'zod/v4' pour l'API moderne

// Schema partagé entre les collections
const articleSchema = z.object({
  title: z.string(),
  description: z.string(),
  slug: z.string(),
  pillar: z.enum(['ai', 'engineering', 'ux']),
  category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
  level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),
  tags: z.array(z.string()).default([]),
  publishedAt: z.iso.date(),           // ⚠️ Zod 4 : z.iso.date() pour format YYYY-MM-DD (compatible JSON Schema)
  updatedAt: z.iso.date().optional(),  // ⚠️ z.date() n'a PAS d'équivalent JSON Schema - utiliser z.iso.date()
  image: z.string().optional(),
  draft: z.boolean().default(false),
  translationKey: z.string().optional(),  // Lie les traductions FR/EN d'un même article
})

// Indexes partagés
const articleIndexes = [
  { columns: ['path'], unique: true },
  { columns: ['pillar'] },
  { columns: ['publishedAt'] },
  { columns: ['draft'] },
  { columns: ['draft', 'publishedAt'], name: 'idx_published' },
]

export default defineContentConfig({
  collections: {
    articles_fr: defineCollection(
      asSitemapCollection({
        type: 'page',
        source: { include: 'fr/**/*.md', prefix: '/blog' },
        schema: articleSchema,
        indexes: articleIndexes,
      })
    ),
    articles_en: defineCollection(
      asSitemapCollection({
        type: 'page',
        source: { include: 'en/**/*.md', prefix: '/blog' },
        schema: articleSchema,
        indexes: articleIndexes,
      })
    ),
  }
})
```

**Avantages des collections séparées :**

| Aspect | Bénéfice |
|--------|----------|
| **Performance D1** | Requêtes sur table plus petite = moins de rows_read |
| **Indexes isolés** | Index `idx_published` optimisé par langue |
| **Typage TypeScript** | `Collections['articles_fr']` vs filtrage runtime |
| **Code auto-documenté** | `queryCollection('articles_fr')` - intention claire |
| **Évolutivité** | Ajouter une 3e langue = nouvelle collection isolée |

## Frontend Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Component Structure** | `ui/`, `content/`, `layout/`, `search/` | Clear separation of concerns |
| **State Management** | Vue composables only | SSG blog doesn't need global store |
| **Prose Components** | Custom (ProseCode, ProseH2, ProseDetails) | Required for FR11-13 (code blocks), FR8 (ToC) |

**Component Organization:**

```
app/components/
├── ui/                    # shadcn-vue
├── content/               # ArticleCard, TableOfContents, ReadingProgress
├── layout/                # TheHeader, TheFooter, LanguageSwitcher
└── search/                # SearchCommand, SearchFilters
```

**Page & Layout Transitions:**

Configuration des transitions de pages et layouts dans `nuxt.config.ts` :

```typescript
export default defineNuxtConfig({
  app: {
    pageTransition: { name: 'page', mode: 'out-in' },
    layoutTransition: { name: 'layout', mode: 'out-in' }
  }
})
```

Classes CSS correspondantes dans `main.css` :

```css
/* Transition de pages */
.page-enter-active,
.page-leave-active {
  transition: opacity 200ms, transform 200ms;
}

.page-enter-from {
  opacity: 0;
  transform: translateY(10px);
}

.page-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

/* Transition de layouts */
.layout-enter-active,
.layout-leave-active {
  transition: opacity 300ms;
}

.layout-enter-from,
.layout-leave-to {
  opacity: 0;
}
```

| Option | Valeur | Description |
|--------|--------|-------------|
| `mode: 'out-in'` | Défaut recommandé | L'ancienne page sort avant que la nouvelle entre (évite chevauchements) |
| `mode: 'in-out'` | Rare | La nouvelle page entre avant que l'ancienne sorte |
| `mode: 'default'` | Simultané | Les deux transitions en parallèle |

**Note accessibilité** : Combiner avec `motion-safe:` pour respecter `prefers-reduced-motion` :

```css
.page-enter-active,
.page-leave-active {
  @apply motion-safe:transition-all motion-safe:duration-200;
}
```

## SEO & GEO Implementation

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Suite SEO** | @nuxtjs/seo (bundle) | Inclut sitemap, robots, schema.org, og-image, link-checker |
| **OG Images** | nuxt-og-image (zeroRuntime) | Génération au build via Satori, 100% SSG, CF Pages gratuit |
| **llms.txt** | Module nuxt-llms (auto) | Navigation IA efficace, génération automatique avec @nuxt/content ^3.2.0 |
| **RSS** | Server route generation | Native Content 3 integration |
| **i18n SEO** | useLocaleHead() | Injection auto hreflang + og:locale avec `language` obligatoire |

**Sous-modules inclus dans @nuxtjs/seo :**
- `@nuxtjs/sitemap` - Sitemap XML avec hreflang i18n auto
- `@nuxtjs/robots` - robots.txt dynamique
- `nuxt-schema-org` - JSON-LD Schema.org
- `nuxt-og-image` - Génération images OG au build
- `nuxt-link-checker` - Validation liens (dev)
- `nuxt-seo-utils` - Utilitaires (useSiteConfig, useLocaleHead...)

**Optimisations GEO (Princeton/Georgia Tech):**

| Stratégie | Impact mesuré | Implémentation |
|-----------|---------------|----------------|
| **Citation Addition** | +115% visibilité | Référencer des sources autoritatives |
| **Statistics Addition** | +22-37% | Une statistique tous les 150-200 mots |
| **Quotation Addition** | Élevé | Intégrer des citations d'experts |

**Structure contenu GEO :**
- Réponse directe dans les **40-60 premiers mots**
- Hiérarchie claire H2/H3
- Listes à puces et tableaux comparatifs
- Passages **autonomes et compréhensibles** sans contexte

**Configuration nuxt-llms détaillée :**

```typescript
// nuxt.config.ts
llms: {
  domain: 'https://sebc.dev',  // REQUIS - URL complète
  title: 'sebc.dev',
  description: 'Blog technique sur le développement web moderne',

  sections: [
    {
      title: 'Articles',
      description: 'Tous les articles du blog',
      contentCollection: 'articles_fr',  // Collection Nuxt Content
      contentFilters: [
        { field: 'draft', operator: '<>', value: true }
      ]
    },
    {
      title: 'Optional',  // Section ignorée si contexte LLM limité
      links: [
        { title: 'À propos', href: '/about', description: 'Qui suis-je' }
      ]
    }
  ],

  // Activer llms-full.txt (désactivé par défaut)
  full: {
    title: 'Documentation Complète',
    description: 'Tout le contenu du blog en un fichier'
  }
}
```

**llms.txt vs llms-full.txt :**

| Fichier | Fonction | Taille typique | Usage |
|---------|----------|----------------|-------|
| `llms.txt` | Index avec liens | ~1,600 mots | Guide rapide, économise tokens |
| `llms-full.txt` | Contenu complet | ~58,000 mots | Contexte exhaustif (Claude 200K+) |

**Extension dynamique via hooks Nitro :**

```typescript
// server/plugins/llms-extend.ts
export default defineNitroPlugin((nitroApp) => {
  // Hook pour /llms.txt
  nitroApp.hooks.hook('llms:generate', (event, options) => {
    options.sections.push({
      title: 'API Documentation',
      links: [
        { title: 'Endpoints', href: '/api/docs', description: 'REST API' }
      ]
    })
  })

  // Hook pour /llms-full.txt (contenu détaillé)
  nitroApp.hooks.hook('llms:generate:full', (event, options, contents) => {
    contents.push(`
## Informations techniques

### Stack technologique
- **Framework**: Nuxt 4.2.x avec Vue 3.5
- **CMS**: Nuxt Content 3 avec SQLite/D1
- **Déploiement**: Cloudflare Pages (SSG)
- **Langues**: Français (fr), English (en)

### Formats de contenu
Tous les articles sont disponibles en Markdown avec:
- Blocs de code avec coloration syntaxique Shiki
- Métadonnées Schema.org structurées
    `)
  })
})
```

**Plugin i18n multilingue pour nuxt-llms :**

```typescript
// server/plugins/llms-i18n.ts
export default defineNitroPlugin((nitroApp) => {
  nitroApp.hooks.hook('llms:generate', (event, options) => {
    const locale = detectLocaleFromEvent(event)

    const config = {
      en: {
        title: 'sebc.dev - Technical Blog',
        description: 'Technical articles on modern web development with Nuxt and Vue.js.',
        contentCollection: 'articles_en'
      },
      fr: {
        title: 'sebc.dev - Blog Technique',
        description: 'Articles techniques sur le développement web moderne avec Nuxt et Vue.js.',
        contentCollection: 'articles_fr'
      }
    }

    const localeConfig = config[locale] || config.en
    Object.assign(options, localeConfig)

    // Mettre à jour les sections avec la bonne collection
    options.sections = options.sections.map(section => ({
      ...section,
      contentCollection: section.contentCollection
        ? localeConfig.contentCollection
        : section.contentCollection
    }))
  })
})

function detectLocaleFromEvent(event: H3Event): 'en' | 'fr' {
  const acceptLanguage = getHeader(event, 'accept-language') || ''
  return acceptLanguage.startsWith('fr') ? 'fr' : 'en'
}
```

**Blockquote GEO enrichi (stats auteur) :**

Le blockquote llms.txt doit inclure des métriques concrètes pour renforcer l'autorité :

```markdown
# sebc.dev - Blog Technique

> Blog spécialisé développement Nuxt.js et Vue.js. 30+ articles techniques
> publiés depuis 2023. Mise à jour hebdomadaire. Auteur: développeur senior
> avec 10 ans d'expérience Vue.js, contributeur open source Nuxt.

Ce blog couvre les patterns avancés Nuxt 4, l'optimisation performance,
et les architectures SSG. Chaque article inclut des exemples testés.
```

**Statistiques trafic IA 2025 :**
- Trafic référé IA : **+527%** entre janvier et mai 2025
- Taux de conversion visiteurs IA : **4,4x supérieur** au trafic organique
- Source : Études BrightEdge et Search Engine Land

**Anti-patterns nuxt-llms à éviter :**

| Erreur | Conséquence | Solution |
|--------|-------------|----------|
| `domain` absent | Erreur build | Toujours définir l'URL complète HTTPS |
| `ssr: false` | Fichiers non générés | Garder `ssr: true` (défaut) |
| Fichier dans `public/llms.txt` | Conflit avec module | Supprimer le fichier statique |
| Liens vers pages authentifiées | Échec crawl AI | Liens publics uniquement |
| Ordre modules incorrect | Intégration i18n cassée | `@nuxtjs/i18n` avant `@nuxt/content` |
| contentFilters trop restrictifs | Aucun contenu généré | Tester filtres avec `queryCollection()` d'abord |

## Infrastructure & Deployment

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Image Optimization** | @nuxt/image v2 + Cloudflare | AVIF/WebP auto, Early Hints LCP +30% |
| **CI/CD** | Cloudflare Pages direct | Zero config, preview branches, fast |
| **Environment Config** | .env + runtimeConfig + CF fallback | Local dev + prod parity |
| **Tests a11y** | Stack complet (voir ci-dessous) | Couverture WCAG 2.2 AA |
| **CSS Critique** | @nuxtjs/critters | Inline CSS above-the-fold, FCP -55% |

### Configuration NuxtImage pour LCP

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

## Tree Shaking et Optimisation Bundle

### Tree Shaking Composables Nuxt 4

Nuxt 4 intègre un tree shaking natif des composables via l'option `optimization.treeShake`, permettant d'éliminer automatiquement le code serveur du bundle client et inversement :

```typescript
// nuxt.config.ts
optimization: {
  treeShake: {
    composables: {
      // Composables Vue éliminés du bundle client (server-only)
      client: {
        vue: ['onServerPrefetch', 'onRenderTracked', 'onRenderTriggered'],
        '#app': ['definePayloadReducer', 'onPrehydrate']
      },
      // Composables Vue éliminés du bundle serveur (client-only)
      server: {
        vue: ['onMounted', 'onUpdated', 'onUnmounted', 'onBeforeMount',
              'onBeforeUpdate', 'onBeforeUnmount', 'onActivated', 'onDeactivated'],
        '#app': ['definePayloadReviver']
      }
    }
  }
}
```

### Bonnes Pratiques Imports ESM

Les imports nommés sont essentiels pour un tree shaking efficace. Évitez les imports namespace et les bibliothèques CommonJS :

```typescript
// ✅ Tree-shakeable - imports nommés
import { ref, computed } from 'vue'
import { debounce } from 'lodash-es'
import { Home, Settings } from 'lucide-vue-next'

// ❌ Anti-patterns bloquant le tree shaking
import * as Vue from 'vue'           // Namespace import → bundle entier
import lodash from 'lodash'          // CJS → pas de tree shaking
import { icons } from 'lucide-vue-next'  // Objet entier → toutes les icônes
```

| Pattern | Résultat | Impact bundle |
|---------|----------|---------------|
| `import { fn } from 'lib'` | ✅ Tree-shakeable | Minimal |
| `import * as lib from 'lib'` | ❌ Namespace | Bundle entier |
| `import lib from 'cjs-lib'` | ❌ CommonJS | Bundle entier |
| `import { obj } from 'lib'` puis `obj.fn` | ⚠️ Dépend | Peut inclure tout `obj` |

**Bibliothèques ESM recommandées :**

| Éviter (CJS) | Utiliser (ESM) |
|--------------|----------------|
| `lodash` | `lodash-es` |
| `date-fns` (import global) | `date-fns` (imports nommés) |
| `@fortawesome/fontawesome-free` | `lucide-vue-next` (icons individuels) |

### Seuils de Taille Bundle

Objectifs pour un blog SSG performant :

| Métrique | 🎯 Objectif | ✅ Acceptable | ⚠️ À optimiser |
|----------|-------------|---------------|----------------|
| **JS initial** (gzip) | < 100 KB | 100-200 KB | > 200 KB |
| **JS total** (gzip) | < 300 KB | 300-400 KB | > 400 KB |
| **CSS** (gzip) | < 50 KB | 50-100 KB | > 100 KB |
| **Chunk par route** | < 50 KB | 50-100 KB | > 100 KB |

**Commande d'analyse :**

```bash
# Analyse interactive du bundle
npx nuxi analyze

# Ou avec variable d'environnement
ANALYZE=true pnpm run build
```

## Modules Nuxt Finaux

```typescript
// ORDRE CRITIQUE: @nuxtjs/seo AVANT @nuxt/content
modules: [
  '@nuxt/image',
  '@nuxt/fonts',         // Fonts optimisées avec fallbacks métriques
  '@nuxtjs/i18n',
  '@nuxtjs/seo',         // Suite SEO complète (sitemap, robots, schema.org, og-image, link-checker)
  '@nuxt/content',       // ← APRÈS @nuxtjs/seo
  'nuxt-llms',           // Génération automatique /llms.txt
  'nuxt-security',       // Security headers + CSP hash generation SSG
  'nuxt-vitalizer',      // Optimisation LCP
  '@nuxtjs/critters',    // CSS critique inline (FCP -55%)
  'shadcn-nuxt',
  '@nuxtjs/color-mode',  // Dark mode avec classe .dark
]

// Configuration site (requise par @nuxtjs/seo)
site: {
  url: 'https://sebc.dev',
  name: 'sebc.dev',
  description: 'Blog technique sur le développement web',
  defaultLocale: 'en',  // Cohérent avec i18n.defaultLocale
}

// Configuration @nuxtjs/seo - Meta tags globaux
seo: {
  meta: {
    ogSiteName: 'sebc.dev',
    twitterCard: 'summary_large_image',
  }
}

// Configuration nuxt-og-image (inclus dans @nuxtjs/seo)
ogImage: {
  zeroRuntime: true,           // ESSENTIEL pour SSG pur (pas de server functions)
  runtimeCacheStorage: false,  // Pas de cache runtime en SSG
  defaults: {
    renderer: 'satori',        // Vue → SVG → PNG au build time
    width: 1200,
    height: 630,               // Ratio 1.91:1 standard OG
  }
}

// Configuration nuxt-schema-org (inclus dans @nuxtjs/seo)
schemaOrg: {
  // Optimisations SSG
  reactive: false,    // Désactive la réactivité client (inutile en SSG)
  minify: true,       // Minifie le JSON-LD en production
  defaults: true,     // Génère automatiquement WebSite et WebPage

  // Identité du propriétaire (Person pour blog personnel)
  identity: {
    type: 'Person',
    name: 'Sébastien C.',
    givenName: 'Sébastien',
    familyName: 'C.',
    url: 'https://sebc.dev',
    image: '/images/profile.jpg',
    description: 'Développeur full-stack spécialisé Vue.js et Nuxt',
    jobTitle: 'Lead Developer',

    // URLs équivalentes (social proof E-E-A-T)
    sameAs: [
      'https://github.com/sebcdev',
      'https://twitter.com/sebcdev',
      'https://linkedin.com/in/sebcdev'
    ],

    // Expertise technique (renforce E-E-A-T)
    knowsAbout: [
      { '@type': 'Thing', name: 'Vue.js', sameAs: 'https://en.wikipedia.org/wiki/Vue.js' },
      { '@type': 'Thing', name: 'Nuxt', sameAs: 'https://en.wikipedia.org/wiki/Nuxt.js' },
      'TypeScript',
      'Node.js',
      'Cloud Architecture'
    ]
  }
}

// Configuration @nuxtjs/critters - CSS critique inline
critters: {
  config: {
    preload: 'swap',      // Charge le reste du CSS en async
    inlineFonts: false,   // Ne pas inliner les @font-face (fonts self-hosted)
    pruneSource: false,   // Garder le CSS complet pour le chargement async
  }
}

// Configuration @nuxt/fonts - Fonts optimisées avec fallbacks métriques
// @nuxt/fonts utilise fontaine + capsize pour générer des fallbacks avec métriques ajustées
fonts: {
  // Poids par défaut (limiter pour réduire la taille du bundle)
  defaults: {
    weights: [400, 600, 700],
    styles: ['normal'],
    subsets: ['latin', 'latin-ext']
  },

  families: [
    // Self-hosting pour SSG (recommandé - zéro requête externe)
    {
      name: 'Satoshi',
      provider: 'local',
      src: '/fonts/Satoshi-Variable.woff2',
      weight: '400 700'
    },
    {
      name: 'JetBrains Mono',
      provider: 'local',
      src: '/fonts/JetBrainsMono-Variable.woff2',
      weight: '400 700'
    }
  ],

  // Fallbacks pour calcul automatique des métriques
  fallbacks: {
    'sans-serif': ['Arial', 'Helvetica Neue'],
    'monospace': ['Menlo', 'Monaco']
  },

  // Désactiver les providers externes pour full SSG
  providers: {
    google: false,
    bunny: false
  }
}

// Configuration @nuxtjs/color-mode - Complète
colorMode: {
  preference: 'system',           // Valeur par défaut
  fallback: 'light',              // Fallback si système non détectable
  classSuffix: '',                // CRITIQUE: vide pour Tailwind (.dark)
  storage: 'localStorage',        // Persistence
  storageKey: 'nuxt-color-mode'   // Clé localStorage
}

// Configuration nuxt-security - CSP et headers de sécurité
// Note: Les headers HTTP sont principalement définis dans public/_headers pour CF Pages
// nuxt-security ajoute le support SSG (hash scripts) et la validation en dev
//
// ⚠️ ALERTE SÉCURITÉ - CVE critiques @nuxtjs/mdc (utilisé par Nuxt Content 3)
// - CVE-2025-24981 : Contournement XSS via encodage HTML des URLs JavaScript
// - CVE-2025-54075 : Injection balise <base> détournant toutes URLs relatives
// → Version minimale requise : @nuxtjs/mdc >= 0.17.2
// → Vérifier : npm list @nuxtjs/mdc && npm update @nuxtjs/mdc
//
security: {
  nonce: true,  // Requis pour dev mode
  sri: true,    // Subresource Integrity - protection contre compromission CDN/scripts
  ssg: {
    meta: true,           // CSP via <meta> tag fallback
    hashScripts: true,    // Auto-calcul des hashes scripts au build
    hashStyles: false,    // false car 'unsafe-inline' requis pour shadcn-vue
    exportToPresets: true // Tente export vers headers plateforme
  },
  headers: {
    contentSecurityPolicy: {
      'default-src': ["'self'"],
      'script-src': ["'self'", "'strict-dynamic'"],
      'style-src': ["'self'", "'unsafe-inline'"],  // ⚠️ Requis pour Reka UI
      'img-src': ["'self'", "data:", "blob:", "https:"],
      'font-src': ["'self'", "data:"],
      'connect-src': ["'self'"],
      'frame-ancestors': ["'none'"],
      'base-uri': ["'none'"],
      'form-action': ["'self'"],
      'object-src': ["'none'"],
      'upgrade-insecure-requests': true
    },
    crossOriginOpenerPolicy: 'same-origin-allow-popups',  // OAuth compatible
    crossOriginResourcePolicy: 'same-origin',
    referrerPolicy: 'strict-origin-when-cross-origin',
    strictTransportSecurity: {
      maxAge: 63072000,      // 2 ans
      includeSubdomains: true,
      preload: true
    },
    xContentTypeOptions: 'nosniff',
    xFrameOptions: 'DENY',
    xXSSProtection: '0',    // Désactivé (filtre navigateur déprécié)
    permissionsPolicy: {
      camera: [],
      microphone: [],
      geolocation: [],
      payment: [],
      usb: [],
      bluetooth: [],
      accelerometer: [],
      gyroscope: []
    }
  }
}

// Script anti-FOUC + meta color-scheme (CRITIQUE pour SSG)
// Élimine le flash de thème en appliquant la classe AVANT le premier rendu
app: {
  head: {
    meta: [
      { name: 'color-scheme', content: 'light dark' }  // Évite flash scrollbars/inputs
    ],
    script: [{
      // Script bloquant ~300 octets : préférence user → système → défaut
      children: `(function(){try{var t=localStorage.getItem('nuxt-color-mode');var s=window.matchMedia&&window.matchMedia('(prefers-color-scheme: dark)').matches?'dark':'light';var m=t&&t!=='system'?t:s;document.documentElement.classList.add(m);window.__NUXT_COLOR_MODE__=m}catch(e){}})();`,
      tagPosition: 'head'
    }]
  }
}

// Performance (remplace experimental.inlineSSRStyles)
features: {
  inlineStyles: true,   // CLS 0.77 → 0.00
}

// Transpilation obligatoire pour SSR (évite erreurs ES modules)
build: {
  transpile: [
    'reka-ui',           // "Cannot split a chunk" sans transpilation
    'vee-validate',      // "Unexpected Token: export" sans transpilation
    '@vee-validate/rules'
  ]
}

// Cloudflare Pages SSG - Configuration complète
nitro: {
  preset: 'cloudflare_pages',

  // Configuration Cloudflare spécifique
  cloudflare: {
    deployConfig: true,    // Génère wrangler.json automatiquement
    nodeCompat: true,      // Compatibilité Node.js
    pages: {
      routes: {
        // Optimise _routes.json (limite 100 routes Cloudflare)
        exclude: ['/blog/*', '/categories/*', '/tags/*']
      }
    }
  },

  // Configuration prerendering optimisée
  prerender: {
    autoSubfolderIndex: false, // Évite les doubles redirects Cloudflare
    crawlLinks: true,          // Découvre automatiquement les liens internes
    routes: [                  // Routes additionnelles à prérender
      '/',
      '/sitemap.xml',
      '/robots.txt',
      '/rss.xml'
    ],
    ignore: [                  // Routes à exclure
      '/api/**',
      '/admin/**',
      '/_nuxt/**'
    ],
    failOnError: false,        // Continue malgré les erreurs
    concurrency: 4,            // Prerendering parallèle
    retry: 3,                  // Tentatives en cas d'échec
    retryDelay: 500
  },

  // Compression assets (~15-25% réduction vs gzip seul)
  compressPublicAssets: {
    gzip: true,
    brotli: true,
  },
}

// Lazy loading composants MDC lourds
components: [
  { path: '~/components/content/heavy', isAsync: true },
  '~/components',
],

// Route Rules - Cache Cloudflare optimisé
routeRules: {
  // Cache statique agressif pour assets buildés par Nuxt (1 an, immutable)
  '/_nuxt/**': { headers: { 'cache-control': 'public, max-age=31536000, immutable' } },
  // Cache court pour HTML SSG (permet rollbacks rapides)
  '/**': { headers: { 'cache-control': 'public, max-age=0, must-revalidate' } },
},

// Nuxt Content 3 - D1 Database REQUISE sur Cloudflare Pages
content: {
  database: {
    type: 'd1',
    bindingName: 'DB'  // Binding configuré dans Cloudflare Dashboard
  },

  // Syntax highlighting Shiki - Configuration v3
  build: {
    markdown: {
      highlight: {
        theme: {
          default: 'github-light',      // Thème light (WCAG AA)
          dark: 'github-dark'           // Thème dark (auto via html.dark)
        },
        // Limiter les langages pour réduire le bundle
        langs: ['json', 'js', 'ts', 'html', 'css', 'vue', 'shell', 'yaml', 'md', 'mdc', 'bash', 'toml']
      }
    }
  }
}
```

**Configuration Cloudflare D1 (wrangler.toml) :**

```toml
# wrangler.toml
name = "sebc-dev"
compatibility_date = "2024-09-19"

[[d1_databases]]
binding = "DB"
database_name = "content-db"
database_id = "votre-database-id"  # Généré via: wrangler d1 create content-db
```

**Création de la base D1 :**

```bash
# Créer la base D1
wrangler d1 create content-db

# Récupérer le database_id affiché et l'ajouter dans wrangler.toml
```

## Architecture Dump/Restore D1

Nuxt Content 3 utilise une **architecture dump/restore** plutôt que des migrations traditionnelles :

```
Build Time                          Runtime (Cold Start)
    ↓                                      ↓
Parse contenu MDC                   Détection du dump
    ↓                                      ↓
Génère dump SQL                     Restauration dans D1
    ↓                                      ↓
Inclus dans .output/public          Tables _content_* créées
```

**Processus en 4 étapes :**
1. `nuxt build --preset=cloudflare_pages` parse le contenu et génère le dump
2. Le build produit `.output/public` avec le dump intégré
3. Déploiement via `wrangler pages deploy .output/public` ou Git push
4. Premier accès : Nuxt Content détecte le dump et le restaure dans D1

**Tables créées automatiquement :** `_content_info` et `_content_content` — aucune migration manuelle nécessaire.

## Limites D1 Free Tier (appliquées depuis 10 février 2025)

| Ressource | Free Tier | Workers Paid ($5/mois) |
|-----------|-----------|------------------------|
| **Rows read** | 5 millions/jour | 25 milliards/mois |
| **Rows written** | 100 000/jour | 50 millions/mois |
| **Stockage total** | 5 GB | 1 TB |
| **Taille max par DB** | **500 MB** | 10 GB |
| **Nombre de bases** | 10 | 50 000 |
| **Time Travel** | 7 jours | 30 jours |

**Ce qui compte dans le quota :**
- ✅ Requêtes depuis Workers/Pages déployés
- ✅ `wrangler d1 execute --remote`
- ✅ Requêtes via Dashboard Cloudflare
- ❌ Développement local (`wrangler dev` sans `--remote`)
- ❌ Build-time (D1 n'est pas accessible pendant le build)

**⚠️ Piège majeur** : D1 facture les **rows_read**, pas les rows_returned. Un `SELECT * FROM table` sur 5000 lignes sans index = 5000 rows_read même avec `LIMIT 1`.

## Indexes D1 pour Optimisation

Les indexes sont définis dans `content.config.ts` pour chaque collection (voir [Content Architecture](#content-architecture)).

**Stratégie d'indexation D1 :**

| Index | Colonnes | Usage | Type |
|-------|----------|-------|------|
| `path` | `path` | Lookup par URL | Unique |
| `pillar` | `pillar` | Filtrage par pilier | Simple |
| `publishedAt` | `publishedAt` | Tri chronologique | Simple |
| `draft` | `draft` | Filtrage publié/brouillon | Simple |
| `idx_published` | `(draft, publishedAt)` | Liste publiés triés par date | Composite |

**⚠️ Impact sur les coûts D1 :** Un `WHERE` sur colonne indexée = **1 row read** au lieu d'un table scan complet. L'index composite `idx_published` optimise la requête typique `WHERE draft = false ORDER BY publishedAt DESC` en une seule lecture d'index.

**Avantage collections séparées :** Chaque collection (`articles_fr`, `articles_en`) a ses propres indexes isolés, réduisant la taille des tables scannées.

**Notes importantes:**
- `@nuxtjs/tailwindcss@6.14.0` supporte TW4, mais `@tailwindcss/vite` recommandé pour nouveaux projets
- `nuxt-delay-hydration` obsolète → hydratation lazy native Nuxt 4 (hydrate-on-visible, etc.)
- `experimental.inlineSSRStyles` renommé en `features.inlineStyles`
- `nuxt-vitalizer` 2.0: DelayHydration component supprimé → macros natives
- **Fonts preload** : `crossorigin="anonymous"` obligatoire même en self-hosting. Utiliser `font-display: optional` pour CLS = 0 (voir `tailwindcss-patterns.md`)
- **MiniSearch** : Index généré via script `postgenerate` dans `package.json` :
  ```json
  "scripts": {
    "generate": "nuxt generate",
    "postgenerate": "node scripts/generate-search-index.mjs"
  }
  ```
- **Index MiniSearch** : Fichier `public/search-index.json` généré au build

## Code Highlighting (Shiki)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Configuration path** | `content.build.markdown.highlight` | ⚠️ `content.highlight` (v2) est **déprécié** |
| **Dual-theme** | `{ default: 'github-light', dark: 'github-dark' }` | Switch automatique via classe `html.dark` |
| **Langages** | Set minimal dans `langs` | Réduit le bundle (évite `shiki/bundle/full` 6.4MB) |
| **Accessibilité** | Thèmes high-contrast | Ratio contraste ≥4.5:1 (WCAG AA) |

### Thèmes recommandés

| Mode | Thème standard | Thème high-contrast | Alternatifs |
|------|----------------|---------------------|-------------|
| Light | `github-light` | `github-light-high-contrast` | `min-light`, `vitesse-light` |
| Dark | `github-dark` | `github-dark-high-contrast` | `nord`, `one-dark-pro` |

**Paires équilibrées (WCAG AA - ratio ≥4.5:1) :**
- `github-light` / `github-dark` — Défaut recommandé, excellent contraste
- `github-light-high-contrast` / `github-dark-high-contrast` — Ratio ≥5.5:1, accessibilité maximale
- `min-light` / `min-dark` — Minimaliste, très lisible
- `vitesse-light` / `vitesse-dark` — Design épuré

**Critères de sélection :**
- Privilégier les thèmes `*-high-contrast` pour le contenu technique
- Tester le contraste des commentaires (souvent problématique)
- Utiliser `colorReplacements` pour corriger des couleurs spécifiques

⚠️ **Thème `css-variables` cassé** depuis Content 2.8.5+ (migration Shikiji). Utiliser les thèmes nommés.

### Correction couleurs low-contrast

Pour corriger des couleurs problématiques dans un thème :

```typescript
// nuxt.config.ts
content: {
  build: {
    markdown: {
      highlight: {
        theme: { default: 'min-light', dark: 'min-dark' },
        colorReplacements: {
          '#ff79c6': '#c678dd',  // Remplace couleur trop claire
          '#6272a4': '#5c6370'   // Améliore contraste commentaires
        }
      }
    }
  }
}
```

### Impact bundle size

| Bundle | Taille min | Taille gzip |
|--------|------------|-------------|
| `shiki/bundle/full` | 6.4 MB | 1.2 MB |
| `shiki/bundle/web` | 3.8 MB | 695 KB |
| Fine-grained (core) | ~12 KB | + thèmes/langages |

**Recommandation :** Nuxt Content utilise l'approche fine-grained via `@nuxtjs/mdc`. Limiter `langs` au strict nécessaire :

```typescript
langs: ['json', 'js', 'ts', 'html', 'css', 'vue', 'shell', 'yaml', 'md', 'mdc']
```

## Validation Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Schema Validator** | Zod 4 ou Valibot v1 | Standard Schema natif; Valibot ~1KB gzip / ~2KB min, Zod ~5KB gzip / ~10KB min, zod/mini ~2KB gzip / ~5KB min (réduction 64%) |
| **Import Source** | `import { z } from 'zod/v4'` | ⚠️ Utiliser `zod/v4` pour l'API Zod 4. `@zod/mini` (package npm) **déplacé** vers `zod/mini` (import path) depuis mai 2025 |
| **Validation Scope** | Build time | Erreurs détectées au build via Content 3 collections |
| **Type Inference** | `z.infer<typeof schema>` | Types TypeScript générés automatiquement |
| **Sitemap Integration** | `asSitemapCollection()` | Wrapper requis pour @nuxtjs/sitemap v7.5+ avec Content v3 |
| **SEO Unified** | `asSeoCollection()` | Combine sitemap + schema.org + OG Image + Robots via `@nuxtjs/seo/content` |

### Top-Level Validators Zod 4

Zod 4 promeut les validateurs de format au niveau top-level pour un meilleur tree-shaking et des performances accrues :

| Zod 3 (déprécié) | Zod 4 (recommandé) | Notes |
|------------------|-------------------|-------|
| `z.string().email()` | `z.email()` | Supporte patterns : `z.email({ pattern: z.regexes.unicodeEmail })` |
| `z.string().url()` | `z.url()` | Validation WHATWG URL |
| `z.string().uuid()` | `z.uuid()` | ⚠️ Strict RFC 9562/4122 (variant bits) |
| `z.string().uuid()` | `z.guid()` | Validation permissive (UUIDs legacy) |
| `z.date()` | `z.iso.date()` | Format `YYYY-MM-DD` (string, compatible JSON Schema) |
| `z.date()` | `z.iso.datetime()` | Format ISO 8601 complet avec options |
| `z.string().ip()` | `z.ipv4()` / `z.ipv6()` | `z.string().ip()` supprimé en Zod 4 |

**Options `z.iso.datetime()` :**
```typescript
z.iso.datetime({ offset: true })     // Autorise timezone offsets
z.iso.datetime({ local: true })      // Permet datetimes sans timezone
z.iso.datetime({ precision: 3 })     // Contraint à millisecondes
```

**⚠️ Migration UUID importante :** La validation UUID Zod 4 est stricte (RFC 9562/4122). Les UUIDs qui passaient en Zod 3 peuvent échouer. Utiliser `z.guid()` pour une validation permissive.

### Breaking Changes Critiques Zod 4

#### `.default()` dans `.optional()` - Comportement inversé

**⚠️ CRITIQUE** : En Zod 4, `.default()` s'applique maintenant DANS les champs `.optional()` :

```typescript
const schema = z.object({
  draft: z.string().default("untitled").optional()
})

schema.parse({})
// Zod 3: {}                        ← undefined préservé
// Zod 4: { draft: "untitled" }     ← default appliqué !
```

**Impact frontmatter** : Les champs optionnels avec defaults auront toujours une valeur en Zod 4.

#### `.prefault()` - Defaults pré-transform

Nouveau en Zod 4, applique le default AVANT les transforms (comportement Zod 3) :

```typescript
const titleSchema = z.string()
  .transform(val => val.toUpperCase())
  .prefault('untitled')  // Default sur le type INPUT

titleSchema.parse(undefined) // => "UNTITLED"
```

| Méthode | Appliqué | Type cible |
|---------|----------|------------|
| `.default(x)` | Après transform | Output type |
| `.prefault(x)` | Avant transform | Input type |

#### `.nullish()` pour YAML null

Support des valeurs `null` explicites en YAML frontmatter :

```typescript
// Accepte: undefined, null, ou string URL
coverImage: z.string().url().nullish().default(null)

// Hiérarchie complète
| Pattern | Accepte | Output |
|---------|---------|--------|
| `.optional()` | `T \| undefined` | `T \| undefined` |
| `.nullable()` | `T \| null` | `T \| null` |
| `.nullish()` | `T \| null \| undefined` | `T \| null \| undefined` |
| `.nullish().default(x)` | `T \| null \| undefined` | `T` |
```

#### Enum `.extract()` et `.exclude()`

Créer des sous-ensembles d'enums sans redéfinition :

```typescript
const AllCategories = z.enum(['tutorial', 'news', 'opinion', 'review'])
const BlogOnly = AllCategories.extract(['tutorial', 'opinion'])  // Sous-ensemble
const NonNews = AllCategories.exclude(['news'])                   // Exclusion

// Accès programmatique aux valeurs (remplace .Enum et .Values)
AllCategories.enum // => { tutorial: "tutorial", news: "news", ... }
```

#### Autres breaking changes API

| Zod 3 | Zod 4 | Notes |
|-------|-------|-------|
| `z.record(z.string())` | `z.record(z.string(), z.string())` | **Deux arguments requis** |
| `.merge(otherSchema)` | `.extend(otherSchema.shape)` | `.merge()` déprécié |
| `.format()` / `.flatten()` | `z.treeifyError()` | API erreurs changée |
| `.nonempty()` → `[T, ...T[]]` | `.nonempty()` → `T[]` | Type inference simplifié |
| `z.nativeEnum(TsEnum)` | `z.enum(TsEnum)` | Enums TS natifs supportés |

### asSeoCollection() vs asSitemapCollection()

| Wrapper | Fonctionnalités | Quand utiliser |
|---------|-----------------|----------------|
| `asSitemapCollection()` | Sitemap uniquement | Blog simple sans besoins SEO avancés |
| `asSeoCollection()` | Sitemap + Schema.org + OG Image + Robots | **Recommandé** - SEO complet |

**Configuration asSeoCollection() :**

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { asSeoCollection } from '@nuxtjs/seo/content'
import { z } from 'zod/v4'

export default defineContentConfig({
  collections: {
    articles_fr: defineCollection(
      asSeoCollection({  // ← Wrapper SEO complet
        type: 'page',
        source: { include: 'fr/**/*.md', prefix: '/blog' },
        schema: articleSchema,
        indexes: articleIndexes,
      })
    ),
  }
})
```

**Clés frontmatter SEO activées par asSeoCollection() :**

```yaml
---
title: "Mon article"
description: "Description SEO (max 160 caractères)"
image: "/images/cover.jpg"

# OG Image dynamique (optionnel)
ogImage:
  component: BlogOgImage
  props:
    title: "Mon article"
    readingMins: 5

# Schema.org structuré (optionnel)
schemaOrg:
  - "@type": "BlogPosting"
    headline: "Mon article"
    author:
      "@type": "Person"
      name: "Sébastien C."
    datePublished: "2025-01-15"

# Contrôle sitemap (optionnel)
sitemap:
  lastmod: 2025-01-16
  priority: 0.8

# Contrôle robots (optionnel)
robots:
  noindex: false
  nofollow: false
---
```

### Schema SEO imbriqué (optionnel)

Pour un contrôle granulaire du SEO dans le frontmatter, définir un schema dédié :

```typescript
// content.config.ts
const seoSchema = z.object({
  title: z.string().max(60).optional(),      // Titre OG/Twitter
  description: z.string().max(160).optional(), // Meta description
  image: z.string().optional(),               // OG image path
  canonical: z.string().url().optional(),     // URL canonique
  noIndex: z.boolean().default(false),        // Bloquer indexation
})

const articleSchema = z.object({
  title: z.string().min(1).max(100),
  description: z.string().max(300).optional(),
  // ... autres champs
  seo: seoSchema.optional(),  // ← SEO imbriqué
})
```

**Usage dans le frontmatter :**

```yaml
---
title: "Titre de l'article"
description: "Description générale"
seo:
  title: "Titre SEO optimisé (60 car.)"  # Override pour les moteurs
  description: "Meta description (160 car.)"
  canonical: "https://sebc.dev/blog/article-original"
  noIndex: false
---
```

## Search Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Search Engine** | MiniSearch 7.x | ~7KB minified, zéro dépendance, API flexible |
| **Indexation** | Script build-time + JSON statique | Index pré-généré dans `public/search-index.json` |
| **Loading** | Index complet au premier accès | Fetch lazy sur ouverture search (⌘K) |
| **UX** | Command palette | Intégré avec shadcn-vue Command component (⌘K) |

**Avantages MiniSearch vs Pagefind :**
- Bundle plus léger (~7KB vs ~8KB + chunks externes)
- Pas de fichiers externes à charger dynamiquement
- Boosting configurable par champ (`title: 2`, `content: 1`)
- Fuzzy search et prefix search natifs
- Stemming FR via option `stemmer`

## @nuxt/content v3 API Changes

**Composants supprimés (migration v2 → v3) :**
- ❌ `<ContentDoc />` - supprimé
- ❌ `<ContentList />` - supprimé
- ❌ `<ContentQuery />` - supprimé
- ✅ Utiliser `<ContentRenderer>` pour tout le rendu

**API de requête :**
```typescript
// ❌ ANCIEN (Content v2)
const posts = await queryContent('blog').find()

// ✅ NOUVEAU (Content v3)
const posts = await queryCollection('blog').all()
```

**Autres changements :**
- Mode document-driven supprimé - créer les pages manuellement
- Composants prose personnalisés dans `components/prose/` (non plus `components/content/`)
- Index de recherche MiniSearch : `public/search-index.json`

## APIs Utilitaires @nuxt/content v3.10+

La version 3.10+ introduit des APIs supplémentaires pour les cas d'usage courants :

```typescript
import type { Collections } from '@nuxt/content'

const { locale } = useI18n()
const collection = `articles_${locale.value}` as keyof Collections

// Compter les résultats (évite de charger toutes les données)
const count = await queryCollection(collection)
  .where('draft', '=', false)
  .count()

// Navigation prev/next pour article courant
const [prev, next] = await queryCollectionItemSurroundings(
  collection,
  route.path,
  { fields: ['title', 'path', 'publishedAt'] }
)
```

**Cas d'usage :**

| API | Usage | Alternative |
|-----|-------|-------------|
| `count()` | Afficher "X articles" sans charger les données | `all().length` (moins performant) |
| `queryCollectionItemSurroundings()` | Navigation prev/next article | Requête manuelle avec `order()` |

**Note :** `queryCollectionSearchSections()` existe mais **non recommandé** pour ce projet - MiniSearch (index pré-généré) est plus performant et ne consomme pas le quota D1.

## SSG Build Performance

### Benchmark temps de build

Nuxt Content v3 affiche une complexité **O(n²)** pour les builds SSG avec de grands volumes de contenu (benchmarks communautaires) :

| Documents | Build Time approx | Recommandation |
|-----------|-------------------|----------------|
| 100-500 | ~30s - 1 min | ✅ Optimal |
| 500-1,000 | ~1-2 min | ✅ Acceptable |
| 1,000-2,000 | ~2-3 min | ⚠️ Surveiller |
| 2,000-5,000 | ~3-6 min | ⚠️ Considérer split |
| 5,000-10,000 | ~6-12 min | ❌ Split collections |

**Stratégies pour grands volumes :**
- Diviser les collections (par année, par pilier, etc.)
- Utiliser `crawlLinks: true` avec parcimonie
- Réduire les transformations MDC complexes

### Optimisation prerender partagé

Activer le partage de données entre routes prérendues pour réduire les requêtes redondantes :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  experimental: {
    // Partage les données entre pages prérendues (Nuxt 3.10+)
    sharedPrerenderData: true,

    // Extraction des payloads pour mise en cache
    payloadExtraction: true
  }
})
```

**Impact :**
- Réduit les requêtes `queryCollection` dupliquées entre pages
- Les données communes (listes, navigation) sont calculées une seule fois
- Particulièrement efficace pour les blogs avec sidebars et navigation partagées

**Exemple de gain :**
| Configuration | Requêtes totales | Build time |
|---------------|------------------|------------|
| Sans optimisation | 500 requêtes | 45s |
| `sharedPrerenderData: true` | 180 requêtes | 28s |

## Hydratation Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Méthode** | Hydratation lazy native Nuxt 4 | Remplace nuxt-delay-hydration (obsolète) |
| **Composants lourds** | `hydrate-on-visible` | Hydrate quand visible dans viewport |
| **Composants interactifs** | `hydrate-on-interaction` | Hydrate sur hover/click/focus |
| **Composants différés** | `hydrate-on-idle` | Hydrate pendant idle time navigateur |
| **Composants statiques** | `hydrate-never` | Jamais hydraté - HTML statique uniquement |
| **Délai fixe** | `:hydrate-after="3000"` | Hydrate après délai en ms |
| **Conditionnel** | `:hydrate-when="condition"` | Hydrate selon condition booléenne |
| **Media query** | `hydrate-on-media-query="(max-width: 768px)"` | Hydrate selon media query |

## Performance Monitoring

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

### Lighthouse CI Configuration

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

### Plugin web-vitals Debug (Développement)

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

## Breaking Changes Nuxt 3 → Nuxt 4

| Aspect | Nuxt 3 (ancien) | Nuxt 4 (actuel) |
|--------|-----------------|-----------------|
| **Structure** | `pages/`, `components/` | `app/pages/`, `app/components/` |
| **Generate config** | `generate: {}` | `nitro: { prerender: {} }` |
| **Output** | `dist/` | `.output/public/` |
| **Exclude routes** | `generate.exclude` | `nitro.prerender.ignore` |
| **TypeScript** | Single tsconfig | Project references |
| **Compatibility** | N/A | `compatibilityDate` requis |
| **SSR Styles** | `experimental.inlineSSRStyles` | `features.inlineStyles` |
| **SEO Server-only** | `useServerSeoMeta()` | `if (import.meta.server) { useSeoMeta() }` |

### Migration useServerSeoMeta() (SEO)

```typescript
// ❌ ANCIEN (Nuxt 3) - Déprécié
useServerSeoMeta({
  description: 'Ma description'
})

// ✅ NOUVEAU (Nuxt 4) - Utiliser import.meta.server
if (import.meta.server) {
  useSeoMeta({
    description: 'Ma description'
  })
}
```

**Cas d'usage** : Meta tags statiques qui n'ont pas besoin d'être recalculés côté client (robots, ogSiteName, twitterCard par défaut). Voir `seo-patterns.md` pour les patterns complets.

**Exemples migration generate → nitro.prerender :**

```typescript
// ❌ Nuxt 3 (obsolète dans Nuxt 4)
export default defineNuxtConfig({
  generate: {
    exclude: ['/admin'],
    routes: ['/sitemap.xml']
  }
})

// ✅ Nuxt 4 (correct)
export default defineNuxtConfig({
  nitro: {
    prerender: {
      ignore: ['/admin'],        // Remplace generate.exclude
      routes: ['/sitemap.xml']   // Remplace generate.routes
    }
  }
})
```

## Breaking Changes @nuxtjs/i18n v9 → v10

| Aspect | v9 (ancien) | v10 (actuel) |
|--------|-------------|--------------|
| **Lazy loading** | `lazy: true` optionnel | Toujours lazy (défaut) |
| **`redirectOn: 'root'`** | Redirige toutes pages sans préfixe | Redirige **uniquement** `/` |
| **Hook changement langue** | `onLanguageSwitched()` | `i18n:localeSwitched` hook |
| **Vue I18n** | v9 | v11 |
| **Nitro detection** | Non disponible | Language detection ajoutée |
| **Locale field** | `iso` | `language` (pour hreflang) |

**Migration `redirectOn` :**

```typescript
// v9 - Ce comportement redirigeait TOUTES les pages
detectBrowserLanguage: {
  redirectOn: 'root'  // → Redirige /about, /blog, etc.
}

// v10 - Même config, comportement différent
detectBrowserLanguage: {
  redirectOn: 'root'  // → Redirige UNIQUEMENT /
}

// v10 - Pour retrouver le comportement v9
detectBrowserLanguage: {
  redirectOn: 'all'  // → Redirige toutes les pages
}
```

**Migration hook :**

```typescript
// ❌ v9 (obsolète)
onLanguageSwitched((oldLocale, newLocale) => {
  console.log(oldLocale, newLocale)
})

// ✅ v10 (correct)
nuxtApp.hook('i18n:localeSwitched', ({ oldLocale, newLocale }) => {
  console.log(oldLocale, newLocale)
})
```

**Migration locales config :**

```typescript
// ❌ v9 - iso pour hreflang
locales: [
  { code: 'en', iso: 'en-US', name: 'English' }
]

// ✅ v10 - language remplace iso + isCatchallLocale pour x-default
locales: [
  {
    code: 'fr',
    language: 'fr-FR',
    name: 'Français',
    file: 'fr.json',
    isCatchallLocale: true  // Génère hreflang="x-default" pour FR
  },
  { code: 'en', language: 'en-US', name: 'English', file: 'en.json' }
]
```

**Configuration `isCatchallLocale`** : Définit quelle locale génère le tag `hreflang="x-default"`. Utilisé par les moteurs de recherche pour les visiteurs dont la langue n'est pas supportée.

## Optimisations Bundle i18n (Tree-shaking)

Options de bundle pour réduire la taille du bundle client :

```typescript
// nuxt.config.ts
i18n: {
  bundle: {
    compositionOnly: true,  // Tree-shake Legacy API ($t, $tc, etc. dans Options API)
    runtimeOnly: true       // Pas de compilateur dans le bundle (JSON uniquement)
  }
}
```

| Option | Effet | Économie |
|--------|-------|----------|
| `compositionOnly: true` | Supprime le support Options API | ~30% taille vue-i18n |
| `runtimeOnly: true` | Supprime le compilateur de messages | ~20% taille vue-i18n |

**⚠️ `runtimeOnly: true` requis** : Uniquement si vos traductions sont en **JSON pur**. Si vous utilisez des messages inline dans le code (`$t('Hello {name}')` avec interpolation dynamique), gardez `runtimeOnly: false`.

**Vérification** : Les fichiers dans `i18n/locales/*.json` doivent contenir uniquement du JSON statique :

```json
// ✅ Compatible runtimeOnly: true
{ "greeting": "Bonjour {name}" }

// ❌ Incompatible (nécessite compilateur)
// Messages avec syntaxe ICU complexe ou fonctions
```
