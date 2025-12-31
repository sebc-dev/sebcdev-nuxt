# Breaking Changes Nuxt 3 → Nuxt 4

| Aspect | Nuxt 3 (ancien) | Nuxt 4 (actuel) |
|--------|-----------------|-----------------|
| **Structure** | `pages/`, `components/` | `app/pages/`, `app/components/` |
| **Generate config** | `generate: {}` | `nitro: { prerender: {} }` |
| **Output** | `dist/` | `.output/public/` |
| **Exclude routes** | `generate.exclude` | `nitro.prerender.ignore` |
| **TypeScript** | Single tsconfig | Project references |
| **Compatibility** | N/A | `compatibilityDate` requis |
| **SSR Styles** | `experimental.inlineSSRStyles` | `features.inlineStyles` |
| **SEO Server-only** | `useServerSeoMeta()` | `if (import.meta.server) { useSeoMeta() }` |

## Migration useServerSeoMeta() (SEO)

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

**Cas d'usage** : Meta tags statiques qui n'ont pas besoin d'être recalculés côté client (robots, ogSiteName, twitterCard par défaut). Voir `seo-patterns.md` pour les patterns complets.

**Exemples migration generate → nitro.prerender :**

```typescript
// ❌ Nuxt 3 (obsolète dans Nuxt 4)
export default defineNuxtConfig({
  generate: {
    exclude: ['/admin'],
    routes: ['/sitemap.xml']
  }
})

// ✅ Nuxt 4 (correct)
export default defineNuxtConfig({
  nitro: {
    prerender: {
      ignore: ['/admin'],        // Remplace generate.exclude
      routes: ['/sitemap.xml']   // Remplace generate.routes
    }
  }
})
```
