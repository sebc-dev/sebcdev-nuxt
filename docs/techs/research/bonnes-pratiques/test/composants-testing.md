# Vue 3 / Nuxt 4 component testing: A comprehensive guide

**Vue Test Utils v2 with Vitest and happy-dom** delivers the fastest, most reliable testing experience for Nuxt 4 applications in 2025. This guide covers the complete testing stack for your shadcn-vue/Reka UI components with SSG deployment to Cloudflare Pages—from configuration to coverage enforcement.

The critical insight: Nuxt 4's testing ecosystem has matured significantly, with `@nuxt/test-utils` (v3.21+) absorbing the deprecated `nuxt-vitest` package and providing seamless integration with Vue Test Utils v2. The `happy-dom` environment is **2-10x faster** than jsdom for most tests, though jsdom remains necessary for accessibility testing with vitest-axe.

---

## Foundation: Vitest and Nuxt 4 configuration

The recommended setup uses Vitest projects to separate pure unit tests from Nuxt environment tests, enabling faster execution for logic-only code while providing full Nuxt context where needed.

### Complete vitest.config.ts

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import { defineVitestProject } from '@nuxt/test-utils/config'

export default defineConfig({
  test: {
    globals: true,
    projects: [
      // Pure unit tests - Node environment for speed
      {
        test: {
          name: 'unit',
          include: ['test/unit/**/*.{test,spec}.ts'],
          environment: 'node',
        },
      },
      // Nuxt component tests - happy-dom environment
      await defineVitestProject({
        test: {
          name: 'nuxt',
          include: ['test/nuxt/**/*.{test,spec}.ts', '**/*.nuxt.{test,spec}.ts'],
          environment: 'nuxt',
          environmentOptions: {
            nuxt: {
              rootDir: './',
              domEnvironment: 'happy-dom', // 'jsdom' for a11y tests
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

### Alternative simple configuration

For smaller projects, a single-environment approach works well:

```typescript
// vitest.config.ts
import { defineVitestConfig } from '@nuxt/test-utils/config'

export default defineVitestConfig({
  test: {
    environment: 'nuxt',
    globals: true,
    setupFiles: ['./test/setup.ts'],
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
})
```

### Nuxt 4 app/ directory structure for tests

```
my-nuxt-app/
├─ app/                    # srcDir (Nuxt 4 default)
│  ├─ components/
│  ├─ composables/
│  ├─ pages/
│  └─ utils/
├─ server/
├─ test/
│  ├─ nuxt/               # Nuxt environment tests
│  │  └─ components/
│  ├─ unit/               # Pure unit tests
│  └─ setup.ts
└─ vitest.config.ts
```

---

## @vue/test-utils patterns for script setup components

Vue Test Utils v2 introduces significant API changes from v1. The most impactful: all global configuration moves into a `global` object, and `createLocalVue()` is removed entirely.

### Core mounting pattern for script setup

```typescript
// UserCard.spec.ts
import { mount, flushPromises } from '@vue/test-utils'
import { describe, it, expect, vi } from 'vitest'
import UserCard from '~/components/UserCard.vue'

describe('UserCard', () => {
  it('renders full name from props', () => {
    const wrapper = mount(UserCard, {
      props: {              // 'props' replaces 'propsData' in v2
        firstName: 'John',
        lastName: 'Doe'
      }
    })
    
    expect(wrapper.find('h2').text()).toBe('John Doe')
  })

  it('emits userClicked event when button is clicked', async () => {
    const wrapper = mount(UserCard, {
      props: { firstName: 'Jane', lastName: 'Smith' }
    })

    await wrapper.find('button').trigger('click')

    expect(wrapper.emitted('userClicked')).toBeTruthy()
    expect(wrapper.emitted('userClicked')![0]).toEqual(['Jane Smith'])
  })
})
```

### Global configuration for Nuxt components

```typescript
const wrapper = mount(MyComponent, {
  global: {          // All global options nested here in v2
    plugins: [pinia],
    mocks: {
      $t: (key: string) => key,  // i18n mock
    },
    stubs: {
      NuxtLink: true,
      Teleport: true,
    },
    provide: {
      theme: 'dark'
    }
  }
})
```

### Breaking changes from Vue Test Utils v1 to v2

| v1 API | v2 API | Notes |
|--------|--------|-------|
| `propsData` | `props` | `propsData` still works for compatibility |
| `createLocalVue()` | Removed | Use `global.plugins` instead |
| `mocks: {}` | `global.mocks: {}` | Nested in global object |
| `stubs: {}` | `global.stubs: {}` | Nested in global object |
| `wrapper.destroy()` | `wrapper.unmount()` | Renamed method |
| `scopedSlots` | `slots` | Merged into slots option |
| `findAll().at(index)` | `findAll()[index]` | Returns standard array |

### mount() vs shallowMount() decision matrix

| Use Case | Method | Rationale |
|----------|--------|-----------|
| Integration tests | `mount()` | Tests real component interactions |
| Unit tests (isolated) | `shallowMount()` | Stubs children, faster execution |
| Testing slot content | `mount()` | shallowMount doesn't render slots by default |
| Heavy child dependencies | `mount()` + selective stubs | More control over what renders |

**Key v2 behavior change:** `shallowMount` no longer renders default slot content of stubbed components. Enable this with:

```typescript
import { config } from '@vue/test-utils'
config.global.renderStubDefaultSlot = true
```

---

## Happy-dom configuration and limitations

Happy-dom is **2-10x faster** than jsdom for most DOM operations and is the default for `@nuxt/test-utils`. However, it has important limitations that require workarounds.

### Known limitations and workarounds

| API/Feature | Status | Workaround |
|-------------|--------|------------|
| `getBoundingClientRect()` | Returns zeros | Mock the function |
| `getComputedStyle()` | Limited CSS support | Use jsdom for CSS tests |
| Vitest timer methods | Partial support | Import Happy-DOM timers |
| vitest-axe | Not compatible | Must use jsdom |

### Required setup file for happy-dom

```typescript
// test/setup.ts
import { vi } from 'vitest'
import { PropertySymbol } from 'happy-dom'

// Timer workaround for happy-dom + Vitest
const browserWindow =
  global.document?.[PropertySymbol.ownerWindow] ||
  global.document?.[PropertySymbol.window]

if (browserWindow) {
  global.setTimeout = browserWindow.setTimeout
  global.clearTimeout = browserWindow.clearTimeout
  global.setInterval = browserWindow.setInterval
  global.clearInterval = browserWindow.clearInterval
  global.requestAnimationFrame = browserWindow.requestAnimationFrame
  global.cancelAnimationFrame = browserWindow.cancelAnimationFrame
}

// getBoundingClientRect mock
Element.prototype.getBoundingClientRect = vi.fn(() => ({
  width: 120, height: 120,
  top: 0, left: 0, bottom: 120, right: 120,
  x: 0, y: 0,
  toJSON: () => {},
}))

// ResizeObserver mock
global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}))

// matchMedia mock
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
})
```

### When to choose jsdom over happy-dom

- **Accessibility testing** with vitest-axe (required)
- Tests depending on **CSS layout calculations**
- Testing **canvas operations**
- Need accurate `getComputedStyle()` results

Switch per-file using the comment directive:
```typescript
// @vitest-environment jsdom
```

---

## Nuxt 4 specific testing patterns

Nuxt 4 introduces critical changes to testing: `undefined` replaces `null` for data/error defaults, `shallowRef` becomes the default for fetched data, and the new `app/` directory structure.

### Mocking auto-imported composables with mockNuxtImport

```typescript
import { vi } from 'vitest'
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'

// useState mock
mockNuxtImport('useState', () => {
  return (key: string, init?: () => any) => ref(init ? init() : null)
})

// useRoute mock
mockNuxtImport('useRoute', () => {
  return () => ({
    params: { id: '999' },
    path: '/test/999',
    query: { filter: 'active' },
    fullPath: '/test/999?filter=active',
    name: 'test-id',
  })
})

// Dynamic mock with vi.hoisted (for per-test modifications)
const { useRouterMock } = vi.hoisted(() => ({
  useRouterMock: vi.fn(() => ({
    push: vi.fn(),
    replace: vi.fn(),
    currentRoute: ref({ path: '/' }),
  })),
}))

mockNuxtImport('useRouter', () => useRouterMock)
```

### useAsyncData and useFetch mocking for SSG

```typescript
import { vi } from 'vitest'
import { mockNuxtImport, registerEndpoint } from '@nuxt/test-utils/runtime'

// Option 1: Mock the composable directly
const { mockFetch } = vi.hoisted(() => ({
  mockFetch: vi.fn(),
}))

mockNuxtImport('useFetch', () => mockFetch)
mockNuxtImport('useAsyncData', () => mockFetch)

describe('SSG Data Fetching', () => {
  beforeEach(() => {
    mockFetch.mockReturnValue({
      data: shallowRef([{ id: 1, title: 'Cached Article' }]), // Nuxt 4 uses shallowRef
      pending: ref(false),
      error: ref(undefined),  // Nuxt 4: undefined, not null
      status: ref('success'),
      refresh: vi.fn(),
      execute: vi.fn(),
    })
  })

  it('displays cached data', async () => {
    const component = await mountSuspended(ArticleList)
    expect(component.text()).toContain('Cached Article')
  })
})

// Option 2: Use registerEndpoint for API mocking
registerEndpoint('/api/articles', () => ([
  { id: 1, title: 'Article 1' },
  { id: 2, title: 'Article 2' },
]))

registerEndpoint('/api/articles', {
  method: 'POST',
  handler: () => ({ id: 3, title: 'New Article' }),
})
```

### Testing with mountSuspended (recommended for Nuxt)

```typescript
import { mountSuspended } from '@nuxt/test-utils/runtime'
import MyComponent from '~/components/MyComponent.vue'

describe('MyComponent', () => {
  it('renders with Nuxt context', async () => {
    // mountSuspended handles Suspense automatically
    const wrapper = await mountSuspended(MyComponent, {
      props: { title: 'Test' }
    })
    
    expect(wrapper.text()).toContain('Test')
  })

  it('renders at specific route', async () => {
    const wrapper = await mountSuspended(App, {
      route: '/about'
    })
    expect(wrapper.text()).toContain('About')
  })
})
```

### Nuxt 3 vs Nuxt 4 breaking changes for tests

| Aspect | Nuxt 3 | Nuxt 4 |
|--------|--------|--------|
| data/error defaults | `null` | `undefined` |
| Data reactivity | Deep ref | `shallowRef` |
| pending with `immediate: false` | `true` initially | `false` initially |
| Component names | Filename only | Full path-based |
| Directory structure | Root-based | `app/` directory |

**Migration example:**
```typescript
// Nuxt 3
expect(error.value).toBe(null)

// Nuxt 4
expect(error.value).toBe(undefined)
```

---

## Testing Pinia stores

The `@pinia/testing` package provides `createTestingPinia` for isolated store testing.

```typescript
import { mount } from '@vue/test-utils'
import { createTestingPinia } from '@pinia/testing'
import { vi } from 'vitest'
import Counter from './Counter.vue'
import { useCounterStore } from '@/stores/counter'

describe('Counter with Pinia', () => {
  it('displays count from store', () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [
          createTestingPinia({
            createSpy: vi.fn,
            initialState: {
              counter: { count: 10 }
            }
          })
        ]
      }
    })

    expect(wrapper.text()).toContain('10')
  })

  it('calls store action on button click', async () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [createTestingPinia({ createSpy: vi.fn })]
      }
    })

    const store = useCounterStore()
    await wrapper.find('button').trigger('click')

    // Actions are stubbed by default
    expect(store.increment).toHaveBeenCalledTimes(1)
  })

  it('executes real actions with stubActions: false', async () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [
          createTestingPinia({
            createSpy: vi.fn,
            stubActions: false  // Real actions execute
          })
        ]
      }
    })

    const store = useCounterStore()
    await wrapper.find('button').trigger('click')
    expect(store.count).toBe(1)
  })
})
```

---

## Testing shadcn-vue and Reka UI components

shadcn-vue uses Reka UI (formerly Radix Vue) primitives, which require browser API shims and careful portal handling.

### Required browser API shims

```typescript
// test/setup.ts
import { vi } from 'vitest'

// Critical for Reka UI components
window.PointerEvent = MouseEvent as typeof PointerEvent

global.ResizeObserver = class ResizeObserver {
  observe() {}
  unobserve() {}
  disconnect() {}
}

global.DOMRect = class DOMRect {
  x = 0; y = 0; width = 0; height = 0
  top = 0; right = 0; bottom = 0; left = 0
  toJSON() { return this }
}

Element.prototype.scrollIntoView = vi.fn()
Element.prototype.hasPointerCapture = vi.fn(() => false)
Element.prototype.releasePointerCapture = vi.fn()
Element.prototype.setPointerCapture = vi.fn()
```

### Teleport/Portal handling strategies

```typescript
// Strategy 1: Create portal target before each test
beforeEach(() => {
  const el = document.createElement('div')
  el.id = 'radix-portal'
  document.body.appendChild(el)
})

afterEach(() => {
  document.body.innerHTML = ''
})

// Strategy 2: Stub Teleport globally
const wrapper = mount(Dialog, {
  props: { open: true },
  global: {
    stubs: {
      teleport: true
    }
  }
})
```

### Dialog component test example

```typescript
import { render, screen } from '@testing-library/vue'
import userEvent from '@testing-library/user-event'
import { Dialog, DialogContent, DialogTrigger, DialogTitle } from '@/components/ui/dialog'

describe('Dialog Component', () => {
  beforeEach(() => {
    const portal = document.createElement('div')
    portal.id = 'radix-portal'
    document.body.appendChild(portal)
  })

  afterEach(() => {
    document.body.innerHTML = ''
  })

  it('opens dialog on trigger click', async () => {
    const user = userEvent.setup()
    
    render({
      components: { Dialog, DialogContent, DialogTrigger, DialogTitle },
      template: `
        <Dialog>
          <DialogTrigger>Open</DialogTrigger>
          <DialogContent>
            <DialogTitle>Test Dialog</DialogTitle>
            <p>Dialog content</p>
          </DialogContent>
        </Dialog>
      `
    })

    await user.click(screen.getByText('Open'))
    
    expect(screen.getByRole('dialog')).toBeInTheDocument()
    expect(screen.getByText('Test Dialog')).toBeInTheDocument()
  })

  it('closes dialog on escape key', async () => {
    const user = userEvent.setup()
    
    render({
      components: { Dialog, DialogContent, DialogTrigger },
      template: `
        <Dialog>
          <DialogTrigger>Open</DialogTrigger>
          <DialogContent>Content</DialogContent>
        </Dialog>
      `
    })

    await user.click(screen.getByText('Open'))
    await user.keyboard('{Escape}')
    
    expect(screen.queryByRole('dialog')).not.toBeInTheDocument()
  })
})
```

### Accessibility testing with vitest-axe

**Important:** vitest-axe requires `jsdom`, not `happy-dom`.

```typescript
// @vitest-environment jsdom
import { render } from '@testing-library/vue'
import { axe, toHaveNoViolations } from 'vitest-axe'
import { expect } from 'vitest'

expect.extend(toHaveNoViolations)

describe('Dialog Accessibility', () => {
  it('has no accessibility violations', async () => {
    const { container } = render(Dialog, {
      props: { open: true },
      slots: { default: '<div>Dialog content</div>' }
    })
    
    const results = await axe(container)
    expect(results).toHaveNoViolations()
  })
})
```

---

## Snapshot testing strategies

Snapshots work best for **small, focused outputs** like error messages and code transformations—not large component trees.

### When to use snapshot vs assertion-based testing

| Use Snapshots For | Use Assertions For |
|-------------------|--------------------|
| Error messages, warnings | User interactions, behavior |
| Code transformations | Dynamic content (timestamps, IDs) |
| Small component structures | Complex conditional logic |
| Serialized data structures | Accessibility attributes |
| CSS-in-JS output | Critical user flows |

### Inline vs file snapshots

```typescript
// Inline snapshots (preferred for <10 lines)
it('formats currency correctly', () => {
  const result = formatCurrency(1234.56, 'USD')
  expect(result).toMatchInlineSnapshot(`"$1,234.56"`)
})

// File snapshots (for larger outputs)
it('renders complete card component', () => {
  const wrapper = mount(Card, {
    props: { title: 'Hello', content: 'World' }
  })
  expect(wrapper.html()).toMatchSnapshot()
})
```

### Avoiding brittle snapshots with dynamic values

```typescript
// Mock time for consistent snapshots
beforeEach(() => {
  vi.useFakeTimers()
  vi.setSystemTime(new Date('2025-01-01T12:00:00Z'))
})

afterEach(() => {
  vi.useRealTimers()
})

// Use property matchers for partial snapshots
it('creates user with generated fields', () => {
  const user = createUser({ name: 'John' })
  
  expect(user).toMatchSnapshot({
    id: expect.any(String),
    createdAt: expect.any(Date),
    updatedAt: expect.any(Date)
  })
})

// Custom serializer for dynamic IDs
expect.addSnapshotSerializer({
  test: (val) => val && typeof val === 'object' && val.id,
  serialize: (val, config, indent, depth, refs, printer) => {
    return printer({ ...val, id: '[GENERATED_ID]' }, config, indent, depth, refs)
  }
})
```

### Snapshot anti-patterns to avoid

- **Giant snapshots** (200+ lines) - Too noisy to review meaningfully
- **Snapshot everything** - `expect(2 + 2).toMatchSnapshot()` adds no value
- **Blind updates** - Always review before running `vitest -u`
- **Testing library internals** - Don't snapshot Reka UI internal structures

---

## Coverage configuration and thresholds

Vitest's **v8 provider** is now recommended—since v3.2.0, it uses AST-based remapping that matches Istanbul accuracy with significantly better performance.

### Complete coverage configuration

```typescript
// vitest.config.ts
export default defineVitestConfig({
  test: {
    coverage: {
      provider: 'v8',
      enabled: true,
      reporter: ['text', 'html', 'lcov', 'json-summary'],
      reportsDirectory: './coverage',
      
      // Include patterns for Nuxt 4 app/ directory
      include: [
        'app/components/**/*.{vue,ts}',
        'app/composables/**/*.ts',
        'app/utils/**/*.ts',
        'app/stores/**/*.ts',
        'app/pages/**/*.vue',
        'server/**/*.ts',
      ],
      
      // Exclude patterns
      exclude: [
        'node_modules/**',
        '.nuxt/**',
        '.output/**',
        '**/*.config.{js,ts,mjs}',
        '**/*.{test,spec}.{ts,vue}',
        '**/*.d.ts',
        '**/fixtures/**',
      ],
      
      // Tiered thresholds by component type
      thresholds: {
        lines: 75,
        functions: 70,
        branches: 70,
        statements: 75,
        perFile: true,
        
        'app/composables/**/*.ts': {
          lines: 90, functions: 90, branches: 85, statements: 90,
        },
        'app/utils/**/*.ts': {
          lines: 90, functions: 90, branches: 85, statements: 90,
        },
        'app/stores/**/*.ts': {
          lines: 85, functions: 85, branches: 80, statements: 85,
        },
        'app/components/**/*.vue': {
          lines: 70, functions: 70, branches: 65, statements: 70,
        },
        'app/pages/**/*.vue': {
          lines: 60, functions: 60, branches: 55, statements: 60,
        },
      },
    },
  },
})
```

### Recommended thresholds by component type

| Component Type | Lines | Functions | Branches | Rationale |
|----------------|-------|-----------|----------|-----------|
| Composables | 90% | 90% | 85% | Pure logic, high reuse |
| Utils/Helpers | 90% | 90% | 85% | Isolated, critical functions |
| Pinia Stores | 85% | 85% | 80% | State logic is testable |
| Container Components | 80% | 80% | 75% | Business logic integration |
| UI Components | 70% | 70% | 65% | Focus on behavior, not lines |
| Pages | 60% | 60% | 55% | Often better covered by E2E |

---

## Proper cleanup and isolation

### Using enableAutoUnmount (recommended)

```typescript
// test/setup.ts
import { enableAutoUnmount, config } from '@vue/test-utils'
import { afterEach, vi } from 'vitest'

// Automatically unmount components after each test
enableAutoUnmount(afterEach)

// Clean up mocks
afterEach(() => {
  vi.restoreAllMocks()
  vi.clearAllTimers()
})

// Reset global config if modified in tests
afterEach(() => {
  config.global.renderStubDefaultSlot = false
})
```

### Vitest configuration for isolation

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    clearMocks: true,
    restoreMocks: true,
    setupFiles: ['./test/setup.ts'],
  }
})
```

---

## Critical anti-patterns to avoid

### Don't test implementation details

```typescript
// ❌ BAD: Testing internal state
expect(wrapper.vm.internalCounter).toBe(5)

// ✅ GOOD: Test observable behavior
expect(wrapper.find('[data-testid="count"]').text()).toBe('5')
```

### Don't use setData with Composition API

```typescript
// ❌ BAD: setData doesn't work with script setup
await wrapper.setData({ count: 5 })

// ✅ GOOD: Trigger the action that changes state
await wrapper.find('button').trigger('click')
```

### Don't forget to await async operations

```typescript
// ❌ BAD: Missing await
wrapper.find('button').trigger('click')
expect(wrapper.text()).toContain('Updated')  // May fail!

// ✅ GOOD: Await the trigger
await wrapper.find('button').trigger('click')
expect(wrapper.text()).toContain('Updated')
```

### Don't over-stub

```typescript
// ❌ BAD: Stubbing everything
const wrapper = shallowMount(Component)

// ✅ GOOD: Use mount, stub only problematic dependencies
const wrapper = mount(Component, {
  global: {
    stubs: {
      HeavyExternalComponent: true  // Only stub what's necessary
    }
  }
})
```

---

## Conclusion

Building a robust testing strategy for Nuxt 4 requires understanding the ecosystem's nuances: happy-dom for speed, jsdom for accessibility, `mountSuspended` for Nuxt context, and `mockNuxtImport` for auto-imports. The key takeaways:

- **Use Vitest projects** to separate unit tests (Node) from component tests (Nuxt environment)
- **happy-dom is 2-10x faster** but requires browser API shims; use jsdom for vitest-axe
- **Nuxt 4 defaults changed**: `undefined` replaces `null`, `shallowRef` replaces deep refs
- **Test behavior, not implementation**: focus on user interactions and observable output
- **Set tiered coverage thresholds**: 90% for composables/utils, 60-70% for pages/UI components
- **Snapshots are tools, not tests**: use them for small outputs, prefer assertions for behavior

The shadcn-vue/Reka UI testing patterns require particular attention to PointerEvent shims and Teleport handling, but once configured, they enable comprehensive accessibility and interaction testing that ensures your components work correctly across keyboard and screen reader users.