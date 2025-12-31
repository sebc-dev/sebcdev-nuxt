# Configuration @nuxtjs/i18n v10.2+ pour Nuxt 4 SSG sur Cloudflare Pages

La configuration optimale d'un blog bilingue français/anglais en mode SSG requiert une attention particulière aux **breaking changes de i18n v10**, à la **nouvelle structure Nuxt 4**, et aux **limitations du déploiement statique**. Ce guide couvre les 6 axes de configuration demandés avec des exemples de code prêts à l'emploi.

## Breaking changes critiques de la v9 à la v10

L'upgrade vers @nuxtjs/i18n v10 introduit des changements majeurs qui affectent directement votre configuration. Le plus important : **la propriété `iso` est renommée `language`** pour s'aligner avec les standards web (`navigator.language`, `Accept-Language`). L'option `lazy` a été **supprimée** car le lazy loading est désormais activé par défaut pour tous les fichiers de locale. Vue I18n passe à la v11, dépréciant l'API Legacy et supprimant `$tc()`.

La structure de fichiers par défaut utilise maintenant `restructureDir: 'i18n'`, résolu depuis le `rootDir` du projet. Les options `lazy`, `strategy` et `trailingSlash` sont devenues des **constantes de compilation** et ne peuvent plus être modifiées via runtime config.

## Définition des locales avec BCP 47

Pour un SEO multilingue optimal, la configuration des locales doit utiliser le format BCP 47 complet dans la propriété `language`. Le code court (`fr`, `en`) sert d'identifiant interne, tandis que `language` (`fr-FR`, `en-US`) génère les balises hreflang.

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  i18n: {
    locales: [
      { 
        code: 'fr',           // Identifiant interne (URLs: /, /article)
        language: 'fr-FR',    // BCP 47 pour hreflang et SEO
        name: 'Français',
        file: 'fr.json',
        dir: 'ltr'
      },
      { 
        code: 'en',           // Identifiant interne (URLs: /en, /en/article)
        language: 'en-US',    // BCP 47 pour hreflang
        name: 'English',
        file: 'en.json',
        dir: 'ltr'
      }
    ],
    defaultLocale: 'fr',
    baseUrl: 'https://votre-site.com'  // Requis pour URLs absolues hreflang
  }
})
```

La propriété `language` impacte directement : les attributs `hreflang` dans le sitemap et les balises `<link>`, l'attribut `lang` du `<html>`, les meta `og:locale`, et la détection de langue du navigateur. Utilisez toujours le format région (`fr-FR`) plutôt que langue seule (`fr`) pour un ciblage géographique précis.

## Strategy prefix_except_default pour le SEO français

Avec `prefix_except_default` et français comme `defaultLocale`, les URLs françaises restent sans préfixe (`/article`, `/a-propos`) tandis que l'anglais reçoit le préfixe `/en` (`/en/article`, `/en/about`). Cette stratégie est **recommandée pour le SEO** car elle préserve les URLs existantes de la langue principale.

```typescript
i18n: {
  strategy: 'prefix_except_default',
  defaultLocale: 'fr',
  
  // Custom paths pour URLs localisées différentes
  pages: {
    'about': {
      fr: '/a-propos',
      en: '/about'
    },
    'contact': {
      fr: '/contact',
      en: '/contact'
    }
  }
}
```

**Impact sur les URLs générées** :

| Page | URL française | URL anglaise |
|------|---------------|--------------|
| Index | `/` | `/en` |
| Article | `/article-slug` | `/en/article-slug` |
| À propos | `/a-propos` | `/en/about` |

Le sitemap génère automatiquement les balises hreflang incluant `x-default` pointant vers l'URL française (langue par défaut). L'intégration avec @nuxtjs/sitemap v7.5+ est automatique via `autoI18n`.

## Lazy loading et structure des fichiers de traduction

En v10, le lazy loading est **activé par défaut** — ne configurez plus `lazy: true`. Le `langDir` est résolu relativement à `restructureDir` (défaut: `'i18n'`), lui-même relatif au `rootDir` du projet (pas `srcDir`).

**Structure recommandée pour Nuxt 4 avec `app/` comme srcDir** :

```
projet/
├── app/                    # srcDir
│   ├── pages/
│   ├── components/
│   └── ...
├── i18n/                   # À la racine, PAS dans app/
│   ├── locales/
│   │   ├── fr.json
│   │   └── en.json
│   └── i18n.config.ts      # Config Vue I18n (auto-détecté)
├── content/
│   ├── blog_fr/
│   └── blog_en/
└── nuxt.config.ts
```

Les fichiers doivent rester **au niveau racine** car ils sont utilisés côté client ET serveur. Le placement dans `app/` causerait des problèmes de résolution.

**Format JSON pour les traductions** (recommandé pour SSG) :

```json
// i18n/locales/fr.json
{
  "nav": {
    "home": "Accueil",
    "blog": "Blog",
    "about": "À propos"
  },
  "blog": {
    "title": "Articles",
    "readMore": "Lire la suite"
  },
  "seo": {
    "defaultTitle": "Mon Blog",
    "defaultDescription": "Un blog bilingue sur..."
  }
}
```

**Format TypeScript** (pour traductions dynamiques ou API) :

```typescript
// i18n/locales/en.ts
export default defineI18nLocale(async (locale) => {
  // Possibilité de fetch depuis une API
  return {
    nav: { home: 'Home', blog: 'Blog', about: 'About' },
    blog: { title: 'Articles', readMore: 'Read more' }
  }
})
```

Pour le **bundle size en SSG**, le JSON est préférable car plus léger et statiquement analysable. Les traductions de la locale courante sont injectées dans le HTML pré-rendu ; les autres locales chargent à la demande lors du switch.

## Browser detection optimisée pour SSG sur Cloudflare

La détection de langue du navigateur présente des **limitations importantes en SSG** : pas d'accès au header `Accept-Language` sur un hébergement statique, et la détection retourne `detect_ignore_on_ssg` pendant le build. La configuration recommandée privilégie les cookies côté client.

```typescript
i18n: {
  detectBrowserLanguage: {
    useCookie: true,
    cookieKey: 'i18n_locale',
    cookieDomain: null,       // Domaine actuel
    cookieSecure: true,       // HTTPS uniquement (Cloudflare = HTTPS)
    cookieCrossOrigin: false, // true si iframe
    
    // CRITIQUE pour SSG + SEO
    redirectOn: 'root',       // Détection UNIQUEMENT sur /
    alwaysRedirect: false,    // Ne pas rediriger à chaque visite
    fallbackLocale: 'fr'      // Fallback si détection échoue
  }
}
```

**Pourquoi `redirectOn: 'root'`** : avec `'all'`, Google Crawler (sans `Accept-Language`) serait redirigé sur chaque page, nuisant au SEO. Avec `'root'`, seule la homepage détecte la langue ; les URLs directes (`/en/article`) fonctionnent sans redirection.

**Comportement utilisateur** :
- **Première visite sur `/`** : détection via `navigator.language`, redirection si match, cookie posé
- **Visites suivantes** : cookie lu, préférence respectée
- **Accès direct `/en/article`** : pas de redirection, contenu anglais servi

Pour les sites où le SEO prime sur l'UX de détection, considérez `detectBrowserLanguage: false` et laissez les utilisateurs choisir via un sélecteur de langue.

## Intégration avec Nuxt Content 3 et collections multilingues

L'approche recommandée utilise des **collections séparées par langue** plutôt qu'un champ locale dans une collection unique. Cela simplifie les requêtes et la génération de routes.

```typescript
// content.config.ts
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { z } from 'zod'

const blogSchema = z.object({
  title: z.string(),
  description: z.string(),
  date: z.string(),
  image: z.string().optional(),
  // Pour lier les versions traduites
  translationKey: z.string().optional()
})

export default defineContentConfig({
  collections: {
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'blog_fr/**', prefix: '/blog' },
      schema: blogSchema
    }),
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'blog_en/**', prefix: '/blog' },
      schema: blogSchema
    })
  }
})
```

**Structure des fichiers content** :

```
content/
├── blog_fr/
│   ├── mon-premier-article.md
│   └── deuxieme-article.md
└── blog_en/
    ├── my-first-article.md
    └── second-article.md
```

**Page dynamique `pages/blog/[...slug].vue`** :

```vue
<script setup lang="ts">
import { withLeadingSlash, joinURL } from 'ufo'
import type { Collections } from '@nuxt/content'

const { locale, t } = useI18n()
const localePath = useLocalePath()
const route = useRoute()

// Construire le chemin du contenu
const contentPath = computed(() => 
  withLeadingSlash(joinURL('blog', ...(
    Array.isArray(route.params.slug) 
      ? route.params.slug 
      : [route.params.slug]
  )))
)

// Query avec collection basée sur la locale
const { data: article } = await useAsyncData(
  `blog-${contentPath.value}-${locale.value}`,
  async () => {
    const collection = `blog_${locale.value}` as keyof Collections
    let content = await queryCollection(collection)
      .path(contentPath.value)
      .first()
    
    // Fallback vers français si contenu manquant en anglais
    if (!content && locale.value !== 'fr') {
      content = await queryCollection('blog_fr')
        .path(contentPath.value)
        .first()
      if (content) content._isFallback = true
    }
    
    return content
  },
  { watch: [locale] }
)

// SEO avec hreflang automatique
const head = useLocaleHead({ addSeoAttributes: true })
useHead(() => ({
  htmlAttrs: { lang: head.value.htmlAttrs?.lang },
  link: [...(head.value.link || [])],
  meta: [...(head.value.meta || [])],
  title: article.value?.title || t('blog.title')
}))
</script>

<template>
  <article v-if="article">
    <div v-if="article._isFallback" class="notice">
      {{ t('content.notAvailableInLanguage') }}
    </div>
    <h1>{{ article.title }}</h1>
    <time>{{ new Date(article.date).toLocaleDateString(locale) }}</time>
    <ContentRenderer :value="article" />
  </article>
</template>
```

**Génération des hreflang** via `useLocaleHead()` dans le layout :

```vue
<!-- layouts/default.vue -->
<script setup>
const head = useLocaleHead({
  addDirAttribute: true,
  addSeoAttributes: true
})

useHead(() => ({
  htmlAttrs: { 
    lang: head.value.htmlAttrs?.lang,
    dir: head.value.htmlAttrs?.dir 
  },
  link: [...(head.value.link || [])],
  meta: [...(head.value.meta || [])]
}))
</script>
```

Cela génère automatiquement :
- `<html lang="fr-FR">` ou `<html lang="en-US">`
- `<link rel="alternate" hreflang="fr-FR" href="...">`
- `<link rel="alternate" hreflang="en-US" href="...">`
- `<link rel="alternate" hreflang="x-default" href="...">` (pointe vers français)
- `<meta property="og:locale" content="fr_FR">`

## Configuration nuxt.config.ts complète

Voici la configuration production-ready intégrant tous les éléments :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  compatibilityDate: '2024-12-01',
  
  modules: [
    '@nuxtjs/i18n',
    '@nuxt/content',
    '@nuxtjs/sitemap',
    'nuxt-schema-org'
  ],

  // SSG Configuration
  ssr: true,
  
  nitro: {
    preset: 'cloudflare-pages',
    prerender: {
      autoSubfolderIndex: false, // Match Cloudflare routing
      crawlLinks: true,
      routes: ['/', '/en']       // Points d'entrée pour crawler
    }
  },

  // Site metadata pour SEO modules
  site: {
    url: 'https://votre-site.com',
    name: 'Mon Blog Bilingue'
  },

  // ═══════════════════════════════════════════
  // Configuration @nuxtjs/i18n v10.2+
  // ═══════════════════════════════════════════
  i18n: {
    // Locales avec BCP 47
    locales: [
      { 
        code: 'fr', 
        language: 'fr-FR',
        name: 'Français',
        file: 'fr.json'
      },
      { 
        code: 'en', 
        language: 'en-US',
        name: 'English',
        file: 'en.json'
      }
    ],
    
    defaultLocale: 'fr',
    strategy: 'prefix_except_default',
    
    // URLs absolues pour hreflang/sitemap
    baseUrl: 'https://votre-site.com',
    
    // Fichiers de traduction (lazy loading par défaut en v10)
    // langDir résolu: <rootDir>/i18n/locales/
    
    // Browser detection pour SSG
    detectBrowserLanguage: {
      useCookie: true,
      cookieKey: 'i18n_locale',
      cookieSecure: true,
      redirectOn: 'root',
      alwaysRedirect: false,
      fallbackLocale: 'fr'
    },
    
    // Bundle optimization
    bundle: {
      compositionOnly: true  // Tree-shake Legacy API
    },
    
    // v10 experimental features
    experimental: {
      strictSeo: true  // Gestion automatique des tags SEO i18n
    }
  },

  // ═══════════════════════════════════════════
  // Configuration Sitemap avec autoI18n
  // ═══════════════════════════════════════════
  sitemap: {
    cacheMaxAgeSeconds: 3600 * 24,
    // autoI18n détecté automatiquement depuis i18n config
    exclude: ['/admin/**', '/preview/**'],
    sources: ['/api/__sitemap__/urls']
  },

  // ═══════════════════════════════════════════
  // Schema.org pour SEO structuré
  // ═══════════════════════════════════════════
  schemaOrg: {
    identity: {
      type: 'Organization',
      name: 'Mon Blog',
      logo: '/logo.png'
    }
  },

  // Cache headers pour assets
  routeRules: {
    '/_nuxt/**': { 
      headers: { 'cache-control': 'public, max-age=31536000, immutable' } 
    }
  }
})
```

**Fichier `i18n/i18n.config.ts`** (optionnel, pour config Vue I18n runtime) :

```typescript
export default defineI18nConfig(() => ({
  legacy: false,
  fallbackLocale: 'fr',
  missingWarn: false,
  fallbackWarn: false
}))
```

## Fichiers de configuration Cloudflare Pages

**`public/_redirects`** pour gérer les URLs sans préfixe :

```
# Redirections pour anciennes URLs ou SEO
/a-propos /fr/a-propos 301
/about /en/about 301
```

**`public/_headers`** pour optimisation :

```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff

/
  Content-Language: fr

/en/*
  Content-Language: en
```

## Points d'attention spécifiques SSG + Cloudflare

Trois limitations méritent attention pour ce stack. Premièrement, la **détection `Accept-Language` ne fonctionne pas** sur hébergement statique — seul `navigator.language` côté client est disponible après hydration. Deuxièmement, les **cookies sont client-side uniquement** : le serveur Cloudflare ne les lit pas, la préférence langue n'affecte que les visites après le premier chargement. Troisièmement, avec `prefix_except_default`, accéder à `/article` (français) fonctionne car le fichier HTML est généré, mais il n'y a pas de fichier pour `/en/article` sans le préfixe anglais — le routing est correct par design.

Pour le **prerendering complet**, assurez-vous que toutes les pages sont liées depuis les points d'entrée (`/`, `/en`) via `<NuxtLink>` utilisant `localePath()`. Le crawler Nitro découvrira automatiquement toutes les routes localisées.