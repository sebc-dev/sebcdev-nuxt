# Content testing strategies for Nuxt Content 3.x in production

Schema validation in Nuxt Content 3 **does not fail builds by default**—invalid frontmatter fields are silently dropped or set to undefined. This critical behavior means production-quality content pipelines require explicit validation scripts, layered link checking, and adapted markdown linting tools since no MDC-specific linters exist. For SSG deployments on Cloudflare Pages with language-separated collections, the recommended approach combines pre-build Zod validation scripts, **nuxt-link-checker** with lychee for comprehensive link auditing, and markdownlint-cli2 with the `markdown-it-mdc` parser plugin.

## Frontmatter validation requires explicit enforcement

Nuxt Content 3's collection system uses `defineCollection()` in `content.config.ts` with native Zod v3 or v4 support. Schemas define the source of truth for TypeScript types, but validation occurs **at content parsing time** without throwing errors.

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { z } from 'zod' // Import directly, not from @nuxt/content (deprecated)

const blogSchema = z.object({
  title: z.string().min(1),
  date: z.date(),
  draft: z.boolean().default(false),
  description: z.string().optional(),
  tags: z.array(z.string()).optional()
})

export default defineContentConfig({
  collections: {
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'en/blog/**/*.md', prefix: '' },
      schema: blogSchema
    }),
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/blog/**/*.md', prefix: '' },
      schema: blogSchema
    })
  }
})
```

**Zod v4** (released mid-2025) works natively without `zod-to-json-schema`, offering **14x faster string parsing** and 57% smaller bundles. Import via `zod/v4` for explicit version targeting. The schema converts to JSON Schema Draft-07 internally, but missing required fields simply omit the field rather than triggering build failures.

To enforce validation with build failures, implement a `content:file:afterParse` hook or create a pre-build script:

```typescript
// scripts/validate-content.ts
import { z } from 'zod'
import matter from 'gray-matter'
import { glob } from 'glob'
import { readFileSync } from 'fs'
import { blogSchema } from '../lib/content-schemas'

async function validateCollection(pattern: string, schema: z.ZodSchema) {
  const files = await glob(pattern)
  const errors: Array<{ file: string; issues: z.ZodIssue[] }> = []

  for (const file of files) {
    const { data: frontmatter } = matter(readFileSync(file, 'utf-8'))
    const result = schema.safeParse(frontmatter)
    if (!result.success) {
      errors.push({ file, issues: result.error.issues })
    }
  }
  return errors
}

// Run validation with exit(1) on failure for CI
```

MDC component testing presents unique challenges since `queryCollection()` requires the full Nuxt environment. Use `@nuxt/test-utils` with `mockNuxtImport` to mock the query builder chain, or run integration tests with `mountSuspended` for components that depend on real content queries.

## Link checking demands a layered verification approach

**nuxt-link-checker** (v4.3.9, December 2025) serves as the primary Nuxt-native solution, detecting 12+ link issue types with DevTools integration and build-time scanning. Configure it in `nuxt.config.ts` with `failOnError: true` to halt SSG builds on broken links:

```typescript
export default defineNuxtConfig({
  modules: ['nuxt-link-checker'],
  linkChecker: {
    failOnError: true,
    runOnBuild: true,
    report: ['html', 'markdown']
  },
  nitro: {
    prerender: {
      crawlLinks: true,
      routes: ['/']
    }
  }
})
```

For SSG deployments, links are **only checked on prerendered pages**—ensure `nitro.prerender.crawlLinks: true` discovers all routes. The module hooks into Nitro's prerender process automatically during `nuxt generate`.

**Lychee** (Rust-based) provides the fastest comprehensive scanning for CI pipelines, checking built static output with caching support:

```yaml
- uses: lycheeverse/lychee-action@v2
  with:
    args: >-
      --verbose --cache --max-cache-age 1d
      --exclude linkedin\.com --exclude twitter\.com
      './output/**/*.html'
    fail: true
```

For pre-build internal link validation in markdown source files, **remark-validate-links** works offline without network calls, validating cross-file heading references before content reaches the build pipeline.

| Tool | Internal Links | External Links | Speed | Best For |
|------|----------------|----------------|-------|----------|
| nuxt-link-checker | ✅ | ✅ | Fast | Development + build |
| lychee | ✅ | ✅ | Very fast | CI/CD post-build |
| remark-validate-links | ✅ | ❌ | Fast | Pre-build markdown |
| linkinator | ✅ | ✅ | Medium | Node.js scripting |

## Schema compliance testing integrates with CI through custom scripts

The recommended CI pipeline separates content validation from the build step, failing fast before resource-intensive static generation:

```yaml
# .github/workflows/deploy.yml
jobs:
  validate-content:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: '10.26' }
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'pnpm' }
      - run: pnpm install
      - run: pnpm validate:content  # Exit 1 on schema errors

  build:
    needs: validate-content
    runs-on: ubuntu-latest
    steps:
      - run: pnpm generate
      - uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          projectName: your-project
          directory: .output/public
```

For **Node.js 22 LTS**, enable native SQLite to avoid `better-sqlite3` compilation issues in CI:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  content: {
    experimental: {
      nativeSqlite: true  // Uses Node's built-in sqlite module
    }
  }
})
```

Multi-collection validation for language-separated content shares a base schema with locale-specific extensions:

```typescript
const baseSchema = z.object({
  title: z.string(),
  date: z.date(),
  draft: z.boolean().default(false)
})

export const blogEnSchema = baseSchema.extend({ locale: z.literal('en') })
export const blogFrSchema = baseSchema.extend({ locale: z.literal('fr') })
```

**nuxt-content-assets** allows colocated images with markdown files. Validate asset references exist by scanning frontmatter image fields and inline markdown image syntax against the filesystem before build.

## Testing tools require MDC-aware configuration

**@nuxt/test-utils v3.21.0** provides first-class Vitest support with the `nuxt` test environment. Configure separate test projects for unit tests (fast, Node environment) and Nuxt integration tests:

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import { defineVitestProject } from '@nuxt/test-utils/config'

export default defineConfig({
  test: {
    projects: [
      {
        test: { name: 'unit', include: ['test/unit/*.spec.ts'], environment: 'node' }
      },
      await defineVitestProject({
        test: { name: 'nuxt', include: ['test/nuxt/*.spec.ts'], environment: 'nuxt' }
      })
    ]
  }
})
```

Testing `queryCollection()` requires mocking the chainable API:

```typescript
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'
import { vi } from 'vitest'

mockNuxtImport('queryCollection', () => {
  return () => ({
    path: vi.fn().mockReturnThis(),
    where: vi.fn().mockReturnThis(),
    order: vi.fn().mockReturnThis(),
    all: vi.fn().mockResolvedValue(mockPosts),
    first: vi.fn().mockResolvedValue(mockPosts[0])
  })
})
```

**No MDC-specific linting tools exist**—the ecosystem adapts standard markdown linters. **markdownlint-cli2** with `markdown-it-mdc` parser plugin handles MDC syntax correctly:

```jsonc
// .markdownlint-cli2.jsonc
{
  "markdownItPlugins": [["markdown-it-mdc"]],
  "frontMatter": "^---\\s*$[\\s\\S]*?^---\\s*$",
  "config": {
    "MD033": false,  // Allow inline HTML for Vue components
    "MD041": false,  // First-line heading (frontmatter interferes)
    "MD013": { "line_length": 120, "code_blocks": false }
  },
  "globs": ["content/**/*.{md,mdc}"]
}
```

For frontmatter YAML validation with JSON Schema in editors, **remark-lint-frontmatter-schema** maps schema files to content patterns, enabling in-file schema references via `$schema` in frontmatter.

## Recommended toolchain for zero-cost SSG deployment

The complete validation pipeline for Cloudflare Pages SSG:

```json
{
  "scripts": {
    "validate:content": "tsx scripts/validate-content.ts",
    "validate:assets": "tsx scripts/validate-assets.ts",
    "lint:md": "markdownlint-cli2 'content/**/*.{md,mdc}'",
    "generate": "pnpm validate:content && nuxt generate",
    "test": "vitest"
  }
}
```

- **Build-time schema validation**: Custom Zod validation script with `gray-matter` parsing, exit 1 on errors
- **Link checking**: nuxt-link-checker during build + lychee in CI for comprehensive coverage
- **Markdown linting**: markdownlint-cli2 with markdown-it-mdc plugin, disable MD033/MD041 for MDC compatibility
- **Testing**: @nuxt/test-utils with Vitest, mock queryCollection for unit tests, full Nuxt environment for integration tests
- **CI structure**: Validate → Lint → Test → Build → Deploy, failing fast at each stage

## Conclusion

Production content validation in Nuxt Content 3.x requires proactive tooling since the framework prioritizes graceful degradation over strict enforcement. The critical insight is that **schema validation silently drops invalid fields** rather than failing builds—teams must implement explicit validation scripts that parse frontmatter with `gray-matter` and run Zod schemas with `safeParse()` before the Nuxt build step. For link checking, the combination of nuxt-link-checker at build time with lychee for post-build verification catches both internal routing issues and external dead links without impacting zero-cost deployment constraints. The MDC format's lack of dedicated linting tools means adapting markdownlint with the `markdown-it-mdc` parser and disabling rules that conflict with component syntax. This layered approach—schema validation, link checking, content linting, and mocked queryCollection testing—provides comprehensive coverage while maintaining the fast feedback loops essential for content-heavy SSG projects.