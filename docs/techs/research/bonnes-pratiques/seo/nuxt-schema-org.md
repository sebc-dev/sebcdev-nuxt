# Implémentation Schema.org avec nuxt-schema-org v5.0.10 pour Nuxt 4 SSG

La mise en place efficace des données structurées Schema.org dans un blog Nuxt 4 avec nuxt-schema-org v5.0.10 nécessite une configuration précise tenant compte des évolutions récentes de Google (FAQ rich results restreints depuis août 2023, Sitelinks Searchbox supprimé en novembre 2024). Ce guide détaille les bonnes pratiques pour votre stack technique spécifique avec Nuxt Content 3, i18n, et déploiement SSG sur Cloudflare Pages.

---

## Configuration globale dans nuxt.config.ts

La configuration optimale pour nuxt-schema-org v5.0.10 s'appuie sur le **Nuxt Site Config** pour centraliser les métadonnées du site. L'ordre des modules est critique : `@nuxtjs/seo` (ou `nuxt-schema-org`) doit précéder `@nuxt/content` dans le tableau modules.

```typescript
// nuxt.config.ts
import { definePerson } from 'nuxt-schema-org/schema'

export default defineNuxtConfig({
  future: { compatibilityVersion: 4 },
  
  modules: [
    'nuxt-schema-org',    // DOIT être avant @nuxt/content
    '@nuxt/content',
    '@nuxtjs/i18n'
  ],
  
  // Configuration centralisée du site
  site: {
    url: 'https://votresite.com',
    name: 'Nom du Blog',
    description: 'Description du site',
    defaultLocale: 'fr-FR',
  },
  
  schemaOrg: {
    // Identité du propriétaire (Person pour blog personnel)
    identity: definePerson({
      name: 'Votre Nom',
      image: '/profile.jpg',
      jobTitle: 'Développeur Senior',
      sameAs: [
        'https://github.com/votreprofil',
        'https://linkedin.com/in/votreprofil',
        'https://twitter.com/votreprofil'
      ]
    }),
    
    // Activer les schémas WebSite et WebPage par défaut
    defaults: true,
    
    // Minification en production
    minify: true,
    
    // Désactiver la réactivité côté client (SSG)
    reactive: false,
  },
  
  // Configuration i18n
  i18n: {
    baseUrl: 'https://votresite.com',
    defaultLocale: 'fr',
    strategy: 'prefix_except_default',
    locales: [
      { code: 'fr', language: 'fr-FR', name: 'Français' },
      { code: 'en', language: 'en-US', name: 'English' }
    ],
  },
  
  // Configuration SSG pour Cloudflare Pages
  routeRules: {
    '/blog/**': { prerender: true },
  },
})
```

Le module infère automatiquement `inLanguage` depuis `@nuxtjs/i18n`, résolvant les URLs canoniques localisées. La propriété `reactive: false` optimise les performances en SSG en évitant le code de réactivité côté client inutile.

---

## WebSite Schema avec support multilingue

Le schéma WebSite s'établit idéalement dans `app/app.vue` ou un layout global. Le module génère automatiquement les valeurs par défaut depuis `site:` config, mais vous pouvez les enrichir.

```typescript
// app/app.vue
<script lang="ts" setup>
useSchemaOrg([
  defineWebSite({
    name: 'Mon Blog Tech',
    description: 'Articles techniques sur Vue, Nuxt et TypeScript',
    // inLanguage est auto-rempli par @nuxtjs/i18n
    
    // NOTE: SearchAction n'est plus utile depuis novembre 2024
    // Google a supprimé le Sitelinks Search Box
  }),
  defineWebPage()  // Schéma de base pour toutes les pages
])
</script>
```

**Important concernant SearchAction** : Google a officiellement supprimé la fonctionnalité Sitelinks Search Box le **21 novembre 2024**. Bien que vous puissiez techniquement ajouter un `potentialAction` avec Pagefind, cela ne générera plus de rich snippet dans les résultats de recherche. L'implémentation reste optionnelle pour la sémantique :

```typescript
// Optionnel - n'affecte plus les SERPs
potentialAction: [
  defineSearchAction({
    target: '/search?q={search_term_string}'
  })
]
```

---

## Article et TechArticle pour le contenu technique

La distinction entre `Article` et `TechArticle` est sémantique : utilisez **TechArticle** pour les tutoriels, guides how-to, documentations techniques et troubleshooting. TechArticle hérite toutes les propriétés d'Article avec deux ajouts spécifiques : `dependencies` (prérequis) et `proficiencyLevel` ('Beginner' ou 'Expert').

### Intégration avec Nuxt Content 3

Configurez d'abord la collection avec `asSchemaOrgCollection()` :

```typescript
// content.config.ts
import { defineCollection, defineContentConfig, z } from '@nuxt/content'
import { asSchemaOrgCollection } from 'nuxt-schema-org/content'

export default defineContentConfig({
  collections: {
    blog: defineCollection(
      asSchemaOrgCollection({
        type: 'page',
        source: 'blog/**/*.md',
        schema: z.object({
          title: z.string(),
          description: z.string(),
          publishedAt: z.date(),
          updatedAt: z.date().optional(),
          author: z.string().default('Votre Nom'),
          image: z.string().optional(),
          tags: z.array(z.string()).optional(),
        })
      })
    ),
  },
})
```

### Page de contenu avec defineArticle()

```vue
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
const route = useRoute()
const { locale } = useI18n()

const { data: article } = await useAsyncData(`article-${route.path}`, () =>
  queryCollection('blog').path(route.path).first()
)

// CRITIQUE: useHead() pour rendre le schema depuis le frontmatter
useHead(article.value?.head || {})

// Schema Article enrichi programmatiquement
useSchemaOrg([
  defineArticle({
    '@type': 'TechArticle',  // Résulte en ['Article', 'TechArticle']
    
    // Propriétés recommandées par Google
    headline: article.value?.title,
    description: article.value?.description,
    datePublished: article.value?.publishedAt,
    dateModified: article.value?.updatedAt || article.value?.publishedAt,
    
    // Images: Google recommande 3 ratios d'aspect
    image: article.value?.image ? [
      `https://votresite.com${article.value.image}`,
      // Idéalement fournir 16:9, 4:3, 1:1
    ] : undefined,
    
    // Auteur (référence l'identité globale si non spécifié)
    author: {
      '@type': 'Person',
      name: article.value?.author,
      url: 'https://votresite.com/a-propos'
    },
    
    // Propriétés TechArticle spécifiques
    proficiencyLevel: 'Expert',
    
    // Métadonnées additionnelles
    articleSection: article.value?.tags?.[0],
    keywords: article.value?.tags,
    inLanguage: locale.value,
    wordCount: article.value?.body?.split(/\s+/).length,
  })
])
</script>
```

### Frontmatter Markdown recommandé

```yaml
---
title: "Implémenter Schema.org avec Nuxt 4"
description: "Guide complet pour les données structurées"
publishedAt: 2024-12-15
updatedAt: 2024-12-28
author: "Votre Nom"
image: "/images/blog/schema-org-cover.jpg"
tags: ["nuxt", "seo", "schema-org"]
---
```

**Formats de date** : Utilisez toujours le format **ISO 8601** avec timezone (ex: `2024-12-28T10:00:00+01:00`). Sans timezone, Google utilise celle de Googlebot.

---

## BreadcrumbList pour la navigation

Les breadcrumbs améliorent la compréhension de la structure du site par Google. Le composable `defineBreadcrumb()` génère automatiquement les positions.

### Génération automatique basée sur la route

```typescript
// app/composables/useBreadcrumbSchema.ts
export function useBreadcrumbSchema() {
  const route = useRoute()
  const { t, locale } = useI18n()
  
  const segments = route.path.split('/').filter(Boolean)
  const items = [
    { name: t('nav.home'), item: '/' }
  ]
  
  let currentPath = ''
  segments.forEach((segment, index) => {
    currentPath += `/${segment}`
    const isLast = index === segments.length - 1
    
    items.push({
      name: formatSegmentName(segment),
      // Pas de 'item' pour le dernier élément (convention Google)
      ...(isLast ? {} : { item: currentPath })
    })
  })
  
  useSchemaOrg([
    defineBreadcrumb({
      itemListElement: items
    })
  ])
}
```

### Implémentation dans une page article

```vue
<script setup lang="ts">
useSchemaOrg([
  defineBreadcrumb({
    itemListElement: [
      { name: 'Accueil', item: '/' },
      { name: 'Blog', item: '/blog' },
      { name: article.value?.title }  // Pas d'item = page courante
    ]
  })
])
</script>
```

**Exigences Google** : Au minimum **deux ListItems** pour l'éligibilité aux rich results. La propriété `position` est auto-calculée par le module.

---

## Person Schema pour l'identité auteur

Le schéma Person renforce l'E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness) en établissant clairement l'identité de l'auteur.

### Configuration globale dans nuxt.config.ts

```typescript
schemaOrg: {
  identity: definePerson({
    name: 'Jean Dupont',
    givenName: 'Jean',
    familyName: 'Dupont',
    
    image: '/images/profile.jpg',
    description: 'Développeur full-stack spécialisé Vue.js et Nuxt',
    jobTitle: 'Lead Developer',
    
    // URLs équivalentes (social proof)
    sameAs: [
      'https://github.com/jeandupont',
      'https://linkedin.com/in/jeandupont',
      'https://twitter.com/jeandupont',
      'https://dev.to/jeandupont'
    ],
    
    url: 'https://votresite.com/a-propos',
    
    // Expertise technique
    knowsAbout: [
      { '@type': 'Thing', name: 'Vue.js', sameAs: 'https://en.wikipedia.org/wiki/Vue.js' },
      { '@type': 'Thing', name: 'Nuxt', sameAs: 'https://en.wikipedia.org/wiki/Nuxt.js' },
      'TypeScript',
      'Node.js',
      'Cloud Architecture'
    ],
    
    worksFor: {
      '@type': 'Organization',
      name: 'Votre Entreprise',
      url: 'https://entreprise.com'
    }
  })
}
```

**La relation Person-Article** est automatiquement établie par le module : si `author` n'est pas spécifié dans `defineArticle()`, l'identité globale est utilisée.

---

## FAQ Schema : limitations actuelles

**Mise à jour critique (août 2023)** : Les FAQ rich results sont désormais **limités aux sites gouvernementaux et de santé reconnus**. Pour les blogs techniques, le markup FAQPage ne générera plus de rich snippets dans Google.

Cependant, l'implémentation reste utile pour la sémantique et d'autres moteurs de recherche :

```vue
<script setup lang="ts">
// Pour une page FAQ dédiée
useSchemaOrg([
  defineWebPage({
    '@type': 'FAQPage'  // Résulte en ['WebPage', 'FAQPage']
  }),
  defineQuestion({
    name: 'Comment installer nuxt-schema-org ?',
    acceptedAnswer: 'Exécutez npx nuxt module add schema-org dans votre projet Nuxt.',
    inLanguage: 'fr-FR'
  }),
  defineQuestion({
    name: 'Le module est-il compatible Nuxt 4 ?',
    acceptedAnswer: 'Oui, nuxt-schema-org v5.0.10 supporte compatibilityVersion: 4.'
  })
])
</script>
```

### Extraction FAQ depuis Markdown

```typescript
// app/composables/useFaqFromContent.ts
export function useFaqFromContent(content: string) {
  // Regex pour extraire les patterns Q&A du markdown
  const faqRegex = /##\s*Q:\s*(.+?)\n\n(.+?)(?=\n##|$)/gs
  const faqs = []
  let match
  
  while ((match = faqRegex.exec(content)) !== null) {
    faqs.push({
      question: match[1].trim(),
      answer: match[2].trim()
    })
  }
  
  if (faqs.length > 0) {
    useSchemaOrg([
      defineWebPage({ '@type': 'FAQPage' }),
      ...faqs.map(faq => defineQuestion({
        name: faq.question,
        acceptedAnswer: faq.answer
      }))
    ])
  }
}
```

---

## Spécificités SSG et Cloudflare Pages

Le mode SSG génère les schémas Schema.org au **build time**, intégrés directement dans le HTML statique. Cette approche est optimale pour le SEO car le contenu est immédiatement disponible sans JavaScript.

### Configuration optimale pour SSG

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true,  // Requis pour SSG
  
  schemaOrg: {
    reactive: false,  // Désactiver la réactivité client (inutile en SSG)
  },
  
  nitro: {
    prerender: {
      crawlLinks: true,
      routes: ['/'],
    },
  },
  
  // Pour Cloudflare Pages avec nombreuses routes
  routeRules: {
    '/blog/**': { prerender: true },
  },
})
```

### Déploiement Cloudflare Pages

```bash
# Génération statique
npm run generate

# Le dossier .output/public contient le HTML avec Schema.org pré-rendu
```

**Limite Cloudflare** : Maximum **100 routes exclues** dans la configuration. Pour les blogs volumineux :

```typescript
nitro: {
  cloudflare: {
    pages: {
      routes: {
        exclude: ['/blog/*', '/docs/*']
      }
    }
  }
}
```

---

## Validation et debugging

### Outils de validation

- **Google Rich Results Test** : https://search.google.com/test/rich-results — Validation des rich snippets Google
- **Schema Markup Validator** : https://validator.schema.org — Validation syntaxique Schema.org
- **Nuxt DevTools** : Onglet Schema.org intégré en développement

### Endpoint de debug

En développement, accédez à `/__schema-org__/debug.json` pour visualiser le schéma généré.

### Bonnes pratiques de validation

```typescript
// Vérifier la structure générée
if (process.dev) {
  useHead({
    script: [{
      type: 'application/ld+json',
      innerHTML: JSON.stringify(schemaData, null, 2),
      'data-debug': 'schema-org'
    }]
  })
}
```

---

## Récapitulatif des changements Google 2024-2025

| Fonctionnalité | Statut | Date | Impact |
|----------------|--------|------|--------|
| **Sitelinks Search Box** | Supprimé | Nov 2024 | SearchAction n'affecte plus les SERPs |
| **FAQ Rich Results** | Limité | Août 2023 | Réservé aux sites gov/santé |
| **How-To Rich Results** | Supprimé | 2023 | Plus de rich snippets pour tutoriels |
| **Article author.url** | Ajouté | 2024 | Améliore la désambiguïsation des auteurs |

---

## Conclusion

L'implémentation de Schema.org avec nuxt-schema-org v5.0.10 dans Nuxt 4 SSG repose sur trois piliers : une **configuration centralisée** via `site:` et `schemaOrg.identity`, une **intégration Nuxt Content 3** via `asSchemaOrgCollection()`, et le respect des **exigences Google actualisées** (formats de dates ISO 8601, images multi-ratios, author markup structuré).

Les points d'attention spécifiques : désactiver `reactive` en SSG pour les performances, placer `nuxt-schema-org` avant `@nuxt/content` dans les modules, et accepter que FAQ et SearchAction ne génèrent plus de rich snippets pour les blogs techniques. L'investissement dans les schémas Article, BreadcrumbList et Person reste pleinement pertinent pour le SEO et la visibilité dans les résultats de recherche enrichis.