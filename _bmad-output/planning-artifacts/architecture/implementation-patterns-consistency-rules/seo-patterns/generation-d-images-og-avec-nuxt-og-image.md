# Génération d'images OG avec nuxt-og-image

## Configuration Zero Runtime (SSG)

Le mode `zeroRuntime: true` génère les images au build time via Satori, sans server functions. **100% compatible Cloudflare Pages gratuit**.

```typescript
// nuxt.config.ts
ogImage: {
  zeroRuntime: true,           // ESSENTIEL pour SSG pur
  runtimeCacheStorage: false,  // Pas de cache runtime en SSG
  defaults: {
    renderer: 'satori',        // Vue → SVG → PNG au build
    width: 1200,
    height: 630,               // Ratio 1.91:1 standard
  }
}
```

## Composant template OG Image

```vue
<!-- app/components/OgImage/BlogPost.vue -->
<script setup lang="ts">
withDefaults(defineProps<{
  title?: string
  description?: string
  siteName?: string
}>(), {
  title: 'Article',
  siteName: 'sebc.dev',
})
</script>

<template>
  <div class="h-full w-full flex flex-col justify-between p-16 bg-slate-800">
    <div class="flex flex-col gap-4">
      <h1 class="text-white text-6xl font-bold leading-tight">{{ title }}</h1>
      <p v-if="description" class="text-gray-200 text-2xl">{{ description }}</p>
    </div>
    <span class="text-white text-xl font-semibold">{{ siteName }}</span>
  </div>
</template>
```

## Utilisation avec defineOgImageComponent()

```typescript
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
// ... fetch article data ...

// Associer le template OG à cette page
defineOgImageComponent('BlogPost', {
  title: article.value?.title,
  description: article.value?.description,
})
</script>
```

## Performance build

| Métrique | Valeur |
|----------|--------|
| Temps par image | ~50-100ms |
| 100 articles | ~10 secondes |
| Output | `.output/public/__og-image__/` |
| Cache CDN | Illimité (assets statiques) |
