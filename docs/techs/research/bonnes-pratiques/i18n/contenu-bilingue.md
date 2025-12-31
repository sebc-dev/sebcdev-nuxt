# Multilingual content management in Nuxt 4 with i18n and Content 3

The recommended architecture for bilingual Nuxt sites uses **separate content collections per locale** with dynamically selected queries, the `watch: [locale]` reactivity pattern, and explicit fallback handling. Nuxt Content 3's SQL-backed storage and @nuxtjs/i18n v10's lazy-loading changes require specific configurations that differ significantly from previous versions. This combination works well for SSG on Cloudflare Pages, though some integration friction points remain—particularly around translated slugs and hreflang generation.

## Content directory structure follows the mirror pattern

The officially recommended approach organizes content in **language-separated directories** that mirror each other:

```
content/
  en/
    index.md
    about.md
    blog/
      getting-started.md
  fr/
    index.md
    about.md
    blog/
      getting-started.md
```

Each locale directory contains the same structural hierarchy, enabling predictable path-based queries. The `prefix` option in collection configuration controls whether the language folder appears in generated URLs—set `prefix: ''` or `prefix: '/'` to strip the `/en/` or `/fr/` prefix from paths, allowing the i18n module to handle routing prefixes instead.

**Key constraint**: A document can only belong to ONE collection at a time. This limitation (tracked in nuxt/content#2966) necessitates the separate-collections-per-locale pattern rather than a unified collection with locale filtering.

## Collections configuration with shared Zod schemas

Define separate collections for each locale in `content.config.ts`, sharing a common schema to ensure type consistency:

```typescript
// content.config.ts
import { defineCollection, defineContentConfig, z } from '@nuxt/content'

const articleSchema = z.object({
  title: z.string(),
  description: z.string(),
  date: z.date(),
  tags: z.array(z.string()).optional(),
  author: z.object({
    name: z.string(),
    avatar: z.string().optional()
  }).optional(),
  published: z.boolean().default(true)
})

export default defineContentConfig({
  collections: {
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'en/blog/**', prefix: '/' },
      schema: articleSchema,
      indexes: [
        { columns: ['date'] },
        { columns: ['published'] }
      ]
    }),
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/blog/**', prefix: '/' },
      schema: articleSchema,
      indexes: [
        { columns: ['date'] },
        { columns: ['published'] }
      ]
    })
  }
})
```

**SQLite indexing** significantly improves query performance—particularly important for Cloudflare D1 deployments where indexed `WHERE` clauses count as **1 row read** versus scanning all rows. Define indexes on frequently filtered columns like `date`, `published`, or `category`.

## Dynamic locale-based querying with fallback

The `queryCollection()` API requires explicit collection selection. Construct the collection name dynamically using the current locale:

```typescript
// pages/blog/[...slug].vue
<script setup lang="ts">
import { withLeadingSlash } from 'ufo'
import type { Collections } from '@nuxt/content'

const route = useRoute()
const { locale } = useI18n()
const slug = computed(() => withLeadingSlash(String(route.params.slug)))

const { data: article } = await useAsyncData(
  `blog-${slug.value}`,
  async () => {
    const collection = `blog_${locale.value}` as keyof Collections
    let content = await queryCollection(collection)
      .path(slug.value)
      .where('published', '=', true)
      .first()

    // Fallback to default locale if translation missing
    if (!content && locale.value !== 'en') {
      content = await queryCollection('blog_en')
        .path(slug.value)
        .first()
    }
    return content
  },
  { watch: [locale] }  // Critical: refetch on locale change
)
</script>

<template>
  <article v-if="article">
    <ContentRenderer :value="article" />
  </article>
  <div v-else>
    <p>Content not available in {{ locale }}</p>
  </div>
</template>
```

The **`watch: [locale]`** option is essential—without it, content won't refetch when users switch languages. Use a locale-independent key like `blog-${slug.value}` rather than including locale in the key to prevent cache fragmentation.

## Routing integration with prefix_except_default strategy

Configure @nuxtjs/i18n v10 to complement the content structure:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/content', '@nuxtjs/i18n'],
  
  i18n: {
    locales: [
      { code: 'en', language: 'en-US', name: 'English', file: 'en.json' },
      { code: 'fr', language: 'fr-FR', name: 'Français', file: 'fr.json' }
    ],
    defaultLocale: 'en',
    strategy: 'prefix_except_default',
    lazy: true,  // Note: now default in v10
    langDir: 'locales/',
    detectBrowserLanguage: {
      useCookie: true,
      cookieKey: 'i18n_redirected',
      redirectOn: 'root'  // Behavior changed in v10
    }
  }
})
```

With `prefix_except_default`, English routes have no prefix (`/blog/my-post`) while French routes use `/fr/blog/my-post`. This strategy generates route names following the pattern `{page-name}___{locale-code}`.

**Breaking change from v9**: The `redirectOn: 'root'` option now ONLY redirects the root path. Use `redirectOn: 'all'` if you need browser detection on all paths.

## Sitemap generation with automatic hreflang

The @nuxtjs/sitemap module integrates automatically with @nuxtjs/i18n to generate per-locale sitemaps with proper hreflang annotations:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/i18n', '@nuxtjs/sitemap'],
  
  site: {
    url: 'https://example.com'
  },
  
  sitemap: {
    // Generates: /sitemap_index.xml, /en-sitemap.xml, /fr-sitemap.xml
  }
})
```

For dynamic content routes, create a sitemap source:

```typescript
// server/api/__sitemap__/urls.ts
export default defineSitemapEventHandler(async () => {
  const posts = await $fetch('/api/posts')
  
  return posts.map(post => ({
    loc: `/blog/${post.slug}`,
    lastmod: post.updatedAt,
    _i18nTransform: true  // Auto-generates locale variants with hreflang
  }))
})
```

The **`_i18nTransform: true`** flag automatically generates all locale variants with proper `xhtml:link` hreflang tags in the XML output.

## SSG deployment on Cloudflare Pages

Configure Nitro for static generation with the Cloudflare Pages preset:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true,
  
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      crawlLinks: true,
      autoSubfolderIndex: false,
      routes: ['/', '/fr']  // Explicitly seed locale roots
    }
  }
})
```

**Critical Cloudflare dashboard settings** to disable (prevents hydration errors):
- Speed → Optimization → **Rocket Loader™**: OFF
- Speed → Optimization → Image Optimization → **Mirage**: OFF  
- Scrape Shield → **Email Address Obfuscation**: OFF

Build command for SSG: `nuxt generate` with output directory `.output/public`.

## Schema.org markup for multilingual articles

Use `nuxt-schema-org` (or the full `@nuxtjs/seo` suite) for structured data:

```typescript
// pages/blog/[slug].vue
<script setup>
const { locale } = useI18n()

useSchemaOrg([
  defineArticle({
    headline: article.value.title,
    description: article.value.description,
    inLanguage: locale.value,
    datePublished: article.value.date,
    author: {
      '@type': 'Person',
      name: article.value.author?.name
    }
  })
])
</script>
```

**Module order matters**: When using `@nuxtjs/seo` with `@nuxt/content`, the SEO module must be loaded FIRST in the modules array to properly inject frontmatter handling.

## Anti-patterns and known limitations

Several patterns cause problems in this stack:

- **Hardcoding locale in useAsyncData keys** fragments the cache unnecessarily—use path-only keys
- **Expecting automatic slug translation** doesn't work natively; implement via frontmatter UID fields or manual mapping
- **Querying nested frontmatter values** like `meta.published` isn't supported by queryCollection's SQL backend
- **Using v2's `_locale` field** doesn't exist in Content 3's schema; filter by collection name or path patterns instead
- **Omitting `watch: [locale]`** leaves content stale after language switches

**Current limitation** (GitHub issue #3579): No native connection between locale versions of the same content exists. Implementing language switchers that link to the same article in another language requires custom frontmatter fields like a shared `uid` or explicit `alternateLocales` paths.

## Migration notes from previous versions

| Content 2.x → 3.x | i18n v9 → v10 |
|-------------------|---------------|
| `queryContent()` → `queryCollection('name')` | `lazy` option removed (always lazy) |
| `.findOne()` → `.first()` | `redirectOn: 'root'` behavior fixed |
| `.where({ path: /regex/ })` → `.where('path', 'LIKE', '/pattern%')` | `onLanguageSwitched()` → `i18n:localeSwitched` hook |
| `_path` → `path` (no underscore prefix) | Vue I18n upgraded to v11 |
| Document-driven mode removed | Nitro-level language detection added |

The combination of Nuxt Content 3.10+ with @nuxtjs/i18n 10.2+ provides a solid foundation for multilingual SSG sites, though the separate-collections pattern adds configuration overhead for sites with many locales. The official example at https://content3-i18n.nuxt.dev/ demonstrates this architecture in practice.