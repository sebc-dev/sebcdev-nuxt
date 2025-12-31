# Zod 4 frontmatter validation in Nuxt Content 3

**Zod 4, released stable July 10, 2025, brings significant breaking changes that directly impact frontmatter validation patterns.** The most critical changes include a unified `error` parameter replacing fragmented error APIs, `.default()` now applying within `.optional()` fields (reversing Zod 3 behavior), and native JSON Schema export via `z.toJSONSchema()`. For Nuxt Content 3.10+ with SSG on Cloudflare Pages, the combination of **zod/mini** for bundle optimization and SQLite-backed build-time validation creates an efficient content pipeline—but understanding the API changes is essential for correct schema definition.

## Date coercion: ISO strings to JavaScript Date objects

Date handling in YAML frontmatter requires careful consideration of validation versus transformation. The `z.coerce.date()` method converts strings to Date objects using `new Date(input)`, but produces confusing errors when coercion fails—parsing `'www'` yields `"Invalid input: expected date, received Date"` because the invalid string becomes an Invalid Date object.

The recommended pattern validates format before coercion:

```typescript
// Safe date validation for frontmatter
const safeDate = z.iso.datetime({ offset: true })
  .pipe(z.coerce.date())

// Or with explicit validation
const validatedDate = z.string()
  .refine((val) => !isNaN(Date.parse(val)), {
    message: "Invalid date string"
  })
  .transform((val) => new Date(val))
```

Zod 4 introduces dedicated **ISO validators** (`z.iso.date()`, `z.iso.datetime()`, `z.iso.time()`) that return validated strings before transformation. For timezone handling, `z.iso.datetime({ offset: true })` accepts timezone offsets like `+02:00`, while the default rejects them. YAML frontmatter dates serialize as ISO strings in Nuxt Content's SQLite storage, so dates stored as `format: date-time` in the generated JSON Schema remain queryable across build and runtime.

Date constraints apply after coercion:

```typescript
z.coerce.date()
  .min(new Date("2020-01-01"), { message: "Too old" })
  .max(new Date(), { message: "Cannot be future" })
```

## Enum validation with type inference and custom errors

The `z.enum()` pattern provides the strongest type inference for blog categories. Zod 4 now accepts TypeScript enums directly (replacing the deprecated `z.nativeEnum()`), and the `.enum` accessor replaces removed `.Enum` and `.Values` properties:

```typescript
const CategorySchema = z.enum(['tutorial', 'news', 'opinion', 'review'], {
  error: (issue) => `Invalid category. Expected: tutorial, news, opinion, review`
})

type Category = z.infer<typeof CategorySchema>
// => "tutorial" | "news" | "opinion" | "review"

// Access values programmatically
CategorySchema.enum // => { tutorial: "tutorial", news: "news", ... }
```

For extensibility, Zod 4's `.extract()` and `.exclude()` methods create subset enums without redefinition:

```typescript
const AllCategories = z.enum(['tutorial', 'news', 'opinion', 'review'])
const BlogOnly = AllCategories.extract(['tutorial', 'opinion']) // Subset
const NonNews = AllCategories.exclude(['news'])                 // Exclusion
```

When categories must be dynamic or mixed with other types, `z.union()` provides more flexibility than `z.enum()`, though with weaker type inference. The `as const` assertion on external arrays preserves literal types for enum construction.

## Optional fields and defaults: critical Zod 4 changes

**The most impactful breaking change** affects optional/default interaction. In Zod 4, defaults now apply within optional fields:

```typescript
const schema = z.object({
  draft: z.string().default("untitled").optional()
})

schema.parse({})
// Zod 3: {}
// Zod 4: { draft: "untitled" } ⚠️ Breaking change
```

The pattern hierarchy for frontmatter schemas:

| Pattern | Input Type | Output Type | Behavior |
|---------|------------|-------------|----------|
| `.default('x')` | `T \| undefined` | `T` | Provides fallback |
| `.optional()` | `T \| undefined` | `T \| undefined` | No fallback |
| `.nullish()` | `T \| null \| undefined` | `T \| null \| undefined` | YAML null support |
| `.nullish().default('x')` | `T \| null \| undefined` | `T` | Handles YAML nulls |

For boolean flags in frontmatter, `.default()` triggers only on `undefined`, not `null`:

```typescript
const FrontmatterSchema = z.object({
  draft: z.boolean().default(false),     // Missing → false
  featured: z.boolean().default(false),  // Missing → false
  description: z.string().optional(),    // Missing → undefined (preserved)
  author: z.string().default('Anonymous') // Missing → 'Anonymous'
})
```

Zod 4 introduces `.prefault()` for pre-parse defaults (matching Zod 3's `.default()` behavior with transforms):

```typescript
const titleSchema = z.string()
  .transform(val => val.toUpperCase())
  .prefault('untitled') // Input type default

titleSchema.parse(undefined) // => "UNTITLED"
```

## Array constraints with unique validation and normalization

Zod lacks built-in unique validation, requiring `superRefine` for duplicate detection:

```typescript
const TagsSchema = z.array(z.string())
  .min(1, { message: 'At least one tag required' })
  .max(10, { message: 'Maximum 10 tags' })
  .superRefine((tags, ctx) => {
    const unique = new Set(tags)
    if (tags.length !== unique.size) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: 'Tags must be unique'
      })
    }
  })
```

For normalization pipelines that lowercase, trim, and deduplicate:

```typescript
const NormalizedTags = z.array(z.string())
  .transform(tags => 
    [...new Set(tags.map(t => t.trim().toLowerCase()))]
  )
  .pipe(z.array(z.string()).min(1).max(10))
```

Zod 4's `.nonempty()` type inference changed: it now returns `string[]` (same as `.min(1)`) rather than `[string, ...string[]]`. For tuple-style inference, use `z.tuple([z.string()], z.string())`.

## Zod 4 breaking changes and bundle optimization

### Critical migration items

The unified `error` parameter replaces four separate APIs:

```typescript
// Zod 3 (deprecated)
z.string({ message: "...", invalid_type_error: "...", required_error: "..." })

// Zod 4 (new)
z.string({ error: "..." })
z.string({ error: (issue) => issue.input === undefined ? "Required" : "Invalid" })
```

Additional breaking changes:
- **`.merge()` deprecated**: Use `.extend()` or object spread instead
- **`z.record()` requires two arguments**: `z.record(z.string(), z.string())` not `z.record(z.string())`
- **`.format()` and `.flatten()` removed**: Use `z.treeifyError()` for error formatting
- **String format methods deprecated**: `z.string().email()` → `z.email()`

### Bundle optimization with zod/mini

For SSG on Cloudflare Pages where **bundle size matters**, zod/mini provides ~64% reduction:

| Scenario | Regular Zod | Zod Mini |
|----------|-------------|----------|
| Simple boolean parse | 5.91 KB | 2.12 KB |
| Object with 3 fields | 13.1 KB | 4.0 KB |
| Full library gzipped | ~17 KB | ~1.9 KB |

Zod/mini uses functional composition instead of method chaining:

```typescript
import * as z from 'zod/mini'

// Method chaining (regular)
z.string().min(5).max(100).trim()

// Functional (mini, tree-shakable)
z.string().check(z.minLength(5), z.maxLength(100), z.trim())
```

The tradeoff: zod/mini requires `z.config(z.locales.en())` for localized error messages (defaults to "Invalid input").

### Native JSON Schema export

Zod 4 includes `z.toJSONSchema()` natively, deprecating the `zod-to-json-schema` package:

```typescript
z.toJSONSchema(schema, {
  target: "draft-2020-12", // or "draft-07", "openapi-3.0"
  io: "output",           // Handle transforms
  unrepresentable: "any"  // How to handle z.date(), z.transform(), etc.
})
```

Nuxt Content internally converts Zod schemas to JSON Schema Draft-07 for SQLite storage, making this export capability useful for external tooling.

## Nuxt Content 3.10+ integration patterns

### defineCollection() with Zod schemas

The deprecation warning in Nuxt Content documentation is important: import `z` directly from `zod`, not from `@nuxt/content`:

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { z } from 'zod' // Direct import

export default defineContentConfig({
  collections: {
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'en/blog/**/*.md', prefix: '' },
      schema: z.object({
        title: z.string(),
        date: z.coerce.date(),
        category: z.enum(['tutorial', 'news', 'opinion']),
        tags: z.array(z.string()).default([]),
        draft: z.boolean().default(false)
      })
    }),
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/blog/**/*.md', prefix: '' },
      schema: z.object({
        // Same schema for consistency
      })
    })
  }
})
```

### Build-time validation architecture

Nuxt Content 3's SQLite-backed system processes content at **build time only**:

1. Content files parsed into AST
2. Zod schemas converted to JSON Schema Draft-07
3. Validation applied before SQLite insertion
4. Database dump generated for deployment

**Critical limitation**: No runtime validation occurs. Invalid frontmatter values may be silently omitted or set to `undefined` without explicit build errors. For SSG mode, validation happens during `npx nuxi generate`, and the SQLite dump ships with static output.

### Error handling realities

Build-time validation lacks detailed error reporting. Schema violations appear indirectly through missing fields rather than explicit errors. For development feedback, consider external linting:

```typescript
// Custom error messages still help debugging
schema: z.object({
  category: z.enum(['tutorial', 'news'], {
    error: 'Category must be "tutorial" or "news"'
  }),
  date: z.coerce.date().refine(
    d => !isNaN(d.getTime()),
    { message: 'Invalid date format' }
  )
})
```

## TypeScript integration and type inference

### Extracting types from schemas

The `z.infer<>` utility extracts output types; for transforms, use `z.input<>` for pre-transform types:

```typescript
const PostSchema = z.object({
  title: z.string(),
  date: z.coerce.date(),          // Input: string, Output: Date
  tags: z.array(z.string()).default([])
})

type PostInput = z.input<typeof PostSchema>
// { title: string; date: string; tags?: string[] | undefined }

type PostOutput = z.infer<typeof PostSchema> // Same as z.output<>
// { title: string; date: Date; tags: string[] }
```

### Type-safe content queries

Nuxt Content generates types from collection definitions:

```typescript
// Fully typed query
const { data: post } = await useAsyncData('post', () =>
  queryCollection('blog_en')
    .where('category', '=', 'tutorial')
    .order('date', 'DESC')
    .first()
)
// post.value typed with schema fields + page fields (path, body, seo, etc.)
```

For language-separated collections, use dynamic collection selection:

```typescript
const collection = `blog_${locale.value}` as keyof Collections
const content = await queryCollection(collection).path(slug).first()
```

## Complete frontmatter schema template

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { z } from 'zod'

const categories = ['tutorial', 'news', 'opinion'] as const

const postSchema = z.object({
  // Required
  title: z.string().min(1).max(200),
  category: z.enum(categories, {
    error: `Must be: ${categories.join(', ')}`
  }),
  
  // Dates with ISO validation
  publishedAt: z.iso.datetime({ offset: true }).pipe(z.coerce.date()),
  updatedAt: z.coerce.date().optional(),
  
  // Booleans with defaults
  draft: z.boolean().default(false),
  featured: z.boolean().default(false),
  
  // Optional strings
  description: z.string().max(300).optional(),
  author: z.string().default('Anonymous'),
  
  // Normalized, unique tags
  tags: z.array(z.string())
    .transform(tags => [...new Set(tags.map(t => t.trim().toLowerCase()))])
    .pipe(z.array(z.string()).min(1).max(10))
    .default(['general']),
  
  // Nullable (explicit null in YAML)
  coverImage: z.string().url().nullable().default(null)
})

export default defineContentConfig({
  collections: {
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'en/blog/**/*.md', prefix: '' },
      schema: postSchema
    }),
    blog_fr: defineCollection({
      type: 'page', 
      source: { include: 'fr/blog/**/*.md', prefix: '' },
      schema: postSchema
    })
  }
})
```

## Conclusion

The Zod 4 migration for Nuxt Content requires attention to three behavioral changes: **defaults applying within optional fields**, **the unified error parameter**, and **record schemas requiring two arguments**. For Cloudflare Pages SSG deployments, zod/mini reduces validation overhead by ~64% while maintaining full type inference through `z.infer<>`. Build-time-only validation means frontmatter errors surface as missing data rather than explicit failures—making schema design with sensible defaults critical for production reliability.