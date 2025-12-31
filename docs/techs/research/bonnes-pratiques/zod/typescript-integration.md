# TypeScript + Zod 4 integration in Vue 3 and Nuxt 4

Zod 4's stable release (July 2025) brings transformative improvements for TypeScript developers: **14x faster string parsing**, **57% smaller bundles**, and **100x fewer TypeScript type instantiations**. For Vue 3/Nuxt 4 SSG workflows, schema validation now happens entirely at build time, making runtime performance nearly irrelevant while delivering full type safety from content to component props.

The integration story has three layers: Zod 4's refined type inference APIs (`z.infer`, `z.input`, `z.output`), Vue 3's compiler limitations with complex generic types in `defineProps`, and Nuxt Content's `defineCollection` API that converts Zod schemas to JSON Schema for build-time frontmatter validation. For Cloudflare Pages SSG deployment, all validation occurs during `nuxi generate`—no server functions required.

## Zod 4 type inference fundamentals

The core pattern remains `z.infer<typeof Schema>`, which extracts the **output type** from any Zod schema. Zod 4 introduces clearer semantics for transforms and preprocessing through the `z.input<>` and `z.output<>` utilities.

```typescript
const UserSchema = z.object({
  age: z.string().transform(val => parseInt(val)),
  createdAt: z.date().default(() => new Date())
});

type UserInput = z.input<typeof UserSchema>;  // { age: string; createdAt?: Date }
type UserOutput = z.output<typeof UserSchema>; // { age: number; createdAt: Date }
```

**Breaking change from Zod 3**: The `.default()` method now expects the **output type** and short-circuits parsing entirely. Use `.prefault()` for pre-parse defaults that run through transforms:

```typescript
// Zod 4: default short-circuits, expects OUTPUT type
z.string().transform(val => val.length).default(0);   // parse(undefined) → 0

// Use .prefault() for old Zod 3 behavior (input type)
z.string().transform(val => val.length).prefault("tuna"); // parse(undefined) → 4
```

For typed error handling, `z.inferFormattedError<typeof Schema>` extracts the error structure. Zod 4 deprecates the instance methods `.format()` and `.flatten()` in favor of top-level functions:

```typescript
const result = UserSchema.safeParse(data);
if (!result.success) {
  const tree = z.treeifyError(result.error);   // Nested structure matching schema
  const flat = z.flattenError(result.error);   // { formErrors: [], fieldErrors: {} }
  const pretty = z.prettifyError(result.error); // Human-readable string
}
```

## Vue 3 props integration has known limitations

Using `z.infer<typeof Schema>` with `defineProps` encounters Vue's SFC compiler constraints. The compiler cannot resolve complex utility types imported from external files, producing the error: `[@vue/compiler-sfc] Unresolvable type reference`.

**Working pattern—inline type declaration**:
```vue
<script setup lang="ts">
import { z } from 'zod'

const PropsSchema = z.object({
  userId: z.string().uuid(),
  config: z.object({ theme: z.enum(['light', 'dark']) })
})

// Type alias MUST be in same file
type Props = z.infer<typeof PropsSchema>
const props = defineProps<Props>()
</script>
```

**Production pattern—export interface separately**:
```typescript
// schemas/user.ts
export const UserSchema = z.object({ name: z.string(), email: z.string().email() });
export interface User { name: string; email: string; } // Explicit interface
```

For **runtime validation** (since `defineProps` is compile-time only), implement validation in `onMounted` or via prop validators:

```typescript
defineProps({
  status: {
    type: String as PropType<z.infer<typeof StatusSchema>>,
    validator: (value: string) => StatusSchema.safeParse(value).success
  }
})
```

VeeValidate provides the most mature integration via `@vee-validate/zod`. The `toTypedSchema()` wrapper enables fully typed form values with automatic default extraction from Zod schemas.

## Nuxt Content validates schemas at build time

Nuxt Content v3 integrates Zod directly through `defineCollection` in `content.config.ts`. Schemas are converted to JSON Schema Draft-07 internally, and all frontmatter is validated during `nuxi generate`:

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { z } from 'zod' // Import directly—@nuxt/content z re-export is deprecated

export default defineContentConfig({
  collections: {
    blog: defineCollection({
      type: 'page',
      source: 'blog/**/*.md',
      schema: z.object({
        title: z.string(),
        date: z.date(),
        draft: z.boolean().default(false),
        tags: z.array(z.string()).optional(),
        category: z.enum(['tech', 'life', 'travel']).optional()
      })
    })
  }
})
```

The `queryCollection` composable provides **fully typed queries** inferred from your Zod schemas:

```typescript
const { data: posts } = await useAsyncData('blog', () =>
  queryCollection('blog')
    .where('draft', '=', false)
    .order('date', 'DESC')
    .select('title', 'path', 'date')
    .all()
)
// posts is typed as Array<{ title: string; path: string; date: Date; ... }>
```

For SSG deployment on Cloudflare Pages, the content database bundles as **WASM SQLite**. Initial page loads serve pre-rendered HTML; client-side navigation queries the bundled database. No D1 or server functions required for pure static hosting.

## Zod Mini cuts bundles by 85% for client code

Standard Zod 4's method-chaining API resists tree-shaking—even `z.boolean()` pulls in `.optional()`, `.array()`, and other method implementations. **Zod Mini** (`zod/mini`) uses a functional API enabling full tree-shaking:

| Scenario | Standard Zod | Zod Mini | Reduction |
|----------|-------------|----------|-----------|
| Core bundle (gzipped) | ~5.36 KB | ~1.88 KB | **65%** |
| Object with 3 fields | ~13.1 KB | ~4.0 KB | **69%** |

```typescript
import * as z from "zod/mini";

// Functional API with .check()
const UserSchema = z.object({
  email: z.string().check(z.minLength(1), z.email()),
  age: z.number().check(z.minimum(0), z.maximum(150))
});
```

**Trade-off**: Zod Mini has no default English locale—errors show "Invalid input" unless you configure `z.config(z.locales.en())`. Use standard Zod for backend/build-time code where bundle size is irrelevant.

## Zod 4 migration requires attention to several breaking changes

The unified error API replaces `message`, `invalid_type_error`, and `required_error` with a single `error` parameter accepting strings or functions:

```typescript
// Zod 3 (deprecated)
z.string({ invalid_type_error: "Not a string", required_error: "Required" });

// Zod 4
z.string({ error: issue => issue.input === undefined ? "Required" : "Not a string" });
```

Key API changes to migrate:

- `z.string().email()` → `z.email()` (top-level)
- `z.nativeEnum(Enum)` → `z.enum(Enum)`
- `z.object().strict()` → `z.strictObject()`
- `z.record(valueSchema)` → `z.record(keySchema, valueSchema)` (two args required)
- `.merge()` deprecated → use `.extend()` or spread: `z.object({ ...A.shape, ...B.shape })`
- `z.coerce.string()` input type changed from `string` to `unknown`

A community codemod automates most migrations: `npx zod-v3-to-v4`. For gradual adoption, Zod 4 supports subpath imports (`zod/v3`, `zod/v4`) allowing both versions to coexist.

## Performance patterns for SSG production builds

Schema definition location matters significantly. Creating schemas inside components or functions causes recreation on every render—define schemas at module level and import them:

```typescript
// ❌ Anti-pattern: schema recreation
function validateForm(data: unknown) {
  const schema = z.object({ name: z.string() }); // Created every call
  return schema.parse(data);
}

// ✅ Best practice: module-level definition
const FormSchema = z.object({ name: z.string() });
function validateForm(data: unknown) {
  return FormSchema.parse(data);
}
```

For complex schema hierarchies, compose base schemas via object spread rather than chaining `.extend()` calls:

```typescript
const BaseSchema = z.object({ id: z.string(), created: z.date() });
const UserSchema = z.object({ ...BaseSchema.shape, name: z.string(), email: z.email() });
```

Zod 4's **JSON Schema export** (`z.toJSONSchema()`) enables build-time compilation to optimized validators for high-throughput scenarios. One production API gateway reported CPU usage dropping from **80% to 20%** after pre-compiling schemas to AJV validators.

## Conclusion

Zod 4 with Vue 3/Nuxt 4 delivers type-safe validation with excellent developer experience. Key takeaways:

- **Use `z.input<>` for form types** when schemas include transforms; `z.infer<>` returns output types
- **Keep Zod type aliases in the same file** as `defineProps` to avoid Vue compiler issues  
- **Nuxt Content validates at build time**—no runtime cost for SSG; invalid frontmatter fails the build
- **Use Zod Mini for client bundles** under size constraints; standard Zod for build/server code
- **Migrate `.default()` carefully**—it now expects output types and short-circuits; use `.prefault()` for input-type defaults
- **Define schemas at module level** to avoid recreation overhead; compose via object spread for best TypeScript performance