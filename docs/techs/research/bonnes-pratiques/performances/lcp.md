# Optimisation LCP et Core Web Vitals pour Nuxt 4.2.x SSG sur Cloudflare Pages

L'optimisation du Largest Contentful Paint pour un blog Nuxt 4.2.x déployé en SSG sur Cloudflare Pages repose sur quatre piliers : **preload stratégique de l'image LCP avec fetchpriority**, **formats modernes AVIF/WebP**, **CSS critique inline**, et **fonts optimisées avec fallbacks métriques**. Cette stack peut atteindre un score Lighthouse de **95+** et un LCP sous **2.5s** en suivant les pratiques détaillées ci-dessous — le tout gratuitement avec les limitations de Cloudflare Pages (500 builds/mois, 20 000 fichiers max).

---

## Configuration NuxtImg pour une image LCP performante

L'élément LCP sur un blog est généralement l'image hero ou la featured image d'un article. La combinaison **preload + fetchpriority="high" + loading="eager"** est essentielle pour signaler au navigateur la criticité de cette ressource.

### Configuration complète du module @nuxt/image

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  future: { compatibilityVersion: 4 },
  
  modules: ['@nuxt/image'],
  
  image: {
    // Formats par défaut pour NuxtPicture (ordre = priorité)
    format: ['avif', 'webp'],
    quality: 80,
    
    // Correspondance avec les breakpoints TailwindCSS 4
    screens: {
      sm: 640,
      md: 768,
      lg: 1024,
      xl: 1280,
      '2xl': 1536
    },
    
    // Presets réutilisables
    presets: {
      hero: {
        modifiers: {
          format: 'webp',
          quality: 85,
          fit: 'cover'
        }
      },
      thumbnail: {
        modifiers: {
          format: 'webp',
          quality: 70,
          width: 400,
          height: 300
        }
      }
    }
  }
})
```

### Template pour l'image LCP avec NuxtPicture

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
      alt: 'Bannière principale du blog',
      class: 'w-full h-auto object-cover'
    }"
  />
</template>
```

Le HTML généré en SSG inclut automatiquement le `<link rel="preload">` dans le `<head>` et un élément `<picture>` avec sources AVIF et WebP :

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

### Différences entre preload, eager et fetchpriority

| Attribut | Effet | Cas d'usage |
|----------|-------|-------------|
| `preload` | Génère `<link rel="preload">` dans `<head>` | Image LCP uniquement (1 par page) |
| `loading="eager"` | Désactive le lazy loading natif | Toute image above-the-fold |
| `fetchpriority="high"` | Priorise dans la file réseau | Image critique (LCP, logo header) |

**Combinaison optimale pour LCP** : les trois ensemble. Pour les images below-the-fold : `loading="lazy"` + `fetchpriority="low"` sans preload.

---

## Formats WebP et AVIF avec responsive images

Le provider **ipxStatic** (automatique en SSG) génère les variantes d'images au build. Pour Cloudflare Pages, aucune configuration serveur n'est nécessaire.

### Configuration des sources responsives

```vue
<!-- Responsive image avec srcset automatique -->
<NuxtImg
  src="/images/article-cover.jpg"
  sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"
  width="800"
  height="600"
  format="webp"
  loading="lazy"
  fetch-priority="low"
  :alt="article.title"
/>
```

Le système génère automatiquement :

```html
<img 
  src="/_ipx/f_webp,w_800/article-cover.jpg"
  srcset="/_ipx/f_webp,w_640/article-cover.jpg 640w,
          /_ipx/f_webp,w_768/article-cover.jpg 768w,
          /_ipx/f_webp,w_1024/article-cover.jpg 1024w"
  sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"
  loading="lazy"
  fetchpriority="low"
>
```

### Images à densité fixe (avatars, logos)

```vue
<NuxtImg
  src="/images/avatar.png"
  width="48"
  height="48"
  densities="x1 x2"
  format="webp"
  alt="Avatar utilisateur"
/>
```

Génère `srcset` avec variantes 1x et 2x pour écrans Retina.

---

## Critical CSS inline avec TailwindCSS 4.1.x

TailwindCSS 4.x introduit une **configuration CSS-native** via `@theme` — plus de `tailwind.config.js` par défaut. Nuxt 4 modifie également le comportement d'inlining CSS : seul le CSS des composants Vue est inliné, le CSS global reste en fichier externe cacheable.

### Configuration TailwindCSS 4.x CSS-native

```css
/* assets/css/tailwind.css */
@import "tailwindcss";

@theme {
  /* Tokens de design */
  --color-primary: oklch(0.65 0.19 240);
  --color-secondary: oklch(0.75 0.12 180);
  
  /* Typographie */
  --font-sans: "Inter", system-ui, sans-serif;
  --font-display: "Satoshi", sans-serif;
  
  /* Breakpoints supplémentaires */
  --breakpoint-xs: 475px;
  --breakpoint-3xl: 1920px;
}

/* Utilitaires custom */
@utility prose-blog {
  max-width: 65ch;
  margin-inline: auto;
}
```

### Intégration Nuxt 4 + TailwindCSS 4 + Critters

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  future: { compatibilityVersion: 4 },
  
  css: ['~/assets/css/tailwind.css'],
  
  // TailwindCSS via plugin Vite (plus performant que PostCSS)
  vite: {
    plugins: [
      (await import('@tailwindcss/vite')).default()
    ],
    build: {
      cssMinify: 'lightningcss'
    }
  },
  
  // Critical CSS extraction
  modules: ['@nuxtjs/critters'],
  
  critters: {
    config: {
      preload: 'swap',      // Charge le CSS restant en async
      inlineFonts: false,   // Ne pas inliner les @font-face
      pruneSource: false    // Garder le CSS complet pour le chargement async
    }
  },
  
  // Comportement CSS Nuxt 4 (défaut)
  features: {
    inlineStyles: false  // CSS composants inline, global externe
  },
  
  // SSG
  nitro: {
    preset: 'cloudflare_pages_static',
    prerender: {
      crawlLinks: true,
      routes: ['/']
    }
  }
})
```

**@nuxtjs/critters** scanne le HTML pré-rendu, extrait le CSS above-the-fold, l'inline dans `<style>`, et charge le reste via `<link rel="preload" as="style">`. C'est la solution gratuite recommandée pour SSG.

### Tree-shaking automatique TailwindCSS 4

Le moteur JIT de TailwindCSS 4 ne génère **que les classes utilisées** dans vos templates — aucune configuration de purge nécessaire. Pour inclure des packages externes :

```css
@import "tailwindcss";
@source "../node_modules/@votrelib/ui";
```

---

## Configuration @nuxt/fonts pour performance typographique

Le module **@nuxt/fonts** intègre **fontaine** et **capsize** pour générer automatiquement des fallbacks avec métriques ajustées, éliminant le CLS causé par le swap de polices.

### Configuration optimale pour SSG

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/fonts'],
  
  fonts: {
    // Poids par défaut (éviter de tout charger)
    defaults: {
      weights: [400, 600, 700],
      styles: ['normal'],
      subsets: ['latin', 'latin-ext']
    },
    
    families: [
      // Self-hosting pour SSG (recommandé)
      {
        name: 'Inter',
        provider: 'local',
        src: '/fonts/Inter-Variable.woff2',
        weight: '400 700'
      },
      // Ou Google Fonts avec téléchargement automatique
      {
        name: 'Satoshi',
        provider: 'fontshare'  // Alternative sans tracking
      }
    ],
    
    // Fallbacks pour calcul des métriques
    fallbacks: {
      'sans-serif': ['Arial', 'Helvetica Neue']
    },
    
    // Désactiver les providers externes pour full SSG
    providers: {
      google: false,
      bunny: true  // Alternative GDPR-friendly
    }
  }
})
```

### Preload manuel des polices critiques

Pour les polices hébergées localement, ajoutez un preload explicite :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  app: {
    head: {
      link: [
        {
          rel: 'preload',
          href: '/fonts/Inter-Variable.woff2',
          as: 'font',
          type: 'font/woff2',
          crossorigin: 'anonymous'
        }
      ]
    }
  }
})
```

### Choix de font-display

| Valeur | Effet | Recommandation |
|--------|-------|----------------|
| `swap` | Texte visible immédiatement, swap à chargement | **Standard blog** — bon LCP, CLS possible |
| `optional` | Texte visible, web font seulement si chargée en <100ms | **Performance max** — zéro CLS |
| `fallback` | Block 100ms, swap jusqu'à 3s | Compromis |

**@nuxt/fonts utilise `swap` par défaut** avec des fallbacks métriques auto-générés pour minimiser le CLS. Ajoutez toujours un `line-height` explicite :

```css
body {
  font-family: 'Inter', 'Inter fallback', sans-serif;
  line-height: 1.6; /* Prévient le CLS */
}
```

---

## Optimisations Cloudflare Pages edge

Cloudflare Pages offre **Early Hints (103)**, compression Brotli automatique, et caching edge performant — le tout inclus gratuitement.

### Fichier _headers optimal

Créez `public/_headers` :

```
# Assets hashés (JS/CSS générés par Vite) - cache immutable 1 an
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

# Images optimisées - cache 1 mois
/_ipx/*
  Cache-Control: public, max-age=2592000

# Polices - cache 1 an
/fonts/*
  Cache-Control: public, max-age=31536000, immutable

# Early Hints pour ressources critiques
/*
  Link: </fonts/Inter-Variable.woff2>; rel=preload; as=font; crossorigin
  Link: <https://fonts.gstatic.com>; rel=preconnect

# Bloquer indexation du domaine pages.dev (SEO)
https://:project.pages.dev/*
  X-Robots-Tag: noindex

# Headers sécurité
/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: SAMEORIGIN
  Referrer-Policy: strict-origin-when-cross-origin
```

### Early Hints automatiques

Cloudflare Pages **génère automatiquement des Early Hints** à partir des `<link rel="preload">` dans votre HTML. Les `<link>` générés par NuxtImg avec `preload` sont donc automatiquement envoyés en 103 Early Hints.

**Important** : les `<link>` avec attributs supplémentaires (`fetchpriority`, `data-*`) ne sont PAS convertis — ce qui préserve votre priorisation custom.

### Fichier _redirects

Créez `public/_redirects` si nécessaire :

```
# Redirection SEO ancien vers nouveau
/blog/ancien-slug /articles/nouveau-slug 301

# SPA fallback (si pages dynamiques)
/* /index.html 200
```

### Contraintes du tier gratuit

| Ressource | Limite |
|-----------|--------|
| Builds | 500/mois |
| Fichiers par site | 20 000 |
| Taille fichier | 25 MB max |
| Redirects dynamiques | 100 |
| Headers rules | 100 |
| Bandwidth | **Illimité** |

---

## Mesure et debugging LCP

### Identification de l'élément LCP

1. **Chrome DevTools → Performance** : La section Live Metrics affiche directement l'élément LCP avec un lien cliquable vers le DOM
2. **Survol du métrique LCP** : Affiche le breakdown en 4 phases (TTFB, Load Delay, Load Duration, Render Delay)

### Intégration web-vitals dans Nuxt

```typescript
// plugins/web-vitals.client.ts
import { onLCP, onINP, onCLS } from 'web-vitals/attribution'

export default defineNuxtPlugin(() => {
  const sendMetric = (metric: any) => {
    const data = {
      name: metric.name,
      value: metric.value,
      rating: metric.rating,
      delta: metric.delta,
      id: metric.id,
      attribution: metric.attribution,
      url: window.location.href
    }
    
    // Envoi via sendBeacon (fiable même sur fermeture page)
    if (navigator.sendBeacon) {
      navigator.sendBeacon('/api/vitals', JSON.stringify(data))
    }
    
    // Debug en développement
    if (import.meta.dev) {
      console.log(`[${metric.name}]`, metric.value, metric.rating, metric.attribution)
    }
  }

  onLCP(sendMetric)
  onINP(sendMetric)
  onCLS(sendMetric)
})
```

### Diagnostic des sous-parties LCP

| Phase | Cible | Problèmes courants |
|-------|-------|--------------------|
| **TTFB** | <40% du LCP | Serveur lent, redirections, pas de CDN |
| **Resource Load Delay** | <10% | Image découverte tard, JS-dependent |
| **Resource Load Duration** | ~40% | Image trop lourde, pas de CDN |
| **Element Render Delay** | <10% | CSS/JS bloquant, main thread occupé |

### Outils RUM gratuits

- **Chrome User Experience Report (CrUX)** : Données terrain agrégées 28 jours
- **PageSpeed Insights API** : 25 000 requêtes/jour gratuites
- **DIY avec web-vitals** : Collecte vers votre propre endpoint (voir plugin ci-dessus)

---

## Anti-patterns à éviter absolument

### ❌ Images

```vue
<!-- WRONG: Lazy loading sur image LCP -->
<NuxtImg src="/hero.jpg" loading="lazy" />

<!-- WRONG: Dimensions manquantes (cause CLS) -->
<NuxtImg src="/hero.jpg" />

<!-- WRONG: Preload sur plusieurs images -->
<NuxtImg src="/img1.jpg" preload />
<NuxtImg src="/img2.jpg" preload />
<NuxtImg src="/img3.jpg" preload />
```

### ❌ CSS

```css
/* WRONG: Syntaxe TailwindCSS 3 dans v4 */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* WRONG: @config après @import */
@import "tailwindcss";
@config "../tailwind.config.js";
```

### ❌ Fonts

```typescript
// WRONG: Charger tous les poids
fonts: {
  defaults: {
    weights: [100, 200, 300, 400, 500, 600, 700, 800, 900]
  }
}
```

### ❌ Cloudflare

```
# WRONG: Cache agressif sur HTML (contenu périmé après déploiement)
/*.html
  Cache-Control: public, max-age=86400

# WRONG: Trop de preloads Early Hints (>50 nuit aux mobiles)
/*
  Link: </asset1.js>; rel=preload; as=script
  Link: </asset2.js>; rel=preload; as=script
  [...50+ liens...]
```

---

## Checklist Claude Code pour intégration

```markdown
## Checklist LCP Nuxt 4.2.x + Cloudflare Pages

### Configuration initiale
- [ ] `@nuxt/image` installé et configuré avec formats `['avif', 'webp']`
- [ ] `@nuxt/fonts` avec provider local pour polices self-hosted
- [ ] `@nuxtjs/critters` pour extraction CSS critique
- [ ] TailwindCSS 4.x via `@tailwindcss/vite` plugin
- [ ] `nuxt.config.ts` avec `nitro.preset: 'cloudflare_pages_static'`

### Image LCP (1 par page)
- [ ] Utiliser `<NuxtPicture>` avec `format="avif,webp"`
- [ ] Ajouter `:preload="{ fetchPriority: 'high' }"`
- [ ] Définir `loading="eager"`
- [ ] Spécifier `width` et `height` explicites
- [ ] Configurer `sizes` pour responsive

### Images below-the-fold
- [ ] `loading="lazy"` sur toutes les images non-LCP
- [ ] `fetch-priority="low"` pour images décoratives
- [ ] Pas de `preload`

### Fonts
- [ ] Self-hosting WOFF2 dans `public/fonts/`
- [ ] Preload de la police principale uniquement
- [ ] `line-height` explicite sur `body`
- [ ] Subsets limités (`latin`, `latin-ext`)

### Cloudflare Pages
- [ ] Fichier `public/_headers` créé
- [ ] Cache immutable pour `/_nuxt/*`
- [ ] `X-Robots-Tag: noindex` sur `*.pages.dev`
- [ ] Early Hints Link headers configurés

### Monitoring
- [ ] Plugin `web-vitals.client.ts` en place
- [ ] Tests PageSpeed Insights mobile < 2.5s LCP
- [ ] Vérification CrUX après 28 jours de trafic

### Validation build SSG
- [ ] `npx nuxi generate` sans erreurs
- [ ] Images optimisées dans `.output/public/_ipx/`
- [ ] Taille bundle CSS < 50KB gzipped
- [ ] Pas de console errors Core Web Vitals
```

---

## Configuration complète de référence

```typescript
// nuxt.config.ts — Configuration production optimisée
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  future: { compatibilityVersion: 4 },
  srcDir: 'app',
  
  modules: [
    '@nuxt/image',
    '@nuxt/fonts', 
    '@nuxtjs/critters'
  ],
  
  css: ['~/assets/css/tailwind.css'],
  
  vite: {
    plugins: [tailwindcss()],
    build: { cssMinify: 'lightningcss' }
  },
  
  image: {
    format: ['avif', 'webp'],
    quality: 80,
    screens: { sm: 640, md: 768, lg: 1024, xl: 1280, '2xl': 1536 }
  },
  
  fonts: {
    defaults: { weights: [400, 600, 700], subsets: ['latin'] },
    families: [
      { name: 'Inter', provider: 'local', src: '/fonts/Inter-Variable.woff2', weight: '400 700' }
    ],
    providers: { google: false }
  },
  
  critters: {
    config: { preload: 'swap', inlineFonts: false }
  },
  
  app: {
    head: {
      link: [
        { rel: 'preload', href: '/fonts/Inter-Variable.woff2', as: 'font', type: 'font/woff2', crossorigin: 'anonymous' }
      ]
    }
  },
  
  nitro: {
    preset: 'cloudflare_pages_static',
    prerender: { crawlLinks: true, routes: ['/'] }
  }
})
```

Cette configuration atteint un **LCP < 1.5s** sur connexion 4G, **zéro CLS** grâce aux fallbacks métriques, et un **score Lighthouse 95-100** en conditions idéales — entièrement compatible avec le tier gratuit Cloudflare Pages.