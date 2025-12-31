# Pattern 3 : Discriminated unions pour types de contenu

Les discriminated unions utilisent un champ discriminant (ex: `type`) pour un lookup **O(1)** au lieu de **O(n)** avec `z.union()`. Elles permettent aussi le type narrowing automatique en TypeScript.

## Schema de base avec discriminant

```typescript
import { z } from 'zod/v4'

// Base partagée entre tous les types de contenu
const BaseArticle = {
  title: z.string().min(1).max(100),
  slug: z.string(),
  draft: z.boolean().default(false),
  publishedAt: z.iso.date(),
  tags: z.array(z.string()).default([])
}

// Schemas avec discriminant `type`
const ArticleSchema = z.object({
  ...BaseArticle,
  type: z.literal('article'),
  description: z.string(),
  readingTime: z.number().positive()
})

const TutorialSchema = z.object({
  ...BaseArticle,
  type: z.literal('tutorial'),
  difficulty: z.enum(['beginner', 'intermediate', 'advanced']),
  prerequisites: z.array(z.string()).optional()
})

const NoteSchema = z.object({
  ...BaseArticle,
  type: z.literal('note'),
  body: z.string()
})

// Discriminated union - lookup O(1) par le champ 'type'
const ContentSchema = z.discriminatedUnion('type', [
  ArticleSchema,
  TutorialSchema,
  NoteSchema
])

type Content = z.infer<typeof ContentSchema>
```

## Type narrowing automatique

```typescript
function processContent(content: Content) {
  switch (content.type) {
    case 'article':
      // TypeScript sait que content.description existe
      console.log(content.description, content.readingTime)
      break
    case 'tutorial':
      // TypeScript sait que content.difficulty existe
      console.log(`${content.difficulty}: ${content.prerequisites?.length ?? 0} prérequis`)
      break
    case 'note':
      // TypeScript sait que content.body existe
      console.log(content.body)
      break
  }
}
```

## Composition de discriminated unions

```typescript
// Unions partielles
const BlogContent = z.discriminatedUnion('type', [ArticleSchema, NoteSchema])
const TechContent = z.discriminatedUnion('type', [TutorialSchema])

// Combiner via .options
const AllContent = z.discriminatedUnion('type', [
  ...BlogContent.options,
  ...TechContent.options
])
```

## Performance : discriminatedUnion vs union

| Aspect | `z.union()` | `z.discriminatedUnion()` |
|--------|-------------|-------------------------|
| **Validation** | Essaie chaque schema séquentiellement O(n) | Lookup par discriminant O(1) |
| **Messages d'erreur** | "Invalid union" générique | Spécifique au discriminant |
| **Type narrowing TS** | Type guards manuels requis | Automatique via discriminant |
| **Cas d'usage** | Types hétérogènes | Objets tagués/typés |

## Intégration Nuxt Content

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { asSeoCollection } from '@nuxtjs/seo/content'
import { z } from 'zod/v4'

// ... définition des schemas ci-dessus ...

export default defineContentConfig({
  collections: {
    articles_fr: defineCollection(
      asSeoCollection({
        type: 'page',
        source: { include: 'fr/**/*.md', prefix: '/blog' },
        schema: ContentSchema,  // Discriminated union
      })
    ),
    articles_en: defineCollection(
      asSeoCollection({
        type: 'page',
        source: { include: 'en/**/*.md', prefix: '/blog' },
        schema: ContentSchema,
      })
    ),
  }
})
```

---
