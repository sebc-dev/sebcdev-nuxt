# Vitest testing with Nuxt 4: The complete configuration guide

Nuxt 4's testing story has matured significantly with `@nuxt/test-utils` v3.21.0 (December 2025), offering seamless Vitest integration that properly handles auto-imports, Suspense boundaries, and the new `app/` directory structure. The key insight for production setups is using **Vitest projects** (not the deprecated workspace) to separate fast Node-based unit tests from slower Nuxt environment tests—achieving both speed and full framework integration.

## Core installation and baseline configuration

Install the required packages with pnpm:

```bash
pnpm add -D @nuxt/test-utils vitest @vue/test-utils happy-dom @vitest/coverage-v8 playwright-core
```

Create `vitest.config.ts` in your project root. This configuration separates unit tests (running in fast Node environment) from Nuxt tests (requiring full framework):

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import { defineVitestProject } from '@nuxt/test-utils/config'

export default defineConfig({
  test: {
    projects: [
      // Fast unit tests - Node environment
      {
        test: {
          name: 'unit',
          include: ['test/unit/**/*.{test,spec}.ts'],
          exclude: ['**/*.nuxt.test.ts'],
          environment: 'node',
          globals: true,
        },
      },
      // Nuxt component/integration tests - Nuxt environment
      await defineVitestProject({
        test: {
          name: 'nuxt',
          include: ['test/nuxt/**/*.{test,spec}.ts', '**/*.nuxt.{test,spec}.ts'],
          environment: 'nuxt',
          environmentOptions: {
            nuxt: {
              domEnvironment: 'happy-dom',
              mock: {
                intersectionObserver: true,
                indexedDb: true,
              },
            },
          },
        },
      }),
    ],
  },
})
```

**Critical requirement**: The file must have `"type": "module"` in `package.json`, or rename to `vitest.config.mts`. The `await defineVitestProject()` wrapper is mandatory for proper alias resolution—missing this causes TypeScript and import errors.

Add the test-utils module to `nuxt.config.ts` for DevTools integration:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/test-utils/module'],
})
```

## Nuxt 4 directory structure affects test organization

Nuxt 4 defaults `srcDir` to `app/`, fundamentally changing how paths resolve. Tests should remain at the project root, not inside `app/`:

```
my-nuxt-app/
├── app/                     # srcDir - all application code
│   ├── components/
│   ├── composables/
│   ├── pages/
│   └── app.vue
├── test/                    # Tests at root level
│   ├── unit/               # Pure logic tests (node environment)
│   │   └── utils.test.ts
│   └── nuxt/               # Component tests (nuxt environment)
│       └── MyComponent.nuxt.test.ts
├── vitest.config.ts
└── nuxt.config.ts
```

The `~` and `@` aliases now point to `app/`, not the project root. Tests in `test/nuxt/` automatically receive Nuxt's TypeScript context and recognize auto-imports.

## The three essential testing helpers

### mountSuspended() for async component testing

Unlike `@vue/test-utils` `mount()`, **mountSuspended** wraps components in `<Suspense>` and initializes the full Nuxt runtime—making auto-imports, plugins, and composables available:

```typescript
// test/nuxt/ProductCard.nuxt.test.ts
import { mountSuspended } from '@nuxt/test-utils/runtime'
import ProductCard from '~/components/ProductCard.vue'

it('renders product data', async () => {
  const wrapper = await mountSuspended(ProductCard, {
    props: {
      title: 'Widget Pro',
      price: 99.99,
    },
  })
  
  expect(wrapper.text()).toContain('Widget Pro')
  expect(wrapper.text()).toContain('$99.99')
})

// Testing at a specific route
import App from '~/app.vue'

it('renders the about page', async () => {
  const wrapper = await mountSuspended(App, { route: '/about' })
  expect(wrapper.text()).toContain('About Us')
})
```

**Key differences from @vue/test-utils mount()**:

| Feature | mount() | mountSuspended() |
|---------|---------|------------------|
| Async setup support | Manual | Built-in |
| Auto-imports | No | Yes |
| Nuxt composables | No | Yes |
| Return type | Sync | **Async (must await)** |

The most common gotcha is forgetting to `await`—mountSuspended is always asynchronous.

### mockNuxtImport() for auto-import mocking

This macro mocks any auto-imported composable. Because it transforms to `vi.mock` and gets hoisted, it must appear at module level:

```typescript
import { vi } from 'vitest'
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'

// For dynamic mock behavior across tests, use vi.hoisted
const { mockRoute } = vi.hoisted(() => ({
  mockRoute: vi.fn(),
}))

mockNuxtImport('useRoute', () => mockRoute)

afterEach(() => {
  mockRoute.mockReset()
})

test('displays user ID from route', async () => {
  mockRoute.mockReturnValue({ params: { id: '42' } })
  const wrapper = await mountSuspended(UserPage)
  expect(wrapper.text()).toContain('User #42')
})

test('handles missing ID', async () => {
  mockRoute.mockReturnValue({ params: {} })
  const wrapper = await mountSuspended(UserPage)
  expect(wrapper.text()).toContain('No user selected')
})
```

**Critical gotchas**:
- Can only be called **once per import per file**—second calls are ignored
- The factory must return a **function** for composables: `() => () => value`, not `() => value`
- Cannot reference local variables due to hoisting—use `vi.hoisted()` for anything dynamic

### registerEndpoint() for API mocking

Creates mock Nitro endpoints for testing components that fetch data:

```typescript
import { registerEndpoint, mountSuspended } from '@nuxt/test-utils/runtime'
import UserList from '~/components/UserList.vue'

registerEndpoint('/api/users', () => ([
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
]))

registerEndpoint('/api/users', {
  method: 'POST',
  handler: () => ({ id: 3, name: 'Charlie', created: true }),
})

it('displays fetched users', async () => {
  const wrapper = await mountSuspended(UserList)
  expect(wrapper.text()).toContain('Alice')
  expect(wrapper.text()).toContain('Bob')
})
```

**Limitation**: Endpoints cannot change between tests once registered. For dynamic API mocking, use **MSW (Mock Service Worker)** instead.

## Testing Nuxt composables effectively

### useState testing

```typescript
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'
import { ref } from 'vue'

mockNuxtImport('useState', () => {
  return (key: string, init?: () => any) => ref(init ? init() : undefined)
})

it('initializes counter state', async () => {
  const wrapper = await mountSuspended(CounterComponent)
  expect(wrapper.text()).toContain('Count: 0')
})
```

### useAsyncData and useFetch testing

Mock the composables directly for controlled test scenarios:

```typescript
import { vi } from 'vitest'
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'
import { ref } from 'vue'

const { mockUseFetch } = vi.hoisted(() => ({
  mockUseFetch: vi.fn(),
}))

mockNuxtImport('useFetch', () => mockUseFetch)

afterEach(() => mockUseFetch.mockReset())

test('shows loading state', async () => {
  mockUseFetch.mockReturnValue({
    data: ref(null),
    pending: ref(true),
    error: ref(null),
  })
  
  const wrapper = await mountSuspended(ArticleList)
  expect(wrapper.text()).toContain('Loading...')
})

test('renders fetched articles', async () => {
  mockUseFetch.mockReturnValue({
    data: ref([{ title: 'Testing Guide' }]),
    pending: ref(false),
    error: ref(null),
  })
  
  const wrapper = await mountSuspended(ArticleList)
  expect(wrapper.text()).toContain('Testing Guide')
})
```

## Testing Nuxt Content 3 components

Nuxt Content 3 replaced `queryContent()` with `queryCollection()` and removed most built-in components except `<ContentRenderer>`. Mock the data layer:

```typescript
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'
import { ref } from 'vue'
import BlogPost from '~/pages/blog/[slug].vue'

mockNuxtImport('useAsyncData', () => {
  return () => ({
    data: ref({
      title: 'Test Article',
      body: {
        type: 'root',
        children: [
          { type: 'element', tag: 'p', children: [{ type: 'text', value: 'Article content' }] }
        ],
      },
    }),
    pending: ref(false),
    error: ref(null),
  })
})

it('renders content page', async () => {
  const wrapper = await mountSuspended(BlogPost, {
    route: '/blog/test-article',
  })
  expect(wrapper.text()).toContain('Test Article')
})
```

For stubbing ContentRenderer entirely:

```typescript
import { mockComponent, mountSuspended } from '@nuxt/test-utils/runtime'

mockComponent('ContentRenderer', {
  props: ['value'],
  template: '<div class="content-stub">{{ value?.title }}</div>',
})
```

## Coverage configuration for SSG projects

SSG projects face a specific challenge: Nuxt auto-imports all components during test setup, potentially inflating coverage. **Always explicitly define coverage.include**:

```typescript
// vitest.config.ts
import { defineVitestConfig } from '@nuxt/test-utils/config'

export default defineVitestConfig({
  test: {
    environment: 'nuxt',
    coverage: {
      provider: 'v8',
      enabled: true,
      
      // Explicit inclusion prevents auto-import inflation
      include: [
        'app/components/**/*.vue',
        'app/composables/**/*.ts',
        'app/utils/**/*.ts',
      ],
      
      exclude: [
        '**/.nuxt/**',
        '**/.output/**',
        '**/node_modules/**',
        '**/*.d.ts',
        'test/**',
      ],
      
      reporter: ['text', 'html', 'lcov'],
      reportsDirectory: './coverage',
      
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 75,
        statements: 80,
        perFile: true,
      },
      
      reportOnFailure: true,
    },
  },
})
```

**Vitest 4 change**: The `coverage.all` option is removed. Only files covered by tests appear in reports by default.

## Performance optimization strategies

For large test suites, these optimizations can cut execution time by **50-70%**:

```typescript
export default defineConfig({
  test: {
    pool: 'threads',          // Faster than 'forks' for most cases
    maxWorkers: 4,            // Match CPU cores
    isolate: false,           // Disable if tests don't share state
    css: false,               // Skip CSS processing if not testing styles
  },
})
```

**Use test sharding in CI** for parallel execution across machines:

```yaml
# .github/workflows/test.yml
jobs:
  test:
    strategy:
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - run: pnpm test --reporter=blob --shard=${{ matrix.shard }}/4
      - uses: actions/upload-artifact@v4
        with:
          name: blob-report-${{ matrix.shard }}
          path: .vitest-reports/*
  
  merge:
    needs: test
    steps:
      - uses: actions/download-artifact@v4
      - run: npx vitest --merge-reports
```

**Always use happy-dom over jsdom**—it's significantly faster for component testing with minimal compatibility trade-offs.

## Breaking changes from Nuxt 3 to Nuxt 4

Three changes require test updates:

**1. Data fetching returns undefined, not null**

```typescript
// Nuxt 3
expect(data.value).toBe(null)

// Nuxt 4
expect(data.value).toBe(undefined)
```

**2. Component names are normalized**

Vue now generates component names matching Nuxt's auto-import pattern (folder + filename):

```typescript
// Update findComponent selectors
wrapper.findComponent({ name: 'FolderMyComponent' })  // Not just 'MyComponent'
```

**3. Vitest workspace is deprecated**

Replace `vitest.workspace.ts` with `projects` array in `vitest.config.ts`. The deprecated syntax still works but will be removed in a future major version.

Run automated codemods for migration:

```bash
npx codemod@latest nuxt/4/file-structure
npx codemod@latest nuxt/4/default-data-error-value
```

## Recommended package.json scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:unit": "vitest --project unit",
    "test:nuxt": "vitest --project nuxt",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
}
```

## Conclusion

Effective Nuxt 4 testing hinges on **proper environment separation**—use Vitest projects to run pure logic tests in Node for speed, while component tests get the full Nuxt runtime via `environment: 'nuxt'`. The `mountSuspended`, `mockNuxtImport`, and `registerEndpoint` helpers solve the fundamental challenge of testing within Nuxt's auto-import and async-first architecture.

Key takeaways for production readiness: always `await mountSuspended()`, use `vi.hoisted()` for dynamic mocks with `mockNuxtImport`, explicitly define coverage include paths to avoid SSG inflation, and leverage happy-dom plus test sharding for performance. The `app/` directory structure in Nuxt 4 requires updating test file patterns but otherwise maintains backward compatibility with Nuxt 3 testing patterns.