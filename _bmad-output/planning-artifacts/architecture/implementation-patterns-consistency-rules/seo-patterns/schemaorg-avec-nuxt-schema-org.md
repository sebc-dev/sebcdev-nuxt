# Schema.org avec nuxt-schema-org

## Schema Article pour blog posts

```typescript
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
import { defineArticle, useSchemaOrg } from '#imports'

const siteConfig = useSiteConfig()

// ... fetch post data ...

useSchemaOrg([
  defineArticle({
    '@type': 'BlogPosting',
    headline: () => post.value?.title,
    description: () => post.value?.description,
    image: () => absoluteImageUrl.value,
    datePublished: () => post.value?.publishedAt
      ? new Date(post.value.publishedAt).toISOString()
      : undefined,
    dateModified: () => post.value?.updatedAt
      ? new Date(post.value.updatedAt).toISOString()
      : undefined,
    author: {
      '@type': 'Person',
      name: 'Sébastien C.',
      url: siteConfig.url
    },
    publisher: {
      '@type': 'Organization',
      name: 'sebc.dev',
      logo: `${siteConfig.url}/logo.png`
    },
    keywords: () => post.value?.tags,
    articleSection: () => post.value?.pillar,
    inLanguage: () => locale.value === 'fr' ? 'fr' : 'en'
  })
])
</script>
```

## TechArticle pour contenu technique

La distinction entre `Article` et `TechArticle` est sémantique. Utilisez **TechArticle** pour :
- Tutoriels et guides how-to
- Documentation technique
- Troubleshooting et debugging

TechArticle hérite toutes les propriétés d'Article avec deux ajouts spécifiques :

| Propriété | Valeurs | Usage |
|-----------|---------|-------|
| `proficiencyLevel` | `'Beginner'` \| `'Expert'` | Niveau requis du lecteur |
| `dependencies` | `string[]` | Prérequis techniques |

```typescript
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
useSchemaOrg([
  defineArticle({
    '@type': 'TechArticle',  // Résulte en ['Article', 'TechArticle']

    headline: () => post.value?.title,
    description: () => post.value?.description,
    datePublished: () => post.value?.publishedAt
      ? new Date(post.value.publishedAt).toISOString()
      : undefined,
    dateModified: () => post.value?.updatedAt
      ? new Date(post.value.updatedAt).toISOString()
      : undefined,

    // Propriétés TechArticle spécifiques
    proficiencyLevel: () => {
      const level = post.value?.level
      return level === 'beginner' ? 'Beginner' : 'Expert'
    },

    // Métadonnées additionnelles
    articleSection: () => post.value?.pillar,
    keywords: () => post.value?.tags,
    inLanguage: () => locale.value,
    wordCount: () => post.value?.body?.split(/\s+/).length,
  })
])
</script>
```

**Note format de date** : Utilisez toujours le format **ISO 8601 avec timezone** (ex: `2025-12-28T10:00:00+01:00`). Sans timezone, Google utilise celle de Googlebot.

## Layout avec WebSite schema

```typescript
<!-- app/layouts/default.vue -->
<script setup lang="ts">
import { defineWebSite, defineWebPage, useSchemaOrg } from '#imports'

const siteConfig = useSiteConfig()
const route = useRoute()

useSchemaOrg([
  defineWebSite({
    name: 'sebc.dev',
    description: 'Blog technique sur le développement web',
    url: siteConfig.url,
    inLanguage: ['fr', 'en']
  }),
  defineWebPage({
    '@id': `${siteConfig.url}${route.path}`,
    url: `${siteConfig.url}${route.path}`
  })
])
</script>
```
