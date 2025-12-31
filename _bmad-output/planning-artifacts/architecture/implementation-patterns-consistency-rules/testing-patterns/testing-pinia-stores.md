# Testing Pinia Stores

**Installation :**

```bash
pnpm add -D @pinia/testing
```

## Pattern avec createTestingPinia

```typescript
import { mount } from '@vue/test-utils'
import { createTestingPinia } from '@pinia/testing'
import { vi } from 'vitest'
import Counter from './Counter.vue'
import { useCounterStore } from '~/stores/counter'

describe('Counter avec Pinia', () => {
  it('affiche le count depuis le store', () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [
          createTestingPinia({
            createSpy: vi.fn,
            initialState: {
              counter: { count: 10 },
            },
          }),
        ],
      },
    })

    expect(wrapper.text()).toContain('10')
  })

  it('appelle l\'action store au clic', async () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [createTestingPinia({ createSpy: vi.fn })],
      },
    })

    const store = useCounterStore()
    await wrapper.find('button').trigger('click')

    // Actions stubbées par défaut
    expect(store.increment).toHaveBeenCalledTimes(1)
  })

  it('exécute les vraies actions avec stubActions: false', async () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [
          createTestingPinia({
            createSpy: vi.fn,
            stubActions: false,  // Actions réelles
          }),
        ],
      },
    })

    const store = useCounterStore()
    await wrapper.find('button').trigger('click')
    expect(store.count).toBe(1)
  })
})
```

---
