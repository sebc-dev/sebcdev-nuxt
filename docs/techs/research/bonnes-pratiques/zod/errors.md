# Zod 4 error handling for Nuxt 4 SSG on Cloudflare Pages

**Zod 4 delivers a completely redesigned error system** with native `prettifyError()`, unified localization via `z.config()`, and **57% smaller bundles** (5.36kb gzipped). For your Nuxt 4/Cloudflare Pages stack, the critical strategy is: validate content at build time with full Zod 4, use `zod/mini` (1.88kb) for any client-side form validation, and leverage VeeValidate's `@vee-validate/zod` adapter for shadcn-vue integration. The migration from Zod 3 requires updating all `message` and `errorMap` parameters to the new unified `error` parameter.

## Core API: safeParse() vs parse() patterns

The fundamental choice between `parse()` and `safeParse()` determines your entire error handling architecture. Use **`parse()`** when invalid data should halt execution—ideal for build-time content validation where you want the SSG build to fail on schema violations. Use **`safeParse()`** for form validation and user input where you need graceful error display.

```typescript
// safeParse returns a discriminated union—never throws
const result = schema.safeParse(userInput);
if (!result.success) {
  const errors = z.flattenError(result.error);
  // { formErrors: string[], fieldErrors: { email: string[], password: string[] } }
} else {
  // TypeScript knows result.data is fully typed here
  submitForm(result.data);
}
```

Zod 4 introduces **per-parse error customization**, letting you override messages contextually without modifying the schema:

```typescript
schema.safeParse(data, {
  error: (iss) => {
    if (iss.code === "invalid_type") return `Expected ${iss.expected}`;
    if (iss.code === "too_small") return `Minimum is ${iss.minimum}`;
  }
});
```

## prettifyError() and error formatting utilities

Zod 4 introduces **`z.prettifyError()`** as a native replacement for the `zod-validation-error` package. It produces human-readable multi-line output:

```typescript
if (!result.success) {
  console.log(z.prettifyError(result.error));
}
// Output:
// ✖ Invalid input: expected string, received number
//   → at username
// ✖ Invalid input: expected number, received string
//   → at favoriteNumbers[1]
```

For form integration, Zod 4 provides **three formatting utilities** with distinct use cases:

| Utility | Use Case | Structure |
|---------|----------|-----------|
| `z.flattenError()` | Flat forms | `{ formErrors: [], fieldErrors: { field: [] } }` |
| `z.treeifyError()` | Nested objects | Mirrors schema structure with `.properties` |
| `z.prettifyError()` | Logging/debugging | Human-readable string |

**Breaking change**: The Zod 3 `.format()` method returning `{ _errors: [], field: { _errors: [] } }` is deprecated. Replace with `z.treeifyError()`:

```typescript
// Zod 3 (deprecated)
const formatted = result.error.format();
formatted.email?._errors;

// Zod 4
const tree = z.treeifyError(result.error);
tree.properties?.email?.errors;
```

## Localization with z.config() and 40+ built-in locales

Zod 4 ships with **40+ built-in locales** including French (`fr`) and Canadian French (`frCA`). Configure globally using `z.config()`:

```typescript
import * as z from "zod";

// Load French locale
z.config(z.locales.fr());

// Or lazy-load for bundle optimization
async function setLocale(locale: string) {
  const { default: localeModule } = await import(`zod/v4/locales/${locale}.js`);
  z.config(localeModule());
}
```

Custom error messages follow a **precedence hierarchy** (highest to lowest): schema-level → per-parse → global config → locale. For multilingual blogs, define schema-level messages for field-specific overrides and rely on locale for defaults:

```typescript
const postSchema = z.object({
  title: z.string({ 
    error: (iss) => iss.input === undefined 
      ? "Le titre est requis" 
      : "Titre invalide" 
  }),
  slug: z.string().regex(/^[a-z0-9-]+$/, { error: "URL slug format invalide" })
});
```

**Critical caveat for zod/mini**: It ships with **no default locale**—all errors display "Invalid input" unless you explicitly load one with `z.config(z.locales.en())`.

## Breaking changes from Zod 3.x requiring migration

The most impactful breaking changes for your codebase involve error handling and schema APIs:

**Error API unification** replaces multiple parameters with a single `error`:
```typescript
// Zod 3 (broken in v4)
z.string({ required_error: "Required", invalid_type_error: "Not a string" });
z.string().min(5, { message: "Too short" });

// Zod 4
z.string({ error: (iss) => iss.input === undefined ? "Required" : "Not a string" });
z.string().min(5, { error: "Too short" });
```

**Object methods deprecated**—use top-level functions:
```typescript
// Zod 3
z.object({ name: z.string() }).strict();
schema1.merge(schema2);

// Zod 4
z.strictObject({ name: z.string() });
z.object({ ...schema1.shape, ...schema2.shape });
```

**String validators moved to top-level**:
```typescript
// Zod 3 (still works but deprecated)
z.string().email().uuid()

// Zod 4 preferred
z.email()
z.uuid()
```

**Behavioral changes** that may silently affect validation:
- `z.record()` now requires two arguments: `z.record(z.string(), z.string())`
- UUID validation follows RFC 9562 strictly—use `z.guid()` for permissive matching
- `.default()` applies to **output** after transforms (use `.prefault()` for old behavior)
- `z.number().int()` only accepts safe integers (±9007199254740991)

Run the community codemod for automated migration: `npx zod-v3-to-v4`

## zod/mini delivers 85% bundle reduction for client validation

For Cloudflare Pages SSG with bundle size constraints, **zod/mini at 1.88kb gzipped** (vs 5.36kb for Zod 4) is ideal for client-side validation:

```typescript
import * as z from "zod/mini";

// Functional API with .check() instead of method chaining
const contactSchema = z.object({
  email: z.string().check(z.email()),
  message: z.string().check(z.minLength(10), z.maxLength(500))
});

// Wrapper functions instead of methods
const optionalEmail = z.optional(z.email());
```

**Feature differences to consider**:
- No method chaining—uses functional `.check()` pattern
- No default English locale—must load explicitly
- Fully tree-shakable with ESM bundlers
- Compatible with all Zod 4 error formatting utilities

**Recommendation for your stack**: Use full Zod 4 for Nuxt Content schema validation (build-time only, not in bundle), use zod/mini for any client-side form validation in Vue components.

## Vue 3 and Nuxt 4 integration patterns

### VeeValidate + shadcn-vue (recommended approach)

The `@vee-validate/zod` adapter provides seamless integration with shadcn-vue's form components:

```vue
<script setup lang="ts">
import { useForm } from 'vee-validate';
import { toTypedSchema } from '@vee-validate/zod';
import { z } from 'zod';
import { FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Input } from '@/components/ui/input';

const schema = toTypedSchema(z.object({
  email: z.string().email('Adresse email invalide'),
  name: z.string().min(2, 'Nom trop court')
}));

const { handleSubmit, errors, defineField } = useForm({ validationSchema: schema });
const [email, emailAttrs] = defineField('email');
const [name, nameAttrs] = defineField('name');

const onSubmit = handleSubmit((values) => {
  // values is typed: { email: string; name: string }
  console.log(values);
});
</script>

<template>
  <form @submit="onSubmit">
    <FormField v-slot="{ componentField }" name="email">
      <FormItem>
        <FormLabel>Email</FormLabel>
        <Input type="email" v-bind="componentField" />
        <FormMessage />
      </FormItem>
    </FormField>
  </form>
</template>
```

### Nuxt Content frontmatter validation

Nuxt Content v3 natively supports Zod schemas in `content.config.ts`:

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content';
import { z } from 'zod';

export default defineContentConfig({
  collections: {
    blog: defineCollection({
      type: 'page',
      source: 'blog/**/*.md',
      schema: z.object({
        title: z.string(),
        description: z.string().optional(),
        date: z.date(),
        draft: z.boolean().default(false),
        tags: z.array(z.string()).optional(),
        lang: z.enum(['fr', 'en']).default('fr')
      })
    })
  }
});
```

Validation runs **at build time**—invalid frontmatter fails the SSG build with clear error messages. Query results are fully typed based on your schema.

### Shared validators for client and server

Place schemas in `shared/utils/` for isomorphic validation:

```typescript
// shared/utils/validators.ts
import { z } from 'zod';

export const contactSchema = z.object({
  email: z.string().email(),
  message: z.string().min(10)
});

export type ContactForm = z.infer<typeof contactSchema>;
```

## Anti-patterns and performance pitfalls

**Never recreate schemas inside components**—Zod 4 uses JIT compilation where first parse is slower:

```typescript
// ❌ Anti-pattern: 6 ops/ms
const MyComponent = () => {
  const schema = z.object({ name: z.string() }); // Recreated every render
  return schema.safeParse(data);
};

// ✅ Correct: 100+ ops/ms
const schema = z.object({ name: z.string() }); // Module-level
const MyComponent = () => schema.safeParse(data);
```

**Use safeParse for user input, parse for trusted data**:
```typescript
// ❌ Requires try/catch boilerplate
try { schema.parse(formData); } catch (e) { /* handle */ }

// ✅ Cleaner control flow
const result = schema.safeParse(formData);
if (!result.success) handleErrors(result.error);
```

**Avoid async transforms with sync parse**:
```typescript
// ❌ Throws runtime error
const schema = z.string().transform(async (val) => fetchUser(val));
schema.parse("id"); // Error!

// ✅ Use parseAsync
await schema.parseAsync("id");
```

**VeeValidate gotcha**: Zod's `refine()` and `superRefine()` don't execute when object keys are missing. Ensure all form fields have initial values or use `z.optional()`.

## SSG build-time validation strategy

For Cloudflare Pages SSG, structure validation to run **exclusively at build time**:

1. **Nuxt Content schemas** validate frontmatter during `nuxt generate`
2. **API data fetching** validates in `useAsyncData` during prerendering
3. **Client bundle** contains only zod/mini for form validation (if needed)

```typescript
// Build-time validation in Nuxt
export default defineNuxtConfig({
  nitro: {
    prerender: {
      // Content validation happens here
      routes: ['/blog', '/about']
    }
  }
});
```

**Cloudflare edge runtime caveat**: Zod 4's JIT compilation uses `new Function()`, which some edge runtimes restrict. For SSG deployments this is irrelevant since validation runs at build time, but avoid using Zod in edge functions unless testing confirms compatibility.

## Conclusion

Zod 4 represents a maturation of the library with unified error APIs, native formatting utilities, and significant bundle/performance improvements. For your Nuxt 4 + Cloudflare Pages stack:

- **Use full Zod 4** for Nuxt Content schemas (build-time only)
- **Use zod/mini** (1.88kb) for any client-side form validation
- **Integrate via VeeValidate** with `@vee-validate/zod` for shadcn-vue compatibility
- **Configure `z.locales.fr()`** globally for French error messages
- **Migrate error parameters** from `message`/`errorMap` to unified `error`
- **Define all schemas at module level** to leverage Zod 4's JIT optimization
- **Prefer `safeParse()`** with `z.flattenError()` for form validation workflows