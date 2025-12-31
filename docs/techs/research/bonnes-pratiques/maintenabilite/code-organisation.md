# Vue 3 / Nuxt 4 / TypeScript code organization best practices

Modern Vue 3 and Nuxt 4 development demands thoughtful architectural decisions that balance maintainability with developer experience. **The most significant structural change in Nuxt 4 is the new `app/` directory that separates frontend code from server code**, improving IDE type-safety and build performance. For code organization, small-to-medium projects should embrace Nuxt's default type-based structure, while larger applications benefit from feature-based organization using Nuxt Layers. Composables remain the primary extraction pattern for reusable logic, with the ~200 line component limit serving as a reliable splitting threshold. TypeScript strict mode, combined with runtime validation through Zod, eliminates the `any` type anti-pattern that undermines type safety.

## Nuxt 4's new directory structure changes everything

The most impactful change in Nuxt 4.2.x is the separation of `app/` from `server/` at the root level. This isn't merely organizational—it provides **significant performance gains** by reducing filesystem watcher load and ensures correct TypeScript auto-completions since app and server code run in different contexts with different global imports.

```
my-nuxt-app/
├─ app/                     # srcDir - all frontend code
│  ├─ components/           # Auto-imported Vue components
│  ├─ composables/          # Auto-imported composables
│  ├─ layouts/              # Page layouts
│  ├─ pages/                # File-based routing
│  ├─ plugins/              # Vue plugins
│  └─ utils/                # Auto-imported utilities
├─ content/                 # Nuxt Content module files
├─ layers/                  # Auto-registered local layers (DDD)
├─ public/                  # Static assets served at root
├─ server/                  # Nitro server code
│  ├─ api/                  # API routes
│  └─ utils/                # Server utilities
├─ shared/                  # Code shared between app and server
└─ nuxt.config.ts
```

Key breaking changes from Nuxt 3 include the `srcDir` defaulting to `app/` rather than root, the server directory moving to `<rootDir>/server` instead of `<srcDir>/server`, and the `~` alias now pointing to `app/`. Projects migrating from Nuxt 3 can temporarily revert by setting `srcDir: '.'` in nuxt.config.ts, but adopting the new structure is recommended for long-term maintainability.

For SSG deployments on Cloudflare Pages, configure Nitro with the `cloudflare_pages` preset and enable route prerendering. The build command becomes `nuxt generate` with output directory `.output/public`. This structure cleanly separates static content in `content/` from build-time processed assets in `app/assets/` and truly static files in `public/`.

## Feature-based versus type-based organization depends on scale

Type-based organization groups files by their technical role—all components together, all composables together. Feature-based organization groups files by domain—all authentication-related code together regardless of whether it's a component, composable, or type. **The choice depends entirely on project scale and team structure.**

For projects with fewer than 50-100 components, the default Nuxt type-based structure excels. Navigation via editor quick-open (`CMD+P`) works efficiently, and the familiar structure reduces onboarding friction. Nuxt's auto-import system works seamlessly, generating component names from folder paths (e.g., `components/base/Button.vue` becomes `<BaseButton />`).

```
app/
├─ components/
│  ├─ base/                 # BaseButton, BaseCard, BaseInput
│  ├─ auth/                 # LoginForm, SignupForm
│  └─ dashboard/            # DashboardStats, DashboardChart
├─ composables/
│  ├─ useAuth.ts
│  └─ useUser.ts
└─ pages/
```

For larger projects or those with multiple teams, **Nuxt Layers provide first-class support for domain-driven design**. Each layer acts as a self-contained mini-application with its own components, composables, pages, and even server routes. Layers in the `layers/` directory are auto-registered in Nuxt 4, eliminating manual configuration.

```
layers/
├─ auth/
│  ├─ app/
│  │  ├─ components/LoginForm.vue
│  │  ├─ composables/useAuth.ts
│  │  └─ pages/login.vue
│  ├─ server/api/auth/
│  └─ nuxt.config.ts
├─ dashboard/
│  ├─ app/
│  └─ nuxt.config.ts
└─ shared/                  # Base components and utilities
```

The critical anti-pattern to avoid is **premature feature organization**. Creating complex folder hierarchies for a small project adds cognitive overhead without benefit. Start with the type-based default and migrate to layers when genuine domain boundaries emerge and team isolation becomes valuable.

## Composable extraction follows clear patterns

Composables should be extracted when logic involves **state that changes over time** and appears in multiple components. The "rule of three" provides a useful heuristic: when the same reactive pattern appears in three or more components, extract it. However, composables also serve as an organizational tool within a single large component, even without reuse.

Naming conventions are well-established: function names use `camelCase` with the `use` prefix (e.g., `useAuth`), file names use the same pattern (`useAuth.ts`), and interface names follow `Use{Name}Options` for parameters and `Use{Name}Return` for return types. All named exports and default exports from the `composables/` directory are auto-imported in Nuxt 4.

```typescript
// composables/useAuth.ts
interface UseAuthReturn {
  user: Ref<User | null>
  isAuthenticated: ComputedRef<boolean>
  login: (credentials: Credentials) => Promise<void>
  logout: () => void
}

export function useAuth(): UseAuthReturn {
  const user = ref<User | null>(null)
  const isAuthenticated = computed(() => !!user.value)
  
  async function login(credentials: Credentials) {
    const { data } = await useFetch('/api/auth/login', {
      method: 'POST',
      body: credentials
    })
    user.value = data.value
  }
  
  function logout() {
    user.value = null
    navigateTo('/login')
  }
  
  return { user: readonly(user), isAuthenticated, login, logout }
}
```

The decision between composables and Pinia for state management follows a clear principle: **composables for non-singleton instances, Pinia for singleton instances**. When each component needs its own independent state, use a composable that creates state inside the function. When state must be shared globally with DevTools debugging and SSR hydration support, use Pinia. The two approaches complement each other—Pinia's setup stores can consume composables, and composables can wrap Pinia stores to provide a cleaner interface.

A critical anti-pattern is defining reactive state **outside** the composable function, which creates an unintentional singleton shared across all consumers:

```typescript
// ❌ SINGLETON - state shared across ALL components
const globalCount = ref(0)
export function useCounter() {
  return { count: globalCount }
}

// ✅ INSTANCE - each component gets its own state
export function useCounter() {
  const count = ref(0)  // Created inside function
  return { count }
}
```

For nested composables, use the `MaybeRefOrGetter` type with `toValue()` to accept refs, getters, or raw values as arguments. This pattern, established by VueUse, maximizes flexibility while maintaining type safety.

## Component splitting targets the 200-line threshold

The community consensus recommends keeping Vue SFCs under **200 lines total**, with some stricter guidelines suggesting 100 lines. The `eslint-plugin-vue` enforces this through the `vue/max-lines-per-block` rule, which can set separate limits for template, script, and style sections.

When a component exceeds this threshold, evaluate three splitting strategies in order of preference. First, extract logic into composables—this addresses script bloat without creating new components. Second, extract UI sections into child components when template sections are independently meaningful. Third, apply the smart/presentational pattern when a component both fetches data and renders complex UI.

```vue
<!-- Before: Monolithic component -->
<script setup lang="ts">
const user = ref(null)
const isAuthenticated = computed(() => !!user.value)
const login = async () => { /* ... */ }
const logout = () => { /* ... */ }

const projects = ref([])
const loading = ref(false)
const fetchProjects = async () => { /* ... */ }

const tasks = ref([])
const addTask = () => { /* ... */ }
</script>

<!-- After: Using inline composables for organization -->
<script setup lang="ts">
const useAuth = () => {
  const user = ref(null)
  const isAuthenticated = computed(() => !!user.value)
  const login = async () => { /* ... */ }
  const logout = () => { /* ... */ }
  return { user, isAuthenticated, login, logout }
}

const useProjects = () => {
  const projects = ref([])
  const loading = ref(false)
  const fetchProjects = async () => { /* ... */ }
  return { projects, loading, fetchProjects }
}

const { user, isAuthenticated, login, logout } = useAuth()
const { projects, loading, fetchProjects } = useProjects()
</script>
```

Smart (container) components handle state changes, fetch data, and manage complex interactions. They typically live in `pages/` or feature directories. Presentational (dumb) components focus purely on appearance, receive data via props, emit events upward, and never modify parent state directly. They're highly reusable and testable, living in `components/` with prefixes like `Base`.

For **props drilling versus provide/inject**, the guideline is straightforward: use props for 1-2 levels deep (always), consider provide/inject for 3+ levels. When using provide/inject, define typed `InjectionKey` symbols and provide `readonly()` refs to prevent uncontrolled mutation.

```typescript
// keys.ts
export const ThemeKey: InjectionKey<ThemeContext> = Symbol('Theme')

// Provider
const theme = ref<'light' | 'dark'>('light')
provide(ThemeKey, {
  theme: readonly(theme),
  toggleTheme: () => { theme.value = theme.value === 'light' ? 'dark' : 'light' }
})

// Consumer (any depth)
const themeContext = inject(ThemeKey)
if (!themeContext) throw new Error('ThemeProvider not found')
```

Component naming in Nuxt 4 follows path-based auto-import: `components/base/Button.vue` becomes `<BaseButton />`, `components/user/avatar/Image.vue` becomes `<UserAvatarImage />`. The `The` prefix indicates single-instance components (`TheHeader.vue`), while `Base` indicates generic reusable components.

## Type safety requires eliminating every `any`

TypeScript strict mode is enabled by default in Nuxt 4, but achieving true type safety requires attention to common escape hatches. The most frequent sources of `any` are untyped event handlers, untyped refs with empty initial values, API responses, and third-party libraries without type definitions.

```typescript
// ❌ Event handler with implicit any
function handleChange(event) {
  console.log(event.target.value)
}

// ✅ Explicit typing
function handleChange(event: Event) {
  console.log((event.target as HTMLInputElement).value)
}

// ❌ Ref with untyped initial value
const data = ref({})

// ✅ Explicit type annotation
const data = ref<User | null>(null)
```

For props, emits, and slots, Vue 3.3+ introduced type-based declarations that provide compile-time safety without runtime overhead. The `defineProps<T>()`, `defineEmits<T>()`, and `defineSlots<T>()` macros infer types from their generic parameters.

```typescript
<script setup lang="ts">
interface Props {
  title: string
  count?: number
  items: Item[]
}

// Vue 3.5+ reactive props destructure
const { title, count = 0, items = [] } = defineProps<Props>()

// Typed emits with Vue 3.3+ succinct syntax
const emit = defineEmits<{
  change: [id: number]
  update: [value: string, metadata?: object]
  'update:modelValue': [value: string]
}>()

// Typed slots
const slots = defineSlots<{
  default: (props: { message: string }) => any
  item: (props: { product: Product; index: number }) => any
}>()
</script>
```

Generic components, introduced in Vue 3.3, enable type-safe reusable components that adapt to their input types. The `generic` attribute on script setup declares type parameters:

```typescript
<script setup lang="ts" generic="T extends { id: string | number }">
interface Props {
  items: T[]
  selected?: T
}

const props = defineProps<Props>()
const emit = defineEmits<{ select: [item: T] }>()

const isSelected = (item: T) => item.id === props.selected?.id
</script>
```

For runtime validation that bridges TypeScript's compile-time guarantees with actual data, **Zod integration is essential**. Define schemas that both validate data and infer TypeScript types:

```typescript
import { z } from 'zod'

export const UserSchema = z.object({
  id: z.number(),
  name: z.string().min(2),
  email: z.string().email(),
  role: z.enum(['admin', 'user', 'guest'])
})

export type User = z.infer<typeof UserSchema>

// API route with validation
export default defineEventHandler(async (event) => {
  const body = await readBody(event)
  const result = UserSchema.safeParse(body)
  
  if (!result.success) {
    throw createError({ statusCode: 400, data: result.error.flatten() })
  }
  
  return await createUser(result.data)  // Fully typed
})
```

The `satisfies` operator, introduced in TypeScript 4.9, validates types while preserving literal inference—use it for configuration objects and directives where you want type checking without widening.

## ESLint flat config replaces legacy configuration

ESLint 9.0 made flat config (`eslint.config.mjs`) the default, with legacy `.eslintrc` support removed in ESLint 10.0. For Nuxt 4 projects, the `@nuxt/eslint` module generates project-aware flat configs automatically.

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/eslint'],
  eslint: {
    config: {
      stylistic: true  // Enable ESLint Stylistic formatting
    }
  }
})

// eslint.config.mjs
import withNuxt from './.nuxt/eslint.config.mjs'

export default withNuxt({
  files: ['**/*.ts', '**/*.vue'],
  rules: {
    'no-console': 'warn'
  }
})
```

Anthony Fu's `@antfu/eslint-config` has emerged as the most popular opinionated preset with **5.9k GitHub stars**. It uses ESLint Stylistic for formatting, eliminating the need for Prettier. The configuration enables single quotes, no semicolons, sorted imports, and dangling commas by default—all auto-fixable on save.

```javascript
// eslint.config.mjs
import antfu from '@antfu/eslint-config'
import withNuxt from './.nuxt/eslint.config.mjs'

export default withNuxt(
  antfu({
    stylistic: { indent: 2, quotes: 'single' },
    typescript: true,
    vue: true,
  })
)
```

The shift away from Prettier reflects a broader ecosystem trend. Prettier's "read-and-reprint" approach loses original formatting intent, while ESLint Stylistic provides fine-grained control with a single tool for both linting and formatting. VS Code settings should disable Prettier and enable ESLint fix-on-save.

## Barrel files require careful consideration

Barrel files (`index.ts` re-exports) provide convenient import paths but create **significant bundle optimization problems**. Importing a single component from a barrel file loads all exports, breaking tree-shaking. This is particularly problematic for component libraries and large applications.

Avoid barrel files for components entirely in large projects. Direct imports preserve tree-shaking:

```typescript
// ✅ Direct import - tree-shakeable
import BaseButton from '@/components/base/BaseButton.vue'

// ❌ Barrel import - may load entire directory
import { BaseButton } from '@/components'
```

Barrel files are acceptable for **types only** (no runtime code), **small cohesive modules** (3-5 tightly related exports), and **feature folder public APIs** where the entire feature is commonly used together. Add `sideEffects: false` to package.json and use the `exports` field for granular control over import paths.

## Conclusion

Effective Vue 3 / Nuxt 4 / TypeScript code organization centers on a few key decisions: embrace the new `app/` directory structure for its performance and type-safety benefits; start with type-based organization and migrate to Nuxt Layers when genuine domain boundaries emerge; extract composables at the ~200 line component threshold following the `use` prefix convention; eliminate `any` through strict mode, type-based declarations, and Zod validation; and adopt ESLint flat config with stylistic rules, abandoning Prettier for a unified tooling approach.

The most common anti-patterns to avoid are premature feature organization, singleton state leaks in composables, mixing runtime and type-based prop declarations, and barrel files for components. For SSG deployments, ensure DOM access occurs only in `onMounted` or behind `import.meta.client` guards, and use `<ClientOnly>` for components that cannot render on the server.

These patterns reflect the Vue/Nuxt ecosystem's maturation toward opinionated conventions that balance flexibility with maintainability—a balance essential for long-term project health.