# Testing Composables Nuxt

## useState

```typescript
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'
import { ref } from 'vue'

mockNuxtImport('useState', () => {
  return (key: string, init?: () => any) => ref(init ? init() : undefined)
})

it('initialise le state', async () => {
  const wrapper = await mountSuspended(CounterComponent)
  expect(wrapper.text()).toContain('Count: 0')
})
```

## useFetch / useAsyncData

```typescript
import { vi } from 'vitest'
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'
import { ref } from 'vue'

const { mockUseFetch } = vi.hoisted(() => ({
  mockUseFetch: vi.fn(),
}))

mockNuxtImport('useFetch', () => mockUseFetch)

afterEach(() => mockUseFetch.mockReset())

test('affiche état loading', async () => {
  mockUseFetch.mockReturnValue({
    data: ref(null),
    pending: ref(true),
    error: ref(null),
  })

  const wrapper = await mountSuspended(ArticleList)
  expect(wrapper.text()).toContain('Chargement...')
})

test('affiche les articles', async () => {
  mockUseFetch.mockReturnValue({
    data: ref([{ title: 'Guide Testing' }]),
    pending: ref(false),
    error: ref(null),
  })

  const wrapper = await mountSuspended(ArticleList)
  expect(wrapper.text()).toContain('Guide Testing')
})
```

---
