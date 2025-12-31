# Breaking Changes Nuxt 3 → Nuxt 4 (SEO)

## useServerSeoMeta() déprécié

```typescript
// ❌ ANCIEN (Nuxt 3) - Déprécié
useServerSeoMeta({
  description: 'Ma description'
})

// ✅ NOUVEAU (Nuxt 4) - Utiliser import.meta.server
if (import.meta.server) {
  useSeoMeta({
    description: 'Ma description'
  })
}
```

## Checklist migration SEO

| Ancien (Nuxt 3) | Nouveau (Nuxt 4) |
|-----------------|------------------|
| `useServerSeoMeta()` | `if (import.meta.server) { useSeoMeta() }` |
| `generate.routes` | `nitro.prerender.routes` |
| `generate.exclude` | `nitro.prerender.ignore` |
