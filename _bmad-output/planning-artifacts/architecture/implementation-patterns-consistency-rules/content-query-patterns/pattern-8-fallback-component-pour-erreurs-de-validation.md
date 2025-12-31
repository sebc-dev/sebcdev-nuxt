# Pattern 8 : Fallback component pour erreurs de validation

Gérer gracieusement les contenus avec erreurs de validation côté composant :

## Composant FallbackContent

```vue
<!-- components/content/FallbackContent.vue -->
<script setup lang="ts">
interface Props {
  message?: string
  showHomeLink?: boolean
}

withDefaults(defineProps<Props>(), {
  message: 'Ce contenu n\'est pas disponible.',
  showHomeLink: true
})

const localePath = useLocalePath()
</script>

<template>
  <div class="flex flex-col items-center justify-center py-16 text-center">
    <div class="rounded-lg bg-muted p-8 max-w-md">
      <Icon name="lucide:file-warning" class="w-12 h-12 text-muted-foreground mb-4" />
      <p class="text-muted-foreground mb-4">{{ message }}</p>
      <NuxtLink
        v-if="showHomeLink"
        :to="localePath('/')"
        class="text-primary hover:underline"
      >
        Retour à l'accueil
      </NuxtLink>
    </div>
  </div>
</template>
```

## Usage dans les pages

```vue
<!-- pages/blog/[...slug].vue -->
<script setup lang="ts">
import type { Collections } from '@nuxt/content'

const route = useRoute()
const { locale } = useI18n()

const { data: article } = await useAsyncData(
  `article-${locale.value}-${route.path}`,
  async () => {
    const collection = `articles_${locale.value}` as keyof Collections
    return queryCollection(collection).path(route.path).first()
  }
)

// 404 si article null (validation échouée ou inexistant)
if (!article.value) {
  throw createError({
    statusCode: 404,
    statusMessage: 'Article non trouvé'
  })
}
</script>

<template>
  <article v-if="article">
    <ContentRenderer :value="article" />
  </article>
  <FallbackContent v-else message="Cet article n'a pas pu être chargé." />
</template>
```

## Pattern avec état de chargement

```vue
<script setup lang="ts">
const { data: article, pending, error } = await useAsyncData(...)
</script>

<template>
  <!-- Loading state -->
  <div v-if="pending" class="animate-pulse">
    <div class="h-8 bg-muted rounded w-3/4 mb-4" />
    <div class="h-4 bg-muted rounded w-full mb-2" />
    <div class="h-4 bg-muted rounded w-5/6" />
  </div>

  <!-- Error state -->
  <FallbackContent
    v-else-if="error"
    message="Une erreur est survenue lors du chargement."
  />

  <!-- Empty state (validation failed) -->
  <FallbackContent
    v-else-if="!article"
    message="Ce contenu n'est pas disponible."
  />

  <!-- Success state -->
  <ContentRenderer v-else :value="article" />
</template>
```
