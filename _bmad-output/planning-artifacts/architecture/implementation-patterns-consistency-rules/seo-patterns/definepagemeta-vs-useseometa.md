# definePageMeta() vs useSeoMeta()

`definePageMeta()` est une **macro build-time** qui ne peut pas contenir de valeurs dynamiques.

```typescript
// ✅ BON : definePageMeta pour métadonnées statiques de routing
definePageMeta({
  layout: 'blog',
  middleware: 'auth'
})

// ✅ BON : useSeoMeta pour SEO (supporte réactivité)
const title = ref('Mon titre dynamique')
useSeoMeta({
  title: () => title.value,
  description: () => `Description de ${title.value}`
})

// ❌ MAUVAIS : definePageMeta avec valeurs dynamiques
definePageMeta({
  title: someVariable  // Ne fonctionnera pas - extrait au build
})
```
