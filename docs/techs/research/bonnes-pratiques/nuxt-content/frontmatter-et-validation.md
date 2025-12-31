# Validation frontmatter Nuxt Content 3.10+ avec Zod : guide complet

**La validation du frontmatter dans Nuxt Content 3.x repose sur des schemas Zod définis dans `content.config.ts`, offrant un typage TypeScript automatique et une intégration native avec Nuxt Studio.** Cette approche transforme la gestion du contenu Markdown en permettant une validation stricte au build time, des éditeurs visuels intelligents, et une compatibilité totale avec le déploiement SSG sur Cloudflare Pages. Ce guide couvre les configurations exactes pour Nuxt 4.2.x, @nuxt/content 3.10+, et Zod 4.

## Architecture des collections et schemas Zod

Le fichier **`content.config.ts`** à la racine du projet centralise toutes les définitions de collections. Chaque collection associe une source de fichiers (pattern glob) à un schema Zod qui valide et type le frontmatter.

```typescript
// content.config.ts
import { defineContentConfig, defineCollection, property } from '@nuxt/content'
import { z } from 'zod'  // Import direct depuis 'zod', pas depuis '@nuxt/content'

export default defineContentConfig({
  collections: {
    blog: defineCollection({
      type: 'page',           // 'page' génère des routes, 'data' pour données pures
      source: 'blog/**/*.md', // Pattern glob pour les fichiers
      schema: z.object({
        title: z.string().min(1).max(100),
        description: z.string().max(300).optional(),
        date: z.date(),
        updatedAt: z.date().optional(),
        author: z.string(),
        tags: z.array(z.string()).default([]),
        draft: z.boolean().default(false),
        category: z.enum(['Tech', 'Design', 'Tutorial', 'News']).optional(),
        image: z.object({
          src: property(z.string()).editor({ input: 'media' }),
          alt: z.string()
        }).optional()
      })
    })
  }
})
```

La propriété **`source`** accepte soit un string glob simple (`'blog/*.md'`), soit un objet avancé avec `include`, `exclude`, `prefix` et même `repository` pour du contenu distant Git. La propriété **`schema`** définit la structure exacte du frontmatter avec Zod, générant automatiquement les types TypeScript accessibles via `queryCollection()`.

L'import de Zod doit se faire directement depuis `'zod'` (v3) ou `'zod/v4'` (v4). **L'import depuis `@nuxt/content` est déprécié** et sera supprimé. Pour Zod v3, installer également `zod-to-json-schema` ; Zod v4 génère le JSON Schema nativement.

## Schema complet pour un blog multilingue

Voici une configuration production-ready intégrant tous les champs essentiels :

```typescript
// content.config.ts
import { defineContentConfig, defineCollection, property } from '@nuxt/content'
import { z } from 'zod/v4'
import { asSeoCollection } from '@nuxtjs/seo/content'

const imageSchema = z.object({
  src: property(z.string()).editor({ input: 'media' }),
  alt: z.string(),
  width: z.number().optional(),
  height: z.number().optional()
})

const seoSchema = z.object({
  title: z.string().max(60).optional(),
  description: z.string().max(160).optional(),
  image: z.string().optional(),
  canonical: z.string().url().optional(),
  noIndex: z.boolean().default(false)
})

const blogPostSchema = z.object({
  // Champs essentiels
  title: z.string().min(1).max(100),
  description: z.string().max(300).optional(),
  
  // Dates - z.date() recommandé, Nuxt Content gère la conversion ISO
  date: z.date(),
  updatedAt: z.date().optional(),
  
  // Auteur - référence vers collection ou inline
  author: z.string(),  // Slug référençant authors collection
  
  // Catégorisation
  category: z.enum(['Tech', 'Design', 'Tutorial', 'News']).optional(),
  tags: z.array(z.string()).default([]),
  
  // État
  draft: z.boolean().default(false),
  featured: z.boolean().default(false),
  
  // Multilingue
  locale: z.enum(['fr', 'en', 'es']).default('fr'),
  translationKey: z.string().optional(),  // Lier les traductions entre elles
  
  // Media avec éditeur Studio
  image: imageSchema.optional(),
  icon: property(z.string().optional()).editor({ 
    input: 'icon',
    iconLibraries: ['lucide', 'heroicons']
  }),
  
  // SEO intégré
  seo: seoSchema.optional()
}).refine(
  (data) => data.draft || data.date <= new Date(),
  { 
    message: 'La date de publication ne peut pas être future pour un article publié',
    path: ['date']
  }
)

export default defineContentConfig({
  collections: {
    blog: defineCollection(
      asSeoCollection({  // Wrapper pour intégration @nuxtjs/seo
        type: 'page',
        source: 'blog/**/*.md',
        schema: blogPostSchema
      })
    ),
    
    authors: defineCollection({
      type: 'data',
      source: 'authors/*.yml',
      schema: z.object({
        name: z.string(),
        slug: z.string(),
        avatar: imageSchema.optional(),
        bio: z.string().optional(),
        social: z.object({
          twitter: z.string().optional(),
          github: z.string().optional()
        }).optional()
      })
    })
  }
})
```

Les **champs optionnels** utilisent `.optional()` pour accepter `undefined`, tandis que `.default()` fournit une valeur par défaut si le champ est absent. **`.nullable()`** accepte explicitement `null`. La validation cross-champs via `.refine()` permet des règles métier complexes avec messages d'erreur personnalisés.

## Property editors pour Nuxt Studio

Les éditeurs de propriétés transforment le frontmatter en formulaires visuels dans Nuxt Studio. La fonction **`property()`** encapsule un schema Zod et expose la méthode `.editor()` pour configurer l'interface.

```typescript
import { property } from '@nuxt/content'

schema: z.object({
  // Media picker - ouvre la bibliothèque de médias
  coverImage: property(z.string()).editor({ input: 'media' }),
  
  // Icon picker - sélecteur d'icônes Iconify
  icon: property(z.string().optional()).editor({ 
    input: 'icon',
    iconLibraries: ['lucide', 'simple-icons', 'heroicons']
  }),
  
  // Champ masqué dans l'éditeur
  slug: property(z.string()).editor({ hidden: true }),
  
  // Héritage des props d'un composant Vue
  hero: property(z.object({})).inherit('components/HeroSection.vue')
})
```

Les **types d'inputs disponibles** sont `'media'` pour les fichiers et `'icon'` pour les icônes. Les types Zod primitifs sont automatiquement mappés : `z.string()` → input texte, `z.date()` → date picker, `z.boolean()` → toggle, `z.enum()` → select dropdown, `z.array(z.string())` → liste de badges.

## Intégration SEO et métadonnées

L'intégration SEO nécessite le module **@nuxtjs/seo** chargé **avant** @nuxt/content dans `nuxt.config.ts`. La fonction `asSeoCollection()` active les clés frontmatter SEO :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@nuxtjs/seo',      // DOIT être avant @nuxt/content
    '@nuxt/content',
    '@nuxtjs/i18n'
  ],
  
  site: {
    url: 'https://monblog.com',
    name: 'Mon Blog'
  },
  
  ogImage: {
    defaults: { width: 1200, height: 630, renderer: 'satori' }
  },
  
  schemaOrg: {
    identity: {
      type: 'Organization',
      name: 'Mon Blog',
      logo: 'https://monblog.com/logo.png'
    }
  }
})
```

Le frontmatter peut alors inclure des métadonnées structurées :

```yaml
---
title: "Mon article"
description: "Description SEO (max 160 caractères)"
image: "/images/cover.jpg"

ogImage:
  component: BlogOgImage
  props:
    title: "Mon article"
    readingMins: 5

schemaOrg:
  - "@type": "BlogPosting"
    headline: "Mon article"
    author:
      "@type": "Person"
      name: "Jean Dupont"
    datePublished: "2025-01-15"

sitemap:
  lastmod: 2025-01-16
  priority: 0.8
---
```

Les **dimensions d'images OG recommandées** sont **1200×630 px** (ratio 1.91:1), format JPG ou PNG, taille < 300 KB. Ces dimensions sont universelles pour Facebook, Twitter, et LinkedIn.

## Déploiement SSG sur Cloudflare Pages

Nuxt Content 3.x utilise une **base SQLite** au lieu du système de fichiers. En SSG, un dump SQL est généré et chargé côté client via WASM. Pour **Cloudflare Pages**, une base **D1 est obligatoire** :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      crawlLinks: true,
      routes: ['/sitemap.xml', '/robots.txt']
    }
  },
  
  content: {
    database: {
      type: 'd1',
      bindingName: 'DB'
    }
  }
})
```

La configuration D1 nécessite un fichier `wrangler.jsonc` :

```jsonc
{
  "d1_databases": [{
    "binding": "DB",
    "database_name": "content-db",
    "database_id": "votre-database-id"
  }]
}
```

**Points critiques pour Cloudflare** : désactiver Rocket Loader™ et Mirage dans le dashboard, définir `nitro.autoSubfolderIndex: false`, et créer la base D1 avant le premier déploiement.

## Validation et gestion d'erreurs

La validation Zod s'exécute au **build time** mais **ne fait pas échouer le build par défaut**. Les valeurs invalides sont simplement omises ou mises à `undefined`. Pour forcer l'échec :

```typescript
schema: z.object({
  title: z.string().min(1, "Le titre est obligatoire"),
  category: z.enum(['tech', 'news']).refine(
    val => val !== undefined,
    { message: "La catégorie doit être spécifiée" }
  )
})
```

En mode développement, les erreurs apparaissent en console avec HMR. En production, prévoir des fallbacks côté composant :

```vue
<script setup lang="ts">
const { data: page } = await useAsyncData('page', async () => {
  const content = await queryCollection('blog').path(route.path).first()
  return content
})
</script>

<template>
  <ContentRenderer v-if="page" :value="page" />
  <FallbackContent v-else />
</template>
```

## Structure multilingue avec @nuxtjs/i18n

Pour un blog multilingue, organiser le contenu par locale et créer des collections séparées :

```
content/
├── fr/
│   └── blog/
│       └── article-1.md
├── en/
│   └── blog/
│       └── post-1.md
```

```typescript
// content.config.ts
export default defineContentConfig({
  collections: {
    blogFr: defineCollection({
      type: 'page',
      source: { include: 'fr/blog/**/*.md', prefix: '/blog' },
      schema: blogPostSchema.extend({ locale: z.literal('fr').default('fr') })
    }),
    blogEn: defineCollection({
      type: 'page',
      source: { include: 'en/blog/**/*.md', prefix: '/en/blog' },
      schema: blogPostSchema.extend({ locale: z.literal('en').default('en') })
    })
  }
})
```

Les routes localisées sont générées automatiquement avec `crawlLinks: true`. Utiliser `useLocaleHead()` pour générer les tags `hreflang` et `og:locale`.

## Zod 4 versus Zod 3

**Zod 4** offre des améliorations significatives pour les builds Nuxt Content :

| Métrique | Zod 3 | Zod 4 |
|----------|-------|-------|
| Parsing strings | 1x | **14x plus rapide** |
| Bundle (gzip) | 12.47 KB | **5.36 KB** |
| zod/mini | — | **1.88 KB** |
| JSON Schema | Via adaptateur | **Natif** |

L'import change légèrement : `import { z } from 'zod/v4'`. Les fonctionnalités clés de Zod 4 incluent les formats string au niveau racine (`z.email()`, `z.uuid()`), les métadonnées typées (`.meta()`), et l'internationalisation native (`z.config(fr())`).

## Conclusion

La validation frontmatter avec Nuxt Content 3.10+ et Zod représente une évolution majeure vers un CMS headless typé de bout en bout. Les points essentiels à retenir : **définir les schemas dans `content.config.ts`** avec import direct de Zod, **utiliser `property().editor()`** pour Nuxt Studio, **wrapper avec `asSeoCollection()`** pour le SEO, et **configurer D1** pour Cloudflare Pages. Cette architecture garantit un contenu validé au build time, des types TypeScript inférés automatiquement, et une expérience d'édition visuelle pour les non-développeurs.