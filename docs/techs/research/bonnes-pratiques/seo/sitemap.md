# Guide complet @nuxtjs/sitemap v7.5+ pour Nuxt 4 SSG sur Cloudflare Pages

Le module **@nuxtjs/sitemap v7.5.0** (décembre 2024) offre une intégration native avec Nuxt 4.2.x et un mode `zeroRuntime` idéal pour les déploiements SSG sur Cloudflare Pages. L'intégration i18n est automatique, la découverte d'images fonctionne au build, et la personnalisation XSL facilite le debugging. Ce guide couvre les configurations essentielles et les patterns recommandés pour un déploiement production.

## Configuration autoI18n avec @nuxtjs/i18n v10.2.1+

L'intégration entre les deux modules est **automatique et sans configuration** lorsque les deux sont installés. Le module sitemap détecte `@nuxtjs/i18n` et configure `autoI18n` à partir de vos paramètres i18n existants.

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@nuxtjs/i18n',     // DOIT être avant sitemap
    '@nuxtjs/sitemap'
  ],
  
  site: {
    url: 'https://example.com'
  },

  i18n: {
    locales: [
      { code: 'fr', language: 'fr-FR' },
      { code: 'en', language: 'en-US' },
      { code: 'de', language: 'de-DE' }
    ],
    defaultLocale: 'fr',
    strategy: 'prefix_except_default',
    detectBrowserLanguage: false  // Recommandé pour sitemaps prévisibles
  },

  sitemap: {
    // autoI18n est configuré automatiquement - aucune config explicite nécessaire
    xslColumns: [
      { label: 'URL', width: '50%' },
      { label: 'Last Modified', select: 'sitemap:lastmod', width: '25%' },
      { label: 'Hreflangs', select: 'count(xhtml:link)', width: '25%' }
    ]
  }
})
```

**Génération automatique des sitemaps par locale** : avec la stratégie `prefix_except_default`, le module génère `/sitemap_index.xml`, `/fr-sitemap.xml`, `/en-sitemap.xml` et `/de-sitemap.xml`. Chaque URL inclut les balises `xhtml:link` avec attributs `hreflang` pointant vers toutes les alternatives linguistiques.

Le XML généré pour une page `/about` ressemble à :

```xml
<url>
  <loc>https://example.com/about</loc>
  <xhtml:link rel="alternate" hreflang="fr" href="https://example.com/about"/>
  <xhtml:link rel="alternate" hreflang="en" href="https://example.com/en/about"/>
  <xhtml:link rel="alternate" hreflang="de" href="https://example.com/de/about"/>
</url>
```

Pour les URLs dynamiques provenant d'API, utilisez le flag `_i18nTransform: true` pour générer automatiquement les variantes localisées :

```typescript
// server/api/__sitemap__/urls.ts
export default defineSitemapEventHandler(() => {
  return [
    {
      loc: '/blog/mon-article',
      _i18nTransform: true  // Génère /blog/mon-article, /en/blog/mon-article, /de/blog/mon-article
    }
  ]
})
```

## Sources dynamiques en mode SSG sans server functions

En mode SSG, les endpoints comme `/api/__sitemap__/urls` ne fonctionnent **pas au runtime** car aucun serveur n'est déployé. Trois alternatives existent pour générer des URLs dynamiques au build time.

**Option 1 : Fonction `urls` (recommandée)** — Cette fonction s'exécute une seule fois pendant la génération du sitemap :

```typescript
export default defineNuxtConfig({
  sitemap: {
    urls: async () => {
      const [posts, products] = await Promise.all([
        fetch('https://api.cms.com/posts').then(r => r.json()),
        fetch('https://api.cms.com/products').then(r => r.json())
      ])
      
      return [
        ...posts.map(p => ({
          loc: `/blog/${p.slug}`,
          lastmod: p.updatedAt,
          priority: 0.7
        })),
        ...products.map(p => ({
          loc: `/produits/${p.id}`,
          priority: 0.8
        }))
      ]
    }
  }
})
```

**Option 2 : Hook `prerender:routes`** — Ajoute dynamiquement des routes pendant le build :

```typescript
export default defineNuxtConfig({
  hooks: {
    async 'prerender:routes'(ctx) {
      const posts = await fetch('https://api.cms.com/posts').then(r => r.json())
      for (const post of posts) {
        ctx.routes.add(`/blog/${post.slug}`)
      }
    }
  }
})
```

**Option 3 : Source JSON externe** — Fichier statique accessible au build :

```typescript
sitemap: {
  sources: ['https://cdn.example.com/sitemap-urls.json']
}
```

Pour les **multi-sitemaps** sur sites volumineux (>1000 URLs), divisez par catégorie avec chunking automatique :

```typescript
sitemap: {
  sitemaps: {
    pages: {
      urls: ['/about', '/contact', '/services'],
      exclude: ['/blog/**']
    },
    posts: {
      urls: async () => fetchPosts(),
      chunks: 5000  // Divise en fichiers de 5000 URLs
    }
  }
}
```

## Hook sitemap:resolved pour manipulation avancée

Le hook `sitemap:resolved` permet de modifier les URLs après leur résolution. Pour Nuxt 4, placez ce code dans `server/plugins/sitemap.ts` :

```typescript
// server/plugins/sitemap.ts
import { defineNitroPlugin } from 'nitropack/runtime'

export default defineNitroPlugin((nitroApp) => {
  nitroApp.hooks.hook('sitemap:resolved', async (ctx) => {
    // ctx.urls: ResolvedSitemapUrl[]
    // ctx.sitemapName: string
    
    // Filtrer par sitemap spécifique
    if (ctx.sitemapName === 'posts') {
      ctx.urls = ctx.urls.filter(url => !url.loc.includes('/draft/'))
    }
    
    // Modifier les priorités dynamiquement
    ctx.urls = ctx.urls.map(url => {
      if (url.loc.startsWith('/featured/')) {
        return { ...url, priority: 1.0 }
      }
      return url
    })
  })
})
```

**Important** : En Nuxt 4+, les hooks Nuxt (dans `nuxt.config.ts`) ne fonctionnent que pendant le prerendering. Pour les modifications runtime, utilisez exclusivement les hooks Nitro dans `server/plugins/`.

## Image discovery avec Nuxt Content 3.10+

La découverte d'images nécessite le **prerendering** et fonctionne automatiquement lorsque `discoverImages: true` (défaut). Le module scanne les balises `<img>` **uniquement dans la balise `<main>`** du HTML généré.

```typescript
export default defineNuxtConfig({
  sitemap: {
    discoverImages: true,   // Défaut: activé
    discoverVideos: true    // Découvre aussi les <video>
  },
  
  nitro: {
    prerender: {
      crawlLinks: true,
      routes: ['/', '/sitemap.xml']
    }
  }
})
```

Pour **Nuxt Content v3**, utilisez `asSitemapCollection` pour intégrer automatiquement le contenu Markdown :

```typescript
// content.config.ts
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { asSitemapCollection } from '@nuxtjs/sitemap/content'

export default defineContentConfig({
  collections: {
    content: defineCollection(
      asSitemapCollection({
        type: 'page',
        source: '**/*.md'
      })
    )
  }
})
```

**Ordre des modules critique** : `@nuxtjs/sitemap` doit être listé **AVANT** `@nuxt/content` dans l'array `modules`.

Le XML généré pour les images suit la spécification Google :

```xml
<url>
  <loc>https://example.com/blog/article</loc>
  <image:image>
    <image:loc>https://example.com/images/hero.jpg</image:loc>
  </image:image>
  <image:image>
    <image:loc>https://example.com/images/diagram.png</image:loc>
  </image:image>
</url>
```

Les tags `caption`, `title`, `license` et `geoLocation` sont **dépréciés par Google depuis mai 2022** mais restent utiles pour Bing.

## Configuration XSL pour debugging et visualisation

Le stylesheet XSL transforme le sitemap XML en page HTML lisible. Par défaut, il est servi à `/__sitemap__/style.xsl`.

```typescript
export default defineNuxtConfig({
  sitemap: {
    xsl: '/__sitemap__/style.xsl',  // Chemin par défaut
    xslTips: true,                   // Affiche des conseils en développement
    xslColumns: [
      { label: 'URL', width: '50%' },
      { label: 'Images', select: 'count(image:image)', width: '25%' },
      { label: 'Last Modified', select: 'sitemap:lastmod', width: '25%' }
    ]
  }
})
```

**Utilité développement vs production** : Le XSL n'a **aucun impact SEO** — c'est purement visuel pour le debugging. En production, désactivez-le avec `xsl: false` ou conservez-le pour faciliter la vérification manuelle. Les balises hreflang ne sont **pas visibles** dans la vue XSL mais présentes dans le XML source.

Pour créer un XSL personnalisé, copiez le template par défaut et servez-le depuis `/public/sitemap.xsl` :

```typescript
sitemap: {
  xsl: '/sitemap.xsl'  // Fichier dans public/
}
```

## Optimisations Cloudflare Pages avec preset static

Pour un déploiement SSG pur sur Cloudflare Pages, utilisez le preset `static` plutôt que `cloudflare_pages` (qui active les serverless functions) :

```typescript
export default defineNuxtConfig({
  nitro: {
    preset: 'static'  // Critique pour SSG pur
  },
  
  sitemap: {
    zeroRuntime: true  // Nouveau en v7.5.0 - réduit le bundle de ~50KB
  }
})
```

**Configuration build Cloudflare Pages** :
- Build command : `npm run generate` ou `nuxt generate`
- Output directory : `.output/public`
- Variables d'environnement : `NITRO_PRESET=static` (alternative à la config)

Pour les **headers de cache**, créez un fichier `public/_headers` :

```
/sitemap.xml
  Cache-Control: public, max-age=86400, s-maxage=86400
  Content-Type: application/xml

/sitemap_index.xml
  Cache-Control: public, max-age=86400, s-maxage=86400

/*-sitemap.xml
  Cache-Control: public, max-age=86400, s-maxage=86400
```

Le mode `zeroRuntime: true` tree-shake le code sitemap du bundle runtime et ajoute automatiquement `/sitemap.xml` à `nitro.prerender.routes`.

## Compatibilité Nuxt 4 et structure de projet

Nuxt 4 introduit la nouvelle structure `app/` pour le code Vue et conserve `server/` pour Nitro. Le module sitemap v7.4.4+ inclut `moduleDependencies` pour la compatibilité Nuxt v4.1+.

```
project/
├── app/                    # Code Vue (pages, components, layouts)
│   └── pages/
├── server/
│   ├── api/
│   │   └── __sitemap__/
│   │       └── urls.ts     # Sources dynamiques sitemap
│   └── plugins/
│       └── sitemap.ts      # Hooks Nitro pour sitemap
├── content/                # Nuxt Content files
└── nuxt.config.ts
```

**Aucun breaking change majeur** n'existe entre Nuxt 3 et 4 pour ce module. Les principales différences concernent l'emplacement des fichiers :

- Les hooks Nitro (`sitemap:resolved`, `sitemap:input`) vont dans `server/plugins/`
- Les sources API (`defineSitemapEventHandler`) vont dans `server/api/`
- Les auto-imports diffèrent entre contextes `app/` et `server/`

```typescript
// server/api/__sitemap__/urls.ts (Nuxt 4)
import { defineSitemapEventHandler, asSitemapUrl } from '#imports'

export default defineSitemapEventHandler(async () => {
  const posts = await $fetch('/api/posts')
  return posts.map(post => asSitemapUrl({
    loc: `/blog/${post.slug}`,
    lastmod: post.updatedAt
  }))
})
```

## Configuration complète recommandée

Voici une configuration production-ready combinant tous les aspects :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@nuxtjs/i18n',
    '@nuxtjs/sitemap',
    '@nuxt/content'
  ],

  site: {
    url: 'https://monsite.fr',
    name: 'Mon Site'
  },

  i18n: {
    locales: [
      { code: 'fr', language: 'fr-FR' },
      { code: 'en', language: 'en-US' }
    ],
    defaultLocale: 'fr',
    strategy: 'prefix_except_default'
  },

  sitemap: {
    zeroRuntime: true,
    discoverImages: true,
    
    urls: async () => {
      const posts = await fetch('https://api.cms.com/posts').then(r => r.json())
      return posts.map(p => ({
        loc: `/blog/${p.slug}`,
        lastmod: p.updatedAt,
        _i18nTransform: true
      }))
    },
    
    defaults: {
      changefreq: 'weekly',
      priority: 0.5
    },
    
    exclude: ['/admin/**', '/preview/**'],
    
    xslColumns: [
      { label: 'URL', width: '50%' },
      { label: 'Images', select: 'count(image:image)', width: '25%' },
      { label: 'Hreflangs', select: 'count(xhtml:link)', width: '25%' }
    ]
  },

  nitro: {
    preset: 'static',
    prerender: {
      crawlLinks: true,
      routes: ['/', '/sitemap.xml']
    }
  }
})
```

## Conclusion

La combinaison **@nuxtjs/sitemap v7.5.0**, **Nuxt 4.2.x SSG** et **Cloudflare Pages** offre une solution robuste pour le SEO technique. Les points essentiels à retenir :

Le mode `zeroRuntime: true` est indispensable pour SSG pur — il élimine le code runtime inutile et garantit la génération au build. L'intégration i18n est transparente avec détection automatique des locales, à condition de placer `@nuxtjs/i18n` avant `@nuxtjs/sitemap` dans les modules. Pour les URLs dynamiques sans server functions, privilégiez la fonction `urls` plutôt que les sources API. La découverte d'images nécessite que le contenu soit dans une balise `<main>` et que les routes soient prerendues. Enfin, utilisez explicitement `nitro.preset: 'static'` pour éviter que Cloudflare Pages n'active accidentellement le preset serverless.