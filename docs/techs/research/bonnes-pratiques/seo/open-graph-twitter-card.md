# Open Graph et Twitter Cards pour Nuxt 4 SSG sur Cloudflare Pages

**useSeoMeta() est l'API recommandée** pour implémenter les meta tags sociaux dans Nuxt 4.2.x en mode SSG. Cette fonction type-safe gère automatiquement la distinction entre `name` et `property`, évite les erreurs XSS, et offre un typage TypeScript complet avec plus de 100 propriétés. Pour un blog multilingue déployé sur Cloudflare Pages, la combinaison `@nuxtjs/seo` + `nuxt-og-image` (mode Zero Runtime) + `@nuxtjs/i18n` permet une génération complète des meta tags et images OG au build time, sans server functions et donc **gratuitement**.

---

## Architecture recommandée des modules Nuxt 4

L'ordre de déclaration des modules dans `nuxt.config.ts` est critique pour le bon fonctionnement de la chaîne SEO. **@nuxtjs/seo doit être déclaré avant @nuxt/content** pour que le frontmatter SEO soit correctement traité.

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@nuxtjs/seo',      // 1. PREMIER - suite complète SEO
    '@nuxt/content',    // 2. Content après SEO
    '@nuxtjs/i18n',     // 3. i18n en dernier
  ],
  
  site: {
    url: 'https://monblog.pages.dev',
    name: 'Mon Blog Tech',
    defaultLocale: 'fr-FR',
  },
  
  seo: {
    meta: {
      ogSiteName: 'Mon Blog Tech',
      twitterCard: 'summary_large_image',
      twitterSite: '@monblog',
    }
  },
  
  ogImage: {
    zeroRuntime: true,  // ESSENTIEL pour SSG pur
    runtimeCacheStorage: false,
    defaults: { renderer: 'satori', width: 1200, height: 630 }
  },
  
  nitro: {
    prerender: { crawlLinks: true, routes: ['/'] }
  }
})
```

Le module `@nuxtjs/seo` inclut **6 sous-modules** : Sitemap, Robots, OG Image, Schema.org, Link Checker, et SEO Utils. Cette suite couvre l'ensemble des besoins SEO d'un blog sans configuration additionnelle.

---

## useSeoMeta() versus useHead() : quand utiliser chaque API

**useSeoMeta()** est conçu exclusivement pour les meta tags SEO et sociaux. Son typage plat (`ogTitle`, `twitterCard`, `articlePublishedTime`) élimine la complexité des objets meta traditionnels et prévient les erreurs de syntaxe courantes.

```typescript
// pages/blog/[slug].vue
<script setup lang="ts">
const { data: article } = await useAsyncData('article', () => 
  queryCollection('blog_fr').path(`/blog/${route.params.slug}`).first()
)

useSeoMeta({
  title: () => article.value?.title,
  description: () => article.value?.description,
  ogTitle: () => article.value?.title,
  ogDescription: () => article.value?.description,
  ogImage: () => article.value?.image || '/default-og.png',
  ogType: 'article',
  articlePublishedTime: () => article.value?.date,
  articleModifiedTime: () => article.value?.updatedAt,
  articleAuthor: () => article.value?.author,
  articleSection: () => article.value?.category,
  twitterCard: 'summary_large_image',
})
</script>
```

**useHead()** reste nécessaire pour les éléments non-SEO : scripts externes, feuilles de style, attributs `htmlAttrs`/`bodyAttrs`, favicons, et balises `link`. Pour le SSG, les meta tags statiques peuvent être wrappés dans `if (import.meta.server)` pour une optimisation marginale, mais Nuxt gère déjà efficacement le rendu côté serveur.

---

## Spécifications techniques Open Graph pour articles de blog

Les dimensions d'image Open Graph suivent un standard précis que Facebook et LinkedIn utilisent pour l'affichage. Une image mal dimensionnée sera soit tronquée, soit affichée en miniature, réduisant significativement l'engagement.

| Propriété | Valeur recommandée | Notes |
|-----------|-------------------|-------|
| og:image | **1200×630px** | Ratio 1.91:1, format PNG ou JPEG |
| og:title | **≤65 caractères** | Troncation après ~70 caractères |
| og:description | **150-200 caractères** | Maximum visible selon plateforme |
| og:type | `article` | Active les propriétés article:* |
| article:published_time | ISO 8601 | `2025-12-29T09:00:00+01:00` |

Le format ISO 8601 pour les dates **doit inclure le fuseau horaire** pour éviter les ambiguïtés d'interprétation. Les tags `article:section` (catégorie unique) et `article:tag` (multiples valeurs) enrichissent le contexte sémantique pour les algorithmes de recommandation.

---

## Twitter Cards : différences clés avec Open Graph

Twitter utilise l'attribut `name` au lieu de `property` et **requiert obligatoirement twitter:card** — aucun fallback OG n'existe pour cette propriété. Cependant, `twitter:title`, `twitter:description` et `twitter:image` **tombent automatiquement** sur leurs équivalents OG si absents.

```typescript
useSeoMeta({
  // Ces 3 props OG servent de fallback pour Twitter
  ogTitle: 'Mon Article',
  ogDescription: 'Description de mon article',
  ogImage: 'https://monsite.com/og.png',
  
  // twitter:card est OBLIGATOIRE (pas de fallback)
  twitterCard: 'summary_large_image',
  
  // Optionnels mais recommandés
  twitterSite: '@MonBlog',     // Compte du site
  twitterCreator: '@Auteur',   // Compte de l'auteur
})
```

Pour un blog, **summary_large_image** est systématiquement préférable à `summary` : l'image proéminente (**1200×628px** minimum) capture davantage l'attention dans le flux Twitter. Le format carré de `summary` (144×144px) convient uniquement aux profils ou pages sans visuel principal.

---

## Génération d'images OG en mode Zero Runtime

Le module `nuxt-og-image` avec l'option **zeroRuntime: true** génère les images au build time via Satori (conversion Vue → SVG → PNG), éliminant tout besoin de server functions. Cette approche est **100% compatible avec Cloudflare Pages gratuit**.

```vue
<!-- components/OgImage/BlogPost.vue -->
<script setup lang="ts">
withDefaults(defineProps<{
  title?: string
  description?: string
  siteName?: string
}>(), {
  title: 'Mon Article',
  siteName: 'MonBlog.com',
})
</script>

<template>
  <div class="h-full w-full flex flex-col justify-between p-16 bg-slate-800">
    <div class="flex flex-col gap-4">
      <h1 class="text-white text-6xl font-bold leading-tight">{{ title }}</h1>
      <p class="text-gray-200 text-2xl">{{ description }}</p>
    </div>
    <span class="text-white text-xl font-semibold">{{ siteName }}</span>
  </div>
</template>
```

Utilisez `defineOgImageComponent()` dans vos pages pour associer ce template :

```typescript
defineOgImageComponent('BlogPost', {
  title: article.value?.title,
  description: article.value?.description,
})
```

**Performance build** : Satori génère environ **50-100ms par image**, soit ~10 secondes pour 100 articles. Les images sont créées dans `.output/public/__og-image__/` et servies comme assets statiques avec un cache CDN illimité sur Cloudflare.

---

## Intégration multilingue avec @nuxtjs/i18n v10+

Le format `og:locale` utilise un underscore (`fr_FR`) alors que BCP 47 utilise un tiret (`fr-FR`). **Nuxt SEO Utils convertit automatiquement** le format lors du rendu, donc configurez vos locales en BCP 47.

```typescript
// nuxt.config.ts
i18n: {
  locales: [
    { code: 'fr', language: 'fr-FR', name: 'Français' },
    { code: 'en', language: 'en-US', name: 'English' },
  ],
  defaultLocale: 'fr',
  strategy: 'prefix_except_default',
  baseUrl: 'https://monblog.com', // Requis pour hreflang
}
```

Le composable `useLocaleHead()` génère automatiquement les tags `hreflang` et `og:locale:alternate` :

```html
<!-- Rendu automatique -->
<link rel="alternate" hreflang="fr-FR" href="https://example.com/blog/article">
<link rel="alternate" hreflang="en-US" href="https://example.com/en/blog/article">
<link rel="alternate" hreflang="x-default" href="https://example.com/blog/article">
<meta property="og:locale" content="fr_FR">
<meta property="og:locale:alternate" content="en_US">
```

Pour Nuxt Content 3, structurez vos fichiers par locale (`content/fr/blog/*.md`, `content/en/blog/*.md`) et utilisez `queryCollection()` avec le suffixe de langue approprié.

---

## Frontmatter YAML complet pour Nuxt Content 3

Un frontmatter bien structuré centralise toutes les métadonnées SEO et permet des fallbacks automatiques :

```yaml
---
title: 'Guide Open Graph pour Nuxt 4'
description: 'Implémentez les meta tags OG et Twitter Cards dans votre blog Nuxt SSG'
date: '2025-12-29'
author: 'Jean Dupont'
image: '/images/blog/og-guide.jpg'
tags: [nuxt, seo, open-graph]
category: 'Développement Web'

# Overrides OG optionnels
ogTitle: 'Guide OG complet pour Nuxt 4 SSG'
ogDescription: 'Tout ce qu'il faut savoir sur les meta tags sociaux'

# Schema.org intégré
schemaOrg:
  - '@type': 'BlogPosting'
    headline: 'Guide Open Graph pour Nuxt 4'
    datePublished: '2025-12-29'
    author:
      '@type': 'Person'
      name: 'Jean Dupont'
---
```

L'extraction dans la page utilise `queryCollection()` de Nuxt Content 3, et les champs `ogTitle`/`ogDescription` servent d'override lorsqu'un titre social distinct est souhaitable.

---

## Validation et invalidation du cache des crawlers

Après chaque mise à jour de meta tags, **le cache des plateformes sociales doit être purgé** pour voir les changements. Facebook conserve les données plusieurs jours, Twitter jusqu'à 7 jours.

- **Facebook Sharing Debugger** : `developers.facebook.com/tools/debug/sharing/` — Entrer l'URL, cliquer "Scrape Again" (parfois plusieurs fois)
- **Twitter Card Validator** : `cards-dev.twitter.com/validator` ou alternatives tierces comme typefully.com/tools/twitter-card-validator
- **LinkedIn Post Inspector** : `linkedin.com/post-inspector/`

Les erreurs courantes incluent : image inaccessible (vérifier que l'URL est publique et non bloquée par robots.txt), dimensions incorrectes (utiliser exactement 1200×630), et format WebP non supporté par LinkedIn. **Utilisez PNG** pour une compatibilité universelle.

---

## Composable personnalisé pour centraliser la logique SEO

Pour éviter la duplication de code entre les pages d'articles, créez un composable dédié :

```typescript
// composables/useArticleMeta.ts
export function useArticleMeta(page: Ref<any>) {
  const { locale } = useI18n()
  const config = useRuntimeConfig()
  
  watchEffect(() => {
    if (!page.value) return
    
    useSeoMeta({
      title: page.value.ogTitle || page.value.title,
      description: page.value.ogDescription || page.value.description,
      ogTitle: page.value.ogTitle || page.value.title,
      ogDescription: page.value.ogDescription || page.value.description,
      ogImage: page.value.ogImage || page.value.image || '/default-og.png',
      ogType: 'article',
      ogLocale: locale.value.replace('-', '_'),
      articlePublishedTime: page.value.date,
      articleModifiedTime: page.value.updatedAt,
      articleAuthor: page.value.author,
      twitterCard: 'summary_large_image',
    })
    
    defineOgImageComponent('BlogPost', {
      title: page.value.title,
      description: page.value.description,
    })
  })
}
```

L'utilisation devient alors triviale : `useArticleMeta(article)` dans chaque page de blog.

---

## Conclusion

L'implémentation OG/Twitter Cards pour Nuxt 4 SSG sur Cloudflare Pages repose sur trois piliers : **useSeoMeta() pour le typage et la sécurité**, **nuxt-og-image en mode zeroRuntime pour les images générées au build**, et **useLocaleHead() pour le multilingue automatique**. Cette stack fonctionne entièrement en statique, sans coût d'hébergement, et produit des meta tags complets dès le prerendering. Les points d'attention critiques sont l'ordre des modules dans `nuxt.config.ts`, le format ISO 8601 avec timezone pour les dates, et l'utilisation systématique de PNG (pas WebP) pour les images OG afin d'assurer la compatibilité LinkedIn.