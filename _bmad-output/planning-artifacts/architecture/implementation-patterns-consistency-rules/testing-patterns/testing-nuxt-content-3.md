# Testing Nuxt Content 3

## Mock des données Content

```typescript
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'
import { ref } from 'vue'
import BlogPost from '~/pages/blog/[slug].vue'

mockNuxtImport('useAsyncData', () => {
  return () => ({
    data: ref({
      title: 'Article Test',
      description: 'Description test',
      body: {
        type: 'root',
        children: [
          { type: 'element', tag: 'p', children: [{ type: 'text', value: 'Contenu article' }] }
        ],
      },
    }),
    pending: ref(false),
    error: ref(null),
  })
})

it('rend la page article', async () => {
  const wrapper = await mountSuspended(BlogPost, {
    route: '/blog/article-test',
  })
  expect(wrapper.text()).toContain('Article Test')
})
```

## Stub ContentRenderer

```typescript
import { mockComponent, mountSuspended } from '@nuxt/test-utils/runtime'

mockComponent('ContentRenderer', {
  props: ['value'],
  template: '<div class="content-stub">{{ value?.title }}</div>',
})
```

---
