# Nuxt Content 3.10+ queryCollection API: Complete Guide for SSG Blogs

The queryCollection API in Nuxt Content 3.x represents a fundamental architectural shift from v2's document-based queries to **SQL-backed collection queries**, offering significant performance improvements for SSG deployments. For a Nuxt 4 blog on Cloudflare Pages, this means faster builds, type-safe queries, and zero-cost static hosting without requiring a D1 database—provided you use `nuxt generate` for full static output.

The key insight for your setup: with Nuxt 4.2.x and `srcDir: 'app/'`, your content directory stays at project root (not inside `app/`), and all queryCollection calls work identically at build-time and runtime via WASM SQLite in the browser for client-side navigation.

## The queryCollection API fundamentals

The `queryCollection()` function creates a chainable query builder scoped to a specific collection defined in `content.config.ts`. Unlike v2's `queryContent()`, it requires explicit collection names and uses SQL operators.

**Basic structure and method signatures:**

```typescript
// Base API - auto-imported in Vue and Nitro contexts
function queryCollection<T extends keyof Collections>(
  collection: T
): CollectionQueryBuilder<Collections[T]>

// Server-side requires event as first argument
queryCollection(event, 'docs').path(route.path).first()
```

### Core query methods

**`.all()` and `.first()`** execute queries and return results:

```typescript
// Fetch all matching documents
const { data: posts } = await useAsyncData('blog', () => 
  queryCollection('blog').all()
)

// Fetch single document (returns null if not found)
const { data: page } = await useAsyncData(route.path, () =>
  queryCollection('docs').path(route.path).first()
)
```

**`.path()`** queries by the generated route path—essential for catch-all pages:

```typescript
// pages/[...slug].vue
const slug = computed(() => withLeadingSlash(String(route.params.slug) || '/'))
const { data } = await useAsyncData(`page-${slug.value}`, () =>
  queryCollection('content').path(slug.value).first()
)
```

Path generation follows predictable patterns: `content/blog/hello.md` becomes `/blog/hello`, with numeric prefixes stripped (`1.guide/2.setup.md` → `/guide/setup`).

**`.where()`** uses SQL operators for filtering:

```typescript
// Available operators: '=', '>', '<', '<>', 'IN', 'BETWEEN', 'LIKE', 'IS NULL', 'IS NOT NULL'
queryCollection('docs')
  .where('category', '=', 'news')
  .where('date', '<', '2025-01-01')
  .all()

// Pattern matching with LIKE (% is wildcard)
queryCollection('content')
  .where('path', 'LIKE', '/blog%')
  .all()
```

For complex conditions, use **`.andWhere()`** and **`.orWhere()`**:

```typescript
queryCollection('docs')
  .where('published', '=', true)
  .andWhere(q => q.where('date', '>', '2024-01-01').where('category', '=', 'news'))
  .all()
// SQL: WHERE published = true AND (date > '2024-01-01' AND category = 'news')
```

**`.order()`** sorts results but has a critical caveat—it uses **alphabetical sorting**, not numerical:

```typescript
queryCollection('docs').order('date', 'DESC').all()

// ⚠️ Files named 1.md, 10.md, 2.md sort as: 1, 10, 2
// ✅ Use zero-padding: 01.md, 02.md, 10.md sorts correctly
```

## Pagination patterns for SSG blogs

Effective pagination in SSG requires combining **`.skip()`**, **`.limit()`**, and **`.count()`**. Unlike runtime pagination, SSG must pre-render each paginated page at build time.

**Standard pagination implementation:**

```typescript
// pages/blog/index.vue
const route = useRoute()
const currentPage = computed(() => parseInt(route.query.page as string) || 1)
const limit = 10

const { data: posts } = await useAsyncData(`blog-page-${currentPage.value}`, () =>
  queryCollection('blog')
    .where('path', 'LIKE', '/blog%')
    .order('publishedAt', 'DESC')
    .skip((currentPage.value - 1) * limit)
    .limit(limit)
    .all()
)

const { data: totalCount } = await useAsyncData('blog-count', () =>
  queryCollection('blog')
    .where('path', 'LIKE', '/blog%')
    .count()
)

const totalPages = computed(() => Math.ceil((totalCount.value ?? 0) / limit))
```

**Critical SSG consideration:** Each paginated route (`/blog?page=1`, `/blog?page=2`) must be pre-rendered. Configure explicit routes or use `crawlLinks`:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    prerender: {
      crawlLinks: true,
      routes: ['/blog', '/blog?page=1', '/blog?page=2'] // Explicit pagination routes
    }
  }
})
```

**useAsyncData key strategy:** Always use unique, consistent keys. Key collisions cause cache issues in SSG:

```typescript
// ❌ Bad - key collision
await useAsyncData('blog', () => queryCollection('blog').where('tag', '=', 'vue').all())
await useAsyncData('blog', () => queryCollection('blog').where('tag', '=', 'react').all())

// ✅ Good - unique keys
await useAsyncData(`blog-tag-${tag}`, () => queryCollection('blog').where('tag', '=', tag).all())
```

## Performance optimization with `.select()`

For listing pages that don't need full content, **`.select()` dramatically reduces payload size**:

```typescript
// List page - only fetch metadata fields
const { data: docs } = await useAsyncData('documents-list', () =>
  queryCollection('docs')
    .select('title', 'path', 'description', 'publishedAt')
    .order('publishedAt', 'DESC')
    .all()
)
```

**SSG performance reality check:** Nuxt Content v3 build performance scales roughly **O(n²)** for large content sets according to community benchmarks:

| Documents | Approximate Build Time |
|-----------|------------------------|
| 500 | ~1 minute |
| 2,000 | ~2.5 minutes |
| 5,000 | ~5.5 minutes |
| 10,000 | ~10+ minutes |

For blogs with under **1,000 documents**, this is manageable. For larger sites, consider splitting collections or using incremental builds.

**Enable shared prerender data** (Nuxt 3.10+) to reduce redundant queries:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  experimental: {
    sharedPrerenderData: true,
    payloadExtraction: true
  }
})
```

## Navigation helpers for prev/next articles

**`queryCollectionItemSurroundings()`** provides type-safe prev/next navigation:

```typescript
const route = useRoute()
const { data: surroundings } = await useAsyncData('surround', () =>
  queryCollectionItemSurroundings('blog', route.path, {
    before: 1,
    after: 1,
    fields: ['title', 'description', 'path']
  })
    .where('_draft', '=', false)
    .order('publishedAt', 'DESC')
)

// Returns [previousItem | null, nextItem | null]
```

**Template implementation:**

```vue
<template>
  <nav class="flex justify-between mt-8">
    <NuxtLink v-if="surroundings?.[0]" :to="surroundings[0].path" class="flex-1">
      ← {{ surroundings[0].title }}
    </NuxtLink>
    <span v-else class="flex-1" />
    <NuxtLink v-if="surroundings?.[1]" :to="surroundings[1].path" class="flex-1 text-right">
      {{ surroundings[1].title }} →
    </NuxtLink>
  </nav>
</template>
```

The surroundings query is chainable—you can filter and sort to ensure logical navigation order within a specific category or date range.

## SSG configuration for Cloudflare Pages

For **zero-cost static deployment** (no D1 database required), use `nuxt generate`:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  compatibilityDate: '2024-11-01',
  future: { compatibilityVersion: 4 },
  
  ssr: true,  // Required for SSG
  
  modules: ['@nuxt/content'],
  
  nitro: {
    prerender: {
      crawlLinks: true,
      routes: ['/', '/sitemap.xml'],
      failOnError: true
    }
  },
  
  routeRules: {
    '/blog/**': { prerender: true },
    '/fr/**': { prerender: true }
  }
})
```

**Cloudflare Pages deployment settings:**
- **Build command:** `npx nuxi generate`
- **Output directory:** `.output/public`
- **Environment:** `NODE_VERSION=20`

**Directory structure with Nuxt 4's `srcDir: 'app/'`:**

```
project/
├── app/                    # srcDir (Vue app)
│   ├── pages/
│   │   └── [...slug].vue
│   └── app.vue
├── content/                # At rootDir, NOT in app/
│   ├── fr/
│   │   └── blog/
│   └── en/
│       └── blog/
├── content.config.ts
└── nuxt.config.ts
```

**Multilingual collection setup:**

```typescript
// content.config.ts
import { defineCollection, defineContentConfig, z } from '@nuxt/content'

const blogSchema = z.object({
  title: z.string(),
  description: z.string().optional(),
  publishedAt: z.date(),
  tags: z.array(z.string()).optional()
})

export default defineContentConfig({
  collections: {
    content_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/**', prefix: '' },
      schema: blogSchema
    }),
    content_en: defineCollection({
      type: 'page',
      source: { include: 'en/**', prefix: '' },
      schema: blogSchema
    })
  }
})
```

## TypeScript integration patterns

Nuxt Content v3 auto-generates types from your collection schemas. No manual type definitions needed.

**Schema definition drives type inference:**

```typescript
// content.config.ts
import { z } from 'zod'  // Import directly from zod, not @nuxt/content

export default defineContentConfig({
  collections: {
    blog: defineCollection({
      type: 'page',
      source: 'blog/**/*.md',
      schema: z.object({
        author: z.string(),
        publishedAt: z.date(),
        tags: z.array(z.string()).optional(),
        featured: z.boolean().default(false)
      })
    })
  }
})
```

**Type-safe queries:**

```typescript
// posts is automatically typed based on your schema
const { data: posts } = await useAsyncData('blog', () =>
  queryCollection('blog')
    .where('featured', '=', true)  // Field names are type-checked
    .select('title', 'path', 'publishedAt')  // Select fields are validated
    .all()
)
```

**Server-side TypeScript setup** requires `server/tsconfig.json`:

```json
{
  "extends": "../.nuxt/tsconfig.server.json"
}
```

**Common type errors and solutions:**

| Error | Cause | Solution |
|-------|-------|----------|
| "Collection type not found" | Missing collection definition | Add to `content.config.ts` |
| "Property does not exist" | Field not in schema | Add field to collection schema |
| "Expected 2 arguments" (server) | Missing event parameter | Use `queryCollection(event, 'name')` |
| "no such column" | Querying undefined field | Define all queryable fields in schema |

## Breaking changes from Content 2.x to 3.x

The v3 migration involves significant API changes. Here's a comprehensive mapping:

**Query API migration:**

```typescript
// ❌ v2
const page = await queryContent(route.path).findOne()
const posts = await queryContent().where({ tags: { $contains: 'vue' } }).find()

// ✅ v3
const page = await queryCollection('content').path(route.path).first()
const posts = await queryCollection('blog').where('tags', 'LIKE', '%vue%').all()
```

**Key API changes:**

| v2 | v3 |
|----|-----|
| `queryContent()` | `queryCollection('name')` |
| `.findOne()` | `.first()` |
| `.find()` | `.all()` |
| `fetchContentNavigation()` | `queryCollectionNavigation()` |
| `.findSurround()` | `queryCollectionItemSurroundings()` |
| `doc._path` | `doc.path` |
| Regex in where() | SQL LIKE operator |

**Array field querying (tags):**

```typescript
// ❌ v2 - $contains operator
.where({ tags: { $contains: tagSlug } })

// ✅ v3 - LIKE with quotes to prevent partial matches
.where('tags', 'LIKE', `%"${tag}"%`)

// Multiple tags with AND logic
const query = queryCollection('blog')
query.andWhere(q => {
  tags.forEach(tag => q.where('tags', 'LIKE', `%${tag}%`))
  return q
})
```

**Document-driven mode removed:** You must create explicit catch-all pages:

```vue
<!-- pages/[...slug].vue -->
<script setup lang="ts">
const route = useRoute()
const { data: page } = await useAsyncData(route.path, () =>
  queryCollection('content').path(route.path).first()
)
</script>

<template>
  <ContentRenderer v-if="page" :value="page" />
  <div v-else>Page not found</div>
</template>
```

## Anti-patterns to avoid

**Never query inside `onMounted()`**—breaks SSG:

```typescript
// ❌ Only runs client-side, no prerendering
onMounted(async () => {
  const data = await queryCollection('blog').all()
})

// ✅ Use useAsyncData at top level
const { data } = await useAsyncData('blog', () => queryCollection('blog').all())
```

**Never query nested object fields directly:**

```typescript
// ❌ SQL doesn't support dot notation
.where('meta.published', '=', true)

// ✅ Flatten schema—define fields at top level
schema: z.object({
  published: z.boolean()
})
```

**Never use non-zero-padded numeric prefixes:**

```
// ❌ Sorts as: 1, 10, 11, 2, 3...
content/1.intro.md
content/10.advanced.md

// ✅ Correct alphabetical order
content/01.intro.md
content/10.advanced.md
```

**Never forget unique `useAsyncData` keys** for different queries to the same collection—this causes silent cache bugs in SSG.

## Conclusion

Nuxt Content 3.x's SQL-backed queryCollection API provides a robust foundation for SSG blogs, with notable advantages in type safety and query performance. The architecture shift requires adapting from v2 patterns but delivers predictable, efficient builds.

For your Nuxt 4 + Cloudflare Pages setup, the critical path is: define collections with explicit schemas in `content.config.ts`, use `select()` to minimize payloads on listing pages, ensure unique `useAsyncData` keys for all queries, and deploy with `nuxt generate` for true zero-cost static hosting. The WASM SQLite runtime handles client-side navigation queries automatically, eliminating the need for D1 or any server infrastructure.

Key gaps in the current ecosystem include the lack of a built-in `CONTAINS` operator for array fields (use LIKE workarounds), limited official i18n integration (manage with separate locale collections), and the O(n²) build scaling for very large content sets. For blogs under 1,000 documents, these limitations are manageable with the patterns documented above.