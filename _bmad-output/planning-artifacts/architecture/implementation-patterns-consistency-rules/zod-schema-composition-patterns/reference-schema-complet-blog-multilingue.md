# Référence : Schema complet blog multilingue

```typescript
// lib/content-schemas.ts
import { z } from 'zod/v4'

// Mixins réutilisables
const TimestampFields = {
  publishedAt: z.iso.date(),
  updatedAt: z.iso.date().optional()
}

const SEOFields = {
  description: z.string().max(160).optional(),
  ogImage: z.string().optional()
}

// Base pour tout le contenu blog
const BaseBlog = z.object({
  title: z.string().min(1).max(100),
  slug: z.string(),
  draft: z.boolean().default(false),
  author: z.string(),
  tags: z.array(z.string()).default([]),
  ...TimestampFields,
  ...SEOFields
})

// Schemas par type de contenu
export const ArticleSchema = z.object({
  ...BaseBlog.shape,
  type: z.literal('article'),
  pillar: z.enum(['ai', 'engineering', 'ux']),
  category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
  level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),
  readingTime: z.number().positive()
})

export const TutorialSchema = z.object({
  ...BaseBlog.shape,
  type: z.literal('tutorial'),
  difficulty: z.enum(['beginner', 'intermediate', 'advanced']),
  prerequisites: z.array(z.string()).optional(),
  estimatedTime: z.string().optional()
})

export const NoteSchema = z.object({
  ...BaseBlog.shape,
  type: z.literal('note')
})

// Discriminated union principale
export const BlogContentSchema = z.discriminatedUnion('type', [
  ArticleSchema,
  TutorialSchema,
  NoteSchema
])

// Types exportés
export type Article = z.infer<typeof ArticleSchema>
export type Tutorial = z.infer<typeof TutorialSchema>
export type Note = z.infer<typeof NoteSchema>
export type BlogContent = z.infer<typeof BlogContentSchema>

// Schemas dérivés pour différents usages
export const ArticlePreview = ArticleSchema.pick({
  title: true,
  slug: true,
  description: true,
  publishedAt: true,
  pillar: true,
  tags: true
})

export const ArticleForm = ArticleSchema.omit({
  type: true,
  slug: true,
  readingTime: true
})
```

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { asSeoCollection } from '@nuxtjs/seo/content'
import { BlogContentSchema } from './lib/content-schemas'

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
      asSeoCollection({
        type: 'page',
        source: { include: 'fr/**/*.md', prefix: '/blog' },
        schema: BlogContentSchema,
        indexes: articleIndexes,
      })
    ),
    articles_en: defineCollection(
      asSeoCollection({
        type: 'page',
        source: { include: 'en/**/*.md', prefix: '/blog' },
        schema: BlogContentSchema,
        indexes: articleIndexes,
      })
    ),
  }
})
```
