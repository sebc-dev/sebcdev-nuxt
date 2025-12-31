# Zod 4 schema composition for Nuxt Content 3.10+ projects

**Zod 4 fundamentally changes schema composition patterns**, deprecating `.merge()` entirely and introducing `.safeExtend()` for refined schemas. For your Nuxt 4 / Nuxt Content 3.10+ blog with language-separated collections, the recommended approach combines spread-syntax composition for TypeScript performance, discriminated unions for content type differentiation, and shared base schemas across `blog_fr` and `blog_en` collections. The native `z.toJSONSchema()` in Zod 4 eliminates the need for `zod-to-json-schema` adapter packages.

---

## Schema extension patterns in Zod 4

The `.extend()` method remains the primary composition tool, but Zod 4 adds `.safeExtend()` for schemas with refinements and recommends object spread syntax for optimal TypeScript compiler performance.

### Basic extension syntax

```typescript
import { z } from 'zod'

const BaseContent = z.object({
  title: z.string(),
  slug: z.string(),
  date: z.date(),
  draft: z.boolean().default(false)
})

// Standard .extend() - creates new ZodObject with combined shape
const Article = BaseContent.extend({
  description: z.string(),
  tags: z.array(z.string()).optional(),
  readingTime: z.number().positive()
})

type Article = z.infer<typeof Article>
// { title: string; slug: string; date: Date; draft: boolean; 
//   description: string; tags?: string[]; readingTime: number }
```

### Spread syntax for best TypeScript performance

Chaining multiple `.extend()` calls causes **quadratic TypeScript instantiations**. Zod 4 documentation recommends spread syntax for complex compositions:

```typescript
// ✅ BEST: Object spread (100x fewer tsc instantiations)
const TechArticle = z.object({
  ...BaseContent.shape,
  ...SEOFields.shape,
  codeLanguage: z.string().optional(),
  repository: z.string().url().optional()
})

// ❌ AVOID: Chained extend calls (expensive for tsc)
const TechArticle = BaseContent
  .extend(SEOFields.shape)
  .extend({ codeLanguage: z.string().optional() })
  .extend({ repository: z.string().url().optional() })
```

### The new `.safeExtend()` method (Zod 4.1+)

Regular `.extend()` **throws an error** when used on schemas with refinements. Use `.safeExtend()` to preserve refinements:

```typescript
const ValidatedContent = z.object({
  title: z.string(),
  publishDate: z.date(),
  expiryDate: z.date().optional()
}).refine(
  data => !data.expiryDate || data.expiryDate > data.publishDate,
  'Expiry date must be after publish date'
)

// ❌ THROWS: Regular extend on refined schema
const Extended = ValidatedContent.extend({ author: z.string() })

// ✅ WORKS: safeExtend preserves refinements
const Extended = ValidatedContent.safeExtend({ 
  author: z.string() 
})

// Type narrowing also enforced - prevents incompatible overwrites
ValidatedContent.safeExtend({ title: z.string().min(10) }) // ✅ OK
ValidatedContent.safeExtend({ title: z.number() })          // ❌ Error
```

---

## Schema merging is deprecated—use extend instead

**Breaking change**: `.merge()` is deprecated in Zod 4 due to ambiguity around strictness inheritance and inferior TypeScript performance.

### Migration from Zod 3

```typescript
// ❌ ZOD 3 (deprecated)
const Combined = SchemaA.merge(SchemaB)

// ✅ ZOD 4 Option 1: extend with .shape
const Combined = SchemaA.extend(SchemaB.shape)

// ✅ ZOD 4 Option 2: spread syntax (best performance)
const Combined = z.object({
  ...SchemaA.shape,
  ...SchemaB.shape
})
```

### Conflict resolution behavior

When both schemas share keys, **the second schema's fields take precedence**:

```typescript
const PersonalInfo = z.object({
  name: z.string(),
  email: z.string()  // less strict
})

const ValidatedInfo = z.object({
  email: z.string().email()  // more strict
})

// email uses ValidatedInfo's stricter validation
const Combined = z.object({
  ...PersonalInfo.shape,
  ...ValidatedInfo.shape
})
```

---

## Pick and omit for partial schema extraction

These methods create new `ZodObject` instances with filtered shapes. **Refinements are NOT preserved** when using `.pick()` or `.omit()`.

### Syntax and use cases

```typescript
const FullArticle = z.object({
  id: z.string().uuid(),
  title: z.string(),
  content: z.string(),
  author: z.string(),
  createdAt: z.date(),
  updatedAt: z.date()
})

// Pick specific fields for API responses
const ArticlePreview = FullArticle.pick({ 
  id: true, 
  title: true, 
  author: true 
})
// { id: string; title: string; author: string }

// Omit sensitive/internal fields
const PublicArticle = FullArticle.omit({ 
  id: true,
  createdAt: true,
  updatedAt: true
})

// Create form schemas by omitting auto-generated fields
const ArticleForm = FullArticle.omit({ 
  id: true, 
  createdAt: true, 
  updatedAt: true 
})
```

### Integration with TypeScript utility types

Zod's `.pick()` and `.omit()` mirror TypeScript's `Pick<>` and `Omit<>`, maintaining type safety:

```typescript
type FullArticle = z.infer<typeof FullArticle>
type ArticlePreview = z.infer<typeof ArticlePreview>

// These are equivalent:
type ManualPick = Pick<FullArticle, 'id' | 'title' | 'author'>
// ArticlePreview === ManualPick ✓
```

### Combining with partial for forms

```typescript
// Create an update schema where all fields are optional
const ArticleUpdate = FullArticle
  .omit({ id: true, createdAt: true })
  .partial()  // All remaining fields become optional

type ArticleUpdate = z.infer<typeof ArticleUpdate>
// { title?: string; content?: string; author?: string; updatedAt?: Date }

// Partial with specific fields only
const PartialTitle = FullArticle.partial({ title: true })
// { id: string; title?: string; content: string; ... }
```

---

## Discriminated unions for content type differentiation

Zod 4 significantly enhances discriminated unions with **nested composition support**, **union discriminators**, and **better error messages**. Use these for differentiating Article, Tutorial, Note, and TechArticle content types.

### Basic discriminated union pattern

```typescript
const Article = z.object({
  type: z.literal('article'),
  title: z.string(),
  description: z.string(),
  content: z.string()
})

const Tutorial = z.object({
  type: z.literal('tutorial'),
  title: z.string(),
  steps: z.array(z.string()),
  difficulty: z.enum(['beginner', 'intermediate', 'advanced'])
})

const Note = z.object({
  type: z.literal('note'),
  title: z.string(),
  body: z.string(),
  quick: z.boolean().default(true)
})

const TechArticle = z.object({
  type: z.literal('tech-article'),
  title: z.string(),
  description: z.string(),
  codeLanguage: z.string(),
  repository: z.string().url().optional()
})

// Discriminated union uses 'type' field for O(1) schema lookup
const ContentSchema = z.discriminatedUnion('type', [
  Article,
  Tutorial,
  Note,
  TechArticle
])

type Content = z.infer<typeof ContentSchema>
```

### Type narrowing in TypeScript

```typescript
function processContent(content: Content) {
  switch (content.type) {
    case 'article':
      // TypeScript knows: content.description exists
      console.log(content.description)
      break
    case 'tutorial':
      // TypeScript knows: content.steps and content.difficulty exist
      console.log(`${content.difficulty}: ${content.steps.length} steps`)
      break
    case 'note':
      // TypeScript knows: content.body and content.quick exist
      console.log(content.body)
      break
    case 'tech-article':
      // TypeScript knows: content.codeLanguage exists
      console.log(`Language: ${content.codeLanguage}`)
      break
  }
}
```

### Zod 4 enhancements: nested and union discriminators

```typescript
// NEW IN ZOD 4: Union discriminators
const StatusSchema = z.discriminatedUnion('status', [
  z.object({ status: z.literal('draft'), editedAt: z.date() }),
  // Union of literals as discriminator
  z.object({ 
    status: z.union([z.literal('published'), z.literal('archived')]),
    publishedAt: z.date()
  })
])

// NEW IN ZOD 4: Nested discriminated unions
const ErrorSchema = z.object({
  status: z.literal('error'),
  message: z.string()
})

const ApiResponse = z.discriminatedUnion('status', [
  z.object({ status: z.literal('success'), data: z.unknown() }),
  // Nested discriminated union
  z.discriminatedUnion('code', [
    ErrorSchema.extend({ code: z.literal(400) }),
    ErrorSchema.extend({ code: z.literal(401) }),
    ErrorSchema.extend({ code: z.literal(500) })
  ])
])
```

### Composing discriminated unions

```typescript
const BlogContent = z.discriminatedUnion('type', [Article, Note])
const TechContent = z.discriminatedUnion('type', [Tutorial, TechArticle])

// Combine using .options property
const AllContent = z.discriminatedUnion('type', [
  ...BlogContent.options,
  ...TechContent.options
])
```

### Performance: discriminatedUnion vs regular union

| Aspect | `z.union()` | `z.discriminatedUnion()` |
|--------|-------------|-------------------------|
| **Validation** | Tries each schema sequentially (O(n)) | Uses discriminator for O(1) lookup |
| **Error messages** | Generic "invalid union" | Specific to matched discriminator |
| **TypeScript narrowing** | Manual type guards needed | Automatic via discriminator |
| **Use case** | Heterogeneous types | Tagged/typed objects |

---

## Nuxt Content 3.10+ integration patterns

### Basic collection schema setup

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { z } from 'zod'  // ⚠️ Import directly from 'zod', NOT from '@nuxt/content'

export default defineContentConfig({
  collections: {
    blog: defineCollection({
      type: 'page',
      source: 'blog/*.md',
      schema: z.object({
        title: z.string(),
        description: z.string().optional(),
        date: z.date(),
        draft: z.boolean().default(false),
        tags: z.array(z.string()).optional()
      })
    })
  }
})
```

### Shared schemas for language-separated collections

```typescript
// lib/content-schemas.ts
import { z } from 'zod'

// Base schema for all blog content
export const BaseBlogSchema = z.object({
  title: z.string(),
  description: z.string().optional(),
  date: z.date(),
  draft: z.boolean().default(false),
  author: z.string(),
  tags: z.array(z.string()).optional(),
  image: z.string().optional()
})

// Extended schemas for specific content types
export const ArticleSchema = z.object({
  ...BaseBlogSchema.shape,
  type: z.literal('article'),
  readingTime: z.number().positive()
})

export const TutorialSchema = z.object({
  ...BaseBlogSchema.shape,
  type: z.literal('tutorial'),
  difficulty: z.enum(['beginner', 'intermediate', 'advanced']),
  prerequisites: z.array(z.string()).optional()
})

export const NoteSchema = z.object({
  ...BaseBlogSchema.shape,
  type: z.literal('note')
})

// Discriminated union for all content types
export const BlogContentSchema = z.discriminatedUnion('type', [
  ArticleSchema,
  TutorialSchema,
  NoteSchema
])
```

```typescript
// content.config.ts
import { defineContentConfig, defineCollection, property } from '@nuxt/content'
import { BlogContentSchema } from './lib/content-schemas'

export default defineContentConfig({
  collections: {
    // English content
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'en/blog/**/*.md', prefix: '/blog' },
      schema: BlogContentSchema
    }),
    
    // French content - same schema, different source
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/blog/**/*.md', prefix: '/blog' },
      schema: BlogContentSchema
    })
  }
})
```

### Using Nuxt Content's property() helper

```typescript
import { property } from '@nuxt/content'
import { z } from 'zod'

const schema = z.object({
  title: z.string(),
  // Add Studio editor hints
  image: property(z.string()).editor({ input: 'media' }),
  icon: property(z.string().optional()).editor({ input: 'icon' }),
  internalId: property(z.string()).editor({ hidden: true })
})
```

### SSG build-time validation

Content validation occurs at **build time** during `nuxt generate`. Invalid frontmatter surfaces as build errors:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true,
  nitro: {
    prerender: {
      crawlLinks: true
    }
  }
})
```

---

## Common pitfalls and anti-patterns

### ❌ Using deprecated `.merge()` method

```typescript
// WRONG: .merge() is deprecated in Zod 4
const Combined = SchemaA.merge(SchemaB)

// CORRECT: Use .extend() with .shape
const Combined = SchemaA.extend(SchemaB.shape)

// BEST: Use spread syntax
const Combined = z.object({
  ...SchemaA.shape,
  ...SchemaB.shape
})
```

### ❌ Chaining multiple `.extend()` calls

```typescript
// WRONG: Causes TypeScript performance issues
const Final = Base
  .extend({ a: z.string() })
  .extend({ b: z.string() })
  .extend({ c: z.string() })

// CORRECT: Single spread operation
const Final = z.object({
  ...Base.shape,
  a: z.string(),
  b: z.string(),
  c: z.string()
})
```

### ❌ Using `.extend()` on refined schemas

```typescript
// WRONG: Throws error in Zod 4
const Refined = z.object({ a: z.string() }).refine(/* ... */)
const Extended = Refined.extend({ b: z.number() })

// CORRECT: Use .safeExtend()
const Extended = Refined.safeExtend({ b: z.number() })
```

### ❌ Expecting refinements to survive pick/omit

```typescript
// WRONG: Refinement is silently dropped
const WithRefinement = z.object({
  min: z.number(),
  max: z.number()
}).refine(d => d.max > d.min, 'max must exceed min')

const Picked = WithRefinement.pick({ min: true })
// ⚠️ Refinement is LOST

// CORRECT: Reapply refinements after transformation
const Picked = WithRefinement
  .pick({ min: true, max: true })
  .refine(d => d.max > d.min, 'max must exceed min')
```

### ❌ Using z.intersection() for objects

```typescript
// WRONG: ZodIntersection lacks object methods
const Combined = z.intersection(SchemaA, SchemaB)
Combined.pick({ field: true })  // ❌ Method doesn't exist

// CORRECT: Use extend/spread for objects
const Combined = z.object({
  ...SchemaA.shape,
  ...SchemaB.shape
})
Combined.pick({ field: true })  // ✅ Works
```

### ❌ Ignoring input vs output types with transforms

```typescript
const DateField = z.string().transform(val => new Date(val))

// WRONG: Using z.infer when you need input type
type DateType = z.infer<typeof DateField>  // Date (output only)

// CORRECT: Track both types
type DateInput = z.input<typeof DateField>   // string
type DateOutput = z.output<typeof DateField> // Date
```

### ❌ Importing z from @nuxt/content

```typescript
// WRONG: Deprecated re-export
import { z } from '@nuxt/content'

// CORRECT: Direct import
import { z } from 'zod'
```

---

## Claude Code integration checklist

Use this checklist when implementing Zod 4 schemas in your Nuxt Content project:

### Schema setup

- [ ] Import `z` directly from `'zod'`, not from `'@nuxt/content'`
- [ ] Create shared schemas in `lib/content-schemas.ts`
- [ ] Use spread syntax `{...Schema.shape}` for composition
- [ ] Use `.safeExtend()` when extending schemas with refinements
- [ ] Export both schema and inferred type: `export type Article = z.infer<typeof ArticleSchema>`

### Content collections

- [ ] Define separate collections for each language (`blog_en`, `blog_fr`)
- [ ] Share schema definitions across language collections
- [ ] Use discriminated unions for content type differentiation
- [ ] Add `property()` wrapper for Nuxt Studio integration fields
- [ ] Define all queryable frontmatter fields in schema (unlisted fields go to `meta`)

### Discriminated unions

- [ ] Use `z.discriminatedUnion()` instead of `z.union()` for typed content
- [ ] Ensure discriminator field uses `z.literal()` or `z.enum()`
- [ ] Compose unions using `.options` property: `[...UnionA.options, ...UnionB.options]`
- [ ] Implement type-narrowing switch statements for content processing

### Type safety

- [ ] Use `z.infer<typeof Schema>` for all derived types
- [ ] Use `z.input<>` and `z.output<>` when schemas have transforms
- [ ] Avoid manual type definitions that duplicate schema structure
- [ ] Use `safeParse()` instead of `parse()` for graceful error handling

### Performance optimization

- [ ] Prefer spread syntax over chained `.extend()` calls
- [ ] Use discriminated unions over regular unions for tagged types
- [ ] Avoid deeply nested schema composition
- [ ] Validate at build time (SSG) rather than runtime where possible

### Migration from Zod 3

- [ ] Replace all `.merge()` calls with `.extend(schema.shape)` or spread
- [ ] Replace `zod-to-json-schema` with native `z.toJSONSchema()`
- [ ] Replace `.strict()` with `z.strictObject()`
- [ ] Replace `.passthrough()` with `z.looseObject()`
- [ ] Use codemod: `npx zod-v3-to-v4`

### Testing

- [ ] Validate frontmatter examples against schemas in tests
- [ ] Test discriminated union type narrowing
- [ ] Verify build fails on invalid content during CI
- [ ] Test schema composition outputs expected types

---

## Reference implementation for blog project

```typescript
// lib/content-schemas.ts
import { z } from 'zod'

// Timestamp mixin
const TimestampFields = {
  date: z.date(),
  updatedAt: z.date().optional()
}

// SEO mixin  
const SEOFields = {
  description: z.string().max(160).optional(),
  ogImage: z.string().optional()
}

// Base for all blog content
const BaseBlog = z.object({
  title: z.string().min(1).max(100),
  slug: z.string(),
  draft: z.boolean().default(false),
  author: z.string(),
  tags: z.array(z.string()).optional(),
  ...TimestampFields,
  ...SEOFields
})

// Content type schemas
export const ArticleSchema = z.object({
  ...BaseBlog.shape,
  type: z.literal('article'),
  content: z.string(),
  readingTime: z.number().positive()
})

export const TechArticleSchema = z.object({
  ...BaseBlog.shape,
  type: z.literal('tech-article'),
  content: z.string(),
  codeLanguage: z.string(),
  repository: z.string().url().optional(),
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
  type: z.literal('note'),
  body: z.string()
})

// Main discriminated union
export const BlogContentSchema = z.discriminatedUnion('type', [
  ArticleSchema,
  TechArticleSchema,
  TutorialSchema,
  NoteSchema
])

// Export inferred types
export type Article = z.infer<typeof ArticleSchema>
export type TechArticle = z.infer<typeof TechArticleSchema>
export type Tutorial = z.infer<typeof TutorialSchema>
export type Note = z.infer<typeof NoteSchema>
export type BlogContent = z.infer<typeof BlogContentSchema>

// JSON Schema export (Zod 4 native)
export const BlogContentJSONSchema = z.toJSONSchema(BlogContentSchema)
```

```typescript
// content.config.ts
import { defineContentConfig, defineCollection, property } from '@nuxt/content'
import { BlogContentSchema } from './lib/content-schemas'

export default defineContentConfig({
  collections: {
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'en/blog/**/*.md', prefix: '/blog' },
      schema: BlogContentSchema
    }),
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/blog/**/*.md', prefix: '/blog' },
      schema: BlogContentSchema
    })
  }
})
```

This implementation provides type-safe frontmatter validation, efficient schema composition using Zod 4 best practices, and clean separation between language collections while sharing schema definitions.