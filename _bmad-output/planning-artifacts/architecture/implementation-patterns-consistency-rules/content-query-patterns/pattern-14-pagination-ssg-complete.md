# Pattern 14 : Pagination SSG complète

Pattern combinant `.skip()`, `.limit()` et `.count()` pour une pagination fonctionnelle en SSG.

## Composable de pagination

```typescript
// composables/usePaginatedContent.ts
import type { Collections } from '@nuxt/content'

interface PaginationOptions {
  perPage?: number
}

export function usePaginatedContent(options: PaginationOptions = {}) {
  const { locale } = useI18n()
  const route = useRoute()
  const perPage = options.perPage ?? 10

  const currentPage = computed(() =>
    parseInt(route.query.page as string) || 1
  )

  const collection = computed(() =>
    `articles_${locale.value}` as keyof Collections
  )

  // Requête paginée
  const { data: items, pending } = useAsyncData(
    `paginated-${locale.value}-${currentPage.value}`,
    () => queryCollection(collection.value)
      .where('draft', '=', false)
      .order('publishedAt', 'DESC')
      .skip((currentPage.value - 1) * perPage)
      .limit(perPage)
      .all(),
    { watch: [locale, currentPage] }
  )

  // Compte total (clé stable pour cache)
  const { data: totalCount } = useAsyncData(
    `count-${locale.value}`,
    () => queryCollection(collection.value)
      .where('draft', '=', false)
      .count(),
    { watch: [locale] }
  )

  const totalPages = computed(() =>
    Math.ceil((totalCount.value ?? 0) / perPage)
  )

  const hasNextPage = computed(() =>
    currentPage.value < totalPages.value
  )

  const hasPrevPage = computed(() =>
    currentPage.value > 1
  )

  return {
    items,
    pending,
    currentPage,
    totalPages,
    totalCount,
    hasNextPage,
    hasPrevPage,
    perPage
  }
}
```

## Composant Pagination

```vue
<!-- components/content/Pagination.vue -->
<script setup lang="ts">
interface Props {
  currentPage: number
  totalPages: number
  baseUrl?: string
}

const props = withDefaults(defineProps<Props>(), {
  baseUrl: ''
})

const pageNumbers = computed(() => {
  const pages: (number | '...')[] = []
  const current = props.currentPage
  const total = props.totalPages

  if (total <= 7) {
    return Array.from({ length: total }, (_, i) => i + 1)
  }

  pages.push(1)
  if (current > 3) pages.push('...')
  for (let i = Math.max(2, current - 1); i <= Math.min(total - 1, current + 1); i++) {
    pages.push(i)
  }
  if (current < total - 2) pages.push('...')
  pages.push(total)

  return pages
})
</script>

<template>
  <nav class="flex justify-center gap-2" aria-label="Pagination">
    <NuxtLink
      v-if="currentPage > 1"
      :to="`${baseUrl}?page=${currentPage - 1}`"
      class="px-3 py-2 rounded hover:bg-muted"
    >
      ←
    </NuxtLink>

    <template v-for="page in pageNumbers" :key="page">
      <span v-if="page === '...'" class="px-3 py-2">...</span>
      <NuxtLink
        v-else
        :to="`${baseUrl}?page=${page}`"
        class="px-3 py-2 rounded"
        :class="page === currentPage ? 'bg-primary text-primary-foreground' : 'hover:bg-muted'"
      >
        {{ page }}
      </NuxtLink>
    </template>

    <NuxtLink
      v-if="currentPage < totalPages"
      :to="`${baseUrl}?page=${currentPage + 1}`"
      class="px-3 py-2 rounded hover:bg-muted"
    >
      →
    </NuxtLink>
  </nav>
</template>
```

## Configuration prerender pour pagination

```typescript
// nuxt.config.ts
nitro: {
  prerender: {
    crawlLinks: true,
    // Expliciter les pages paginées si non découvertes automatiquement
    routes: [
      '/blog',
      '/blog?page=1',
      '/blog?page=2',
      '/blog?page=3'
    ]
  }
}
```
