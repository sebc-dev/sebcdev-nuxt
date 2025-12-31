# Title Tags

## titleTemplate dans app.vue

Le `titleTemplate` doit être défini dans `app.vue` pour supporter les fonctions dynamiques (impossible dans `nuxt.config.ts`).

```typescript
<!-- app/app.vue -->
<script setup lang="ts">
// Title template global avec fonction (contrôle total)
useHead({
  titleTemplate: (titleChunk) => {
    return titleChunk ? `${titleChunk} | sebc.dev` : 'sebc.dev'
  }
})

// Meta statiques server-only (performance SSG)
if (import.meta.server) {
  useSeoMeta({
    robots: 'index, follow',
    ogSiteName: 'sebc.dev',
    twitterCard: 'summary_large_image'
  })
}
</script>

<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>
```

## Longueur titre recommandée

| Aspect | Valeur | Raison |
|--------|--------|--------|
| **Caractères** | 50-60 max | Évite troncature SERPs |
| **Pixels** | ~600px | Limite affichage Google |
| **Format** | `{titre} \| {site}` | Pattern standard efficace |
