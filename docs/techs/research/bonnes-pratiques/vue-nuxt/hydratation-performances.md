# Hydratation et performance optimales avec Nuxt 4.2.X sur Cloudflare Pages

L'optimisation de l'hydratation représente le levier de performance le plus impactant pour un blog Nuxt 4. Les directives d'hydratation lazy, combinées aux Server Components et à une configuration Cloudflare adaptée, permettent d'atteindre des scores Core Web Vitals excellents tout en maintenant l'interactivité nécessaire. Ce rapport compile les bonnes pratiques les plus récentes de décembre 2025 pour chaque technique.

## Les quatre stratégies d'hydratation lazy de Nuxt 4

Les directives d'hydratation lazy, introduites dans Nuxt **3.16** et pleinement supportées dans Nuxt 4.2.X, s'appuient sur les API natives de Vue 3.5+. Elles permettent de différer l'exécution du JavaScript jusqu'au moment optimal, réduisant significativement le **TTI** (Time to Interactive) et le **TBT** (Total Blocking Time).

**hydrate-on-visible** utilise l'API `IntersectionObserver` pour hydrater un composant uniquement lorsqu'il entre dans le viewport. Cette stratégie est idéale pour les éléments below-the-fold comme les sections de commentaires, les footers, ou les galeries d'images en bas de page. Une marge de préchargement configurable via `rootMargin` permet d'anticiper l'hydratation :

```vue
<template>
  <!-- Hydrate 100px avant d'être visible -->
  <LazyCommentSection :hydrate-on-visible="{ rootMargin: '100px' }" />
  
  <!-- Footer en bas de page -->
  <LazyAppFooter hydrate-on-visible />
</template>
```

**hydrate-on-idle** exploite `requestIdleCallback` pour hydrater durant les périodes d'inactivité du navigateur. C'est le choix optimal pour les composants visibles mais non critiques : carrousels, widgets analytiques, graphiques. Un timeout maximum garantit l'hydratation même sur des machines moins performantes :

```vue
<template>
  <!-- Hydrate quand le navigateur est idle, max 2 secondes -->
  <LazyAnalyticsWidget :hydrate-on-idle="2000" />
</template>
```

**hydrate-on-interaction** attend une action utilisateur (click, hover, focus) avant d'hydrater. Les événements par défaut sont `pointerenter`, `click` et `focus`. Cette stratégie convient parfaitement aux menus déroulants, modals, ou formulaires de commentaires :

```vue
<template>
  <!-- Hydrate au survol ou clic -->
  <LazyDropdownMenu hydrate-on-interaction />
  
  <!-- Hydrate uniquement au focus -->
  <LazySearchForm hydrate-on-interaction="focus" />
  
  <!-- Plusieurs événements -->
  <LazyMobileMenu :hydrate-on-interaction="['click', 'touchstart']" />
</template>
```

**hydrate-never** rend le composant côté serveur mais n'envoie jamais de JavaScript au client. Le HTML reste statique et non-interactif. Cette option est parfaite pour le contenu purement textuel, les listes sans interaction, ou les sections statiques :

```vue
<template>
  <!-- Contenu article sans interactivité -->
  <LazyArticleBody hydrate-never />
</template>
```

Deux stratégies complémentaires méritent attention : `hydrate-on-media-query` hydrate selon la taille d'écran (utile pour des composants mobiles/desktop différents), et `hydrate-after` impose un délai fixe en millisecondes.

## Contrôle programmatique avec defineLazyHydrationComponent()

La macro `defineLazyHydrationComponent()` offre un contrôle fin sur l'hydratation avec des imports explicites. Elle permet de définir la stratégie directement dans le script setup :

```vue
<script setup lang="ts">
// Définition avec stratégie 'visible'
const LazyHeroGallery = defineLazyHydrationComponent(
  'visible',
  () => import('./components/HeroGallery.vue')
)

// Stratégie 'idle' avec import
const LazyWidgetPanel = defineLazyHydrationComponent(
  'idle',
  () => import('./components/WidgetPanel.vue')
)

// Stratégie conditionnelle
const LazyConditionalFeature = defineLazyHydrationComponent(
  'if',
  () => import('./components/ConditionalFeature.vue')
)
</script>

<template>
  <LazyHeroGallery :hydrate-on-visible="{ rootMargin: '50px' }" />
  <LazyWidgetPanel :hydrate-on-idle="3000" />
  <LazyConditionalFeature :hydrate-when="userAuthenticated" />
</template>
```

Les sept stratégies disponibles sont : `'visible'`, `'idle'`, `'interaction'`, `'mediaQuery'`, `'time'`, `'if'`, et `'never'`. **Contrainte critique** : les arguments doivent être des littéraux, pas des variables. L'événement `@hydrated` permet de déclencher des actions post-hydratation (analytics, animations).

## Server Components pour du contenu zéro-JavaScript

Les fichiers `.server.vue` créent des composants rendus **exclusivement côté serveur**. Le HTML est envoyé au client sans JavaScript associé, réduisant drastiquement la taille du bundle. Cette fonctionnalité reste expérimentale dans Nuxt 4.2.X mais est largement utilisée en production.

La configuration minimale requiert l'activation des component islands :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  experimental: {
    componentIslands: true,
    // Pour imbrication avec composants client
    // componentIslands: { selectiveClient: true }
  }
})
```

Pour un blog, les candidats idéaux aux Server Components sont le **parsing Markdown**, la **coloration syntaxique** (Shiki, Prism), les **headers/footers statiques**, et le contenu CMS en lecture seule. Ces dépendances lourdes restent sur le serveur :

```vue
<!-- components/CodeHighlight.server.vue -->
<script setup>
import { codeToHtml } from 'shiki' // ~100KB restent côté serveur
const props = defineProps({ code: String, lang: String })
const highlighted = await codeToHtml(props.code, { lang: props.lang })
</script>

<template>
  <div v-html="highlighted" />
</template>
```

Pour mixer contenu statique et îlots interactifs, l'attribut `nuxt-client` hydrate sélectivement un composant enfant :

```vue
<!-- components/ArticlePage.server.vue -->
<template>
  <article>
    <h1>{{ article.title }}</h1>
    <div v-html="article.content" />
    
    <!-- Cet îlot sera hydraté -->
    <CommentSection nuxt-client :article-id="article.id" />
  </article>
</template>
```

**Limitations importantes** : pas d'événements JavaScript (`@click` ignoré), pas de réactivité Vue côté client, et chaque modification de props déclenche une requête réseau pour re-render serveur.

## ClientOnly avec fallback pour widgets tiers

Le composant `<ClientOnly>` exclut son contenu du rendu SSR. Le slot `#fallback` affiche un placeholder jusqu'au montage côté client, **crucial pour éviter le CLS** (Cumulative Layout Shift).

Pour les widgets tiers (Disqus, analytics, social), toujours réserver l'espace visuel avec des dimensions explicites :

```vue
<template>
  <ClientOnly>
    <DisqusComments :identifier="postSlug" />
    <template #fallback>
      <div class="comments-placeholder" style="min-height: 450px;">
        <p>Chargement des commentaires...</p>
      </div>
    </template>
  </ClientOnly>
</template>
```

**Pattern skeleton loader** pour une meilleure UX :

```vue
<ClientOnly>
  <TwitterTimeline />
  <template #fallback>
    <div class="skeleton-feed" style="aspect-ratio: 1/1.5;">
      <div class="skeleton-item" v-for="i in 3" :key="i">
        <div class="skeleton-avatar" />
        <div class="skeleton-text" />
      </div>
    </div>
  </template>
</ClientOnly>
```

L'alternative `.client.vue` (suffixe de fichier) rend automatiquement un composant client-only, mais sans fallback natif. **Anti-patterns à éviter** : ClientOnly sans fallback pour gros composants, contenu SEO-critique dans ClientOnly, ou utilisation pour masquer des erreurs d'hydratation plutôt que les corriger.

## Configuration Nuxt Content 3 sur Cloudflare Pages

Nuxt Content 3 utilise désormais **SQLite** comme base de données. Sur Cloudflare Pages, une base **D1** est **obligatoire** car le filesystem Workers est read-only. La configuration complète :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  future: { compatibilityVersion: 4 },
  compatibilityDate: '2024-09-19',
  
  nitro: {
    preset: 'cloudflare_pages',
    cloudflare: {
      deployConfig: true,
      nodeCompat: true
    }
  },
  
  content: {
    database: {
      type: 'd1',
      bindingName: 'DB'
    }
  },
  
  routeRules: {
    '/': { prerender: true },
    '/blog/**': { prerender: true },
    '/_nuxt/**': {
      headers: { 'Cache-Control': 'public, max-age=31536000, immutable' }
    }
  }
})
```

```toml
# wrangler.toml
name = "mon-blog-nuxt"
compatibility_date = "2024-09-19"

[[d1_databases]]
binding = "DB"
database_name = "content-db"
database_id = "votre-database-id"
```

**Paramètres Cloudflare Dashboard à désactiver** pour éviter les erreurs d'hydratation : Rocket Loader™, Mirage, Email Obfuscation, et Auto Minify. Ces fonctionnalités injectent des scripts qui interfèrent avec l'hydratation Vue.

Pour les blogs, le **prerendering statique** est recommandé plutôt que l'edge rendering, car Cloudflare Pages ne supporte pas l'ISR nativement. Le dump SQLite est téléchargé au premier query puis exécuté localement via WASM, permettant le mode offline.

## Métriques de performance et outils de diagnostic

Les trois Core Web Vitals à surveiller prioritairement sont **LCP** (≤2.5s), **INP** (≤200ms, remplace FID depuis mars 2024), et **CLS** (≤0.1). L'hydratation impacte directement INP et TTI en bloquant le thread principal.

**Nuxt DevTools** intégré offre timeline de rendu, inspection des assets, et arbre de rendu. Le module **@nuxt/hints** ajoute des diagnostics Core Web Vitals en temps réel et un diff d'hydratation serveur/client. Pour la production, **@nuxtjs/web-vitals** permet le tracking continu :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/hints', '@nuxtjs/web-vitals'],
  devtools: { enabled: true },
  webVitals: { provider: 'log', debug: true }
})
```

La commande `npx nuxi analyze` génère une visualisation interactive du bundle pour identifier les dépendances lourdes. **Checklist performance** : utiliser `@nuxt/image` avec attribut `priority` pour images LCP, `@nuxt/fonts` pour éviter le FOUT, et `@nuxt/scripts` pour charger les scripts tiers de manière contrôlée.

## Structure de projet optimisée pour Claude Code

Organisation recommandée des fichiers pour un projet blog :

```
├── components/
│   ├── content/           # Composants MDC globaux
│   │   ├── Alert.vue
│   │   └── CodeBlock.server.vue
│   ├── blog/
│   │   ├── ArticleContent.server.vue
│   │   └── CommentSection.vue
│   └── layout/
│       ├── TheHeader.server.vue
│       └── TheFooter.server.vue
├── composables/
│   └── usePerformance.ts
├── content/
│   └── blog/
│       └── *.md
├── plugins/
│   └── web-vitals.client.ts
├── content.config.ts
├── nuxt.config.ts
└── wrangler.toml
```

**Conventions de nommage** : préfixe `Lazy` pour composants à hydratation différée, suffixe `.server.vue` pour Server Components, suffixe `.client.vue` pour composants client-only. Les composants MDC dans `components/content/` sont automatiquement disponibles dans le Markdown.

## Conclusion

L'optimisation de l'hydratation dans Nuxt 4.2.X repose sur une stratégie de **lazy loading par défaut** pour tout composant below-the-fold, l'utilisation de **Server Components** pour le contenu statique lourd (markdown, coloration syntaxique), et des **fallbacks dimensionnés** pour éviter le CLS. Sur Cloudflare Pages, le prerendering statique avec D1 offre les meilleures performances pour un blog. Les métriques INP et LCP doivent être surveillées en continu avec @nuxt/hints en développement et @nuxtjs/web-vitals en production. La combinaison de ces techniques peut réduire le bundle JavaScript client de **40 à 66%** selon la composition du site.