# Plugin TypeScript Declarations

**Augmentation de types pour plugins :**

```typescript
// types/plugins.d.ts
declare module '#app' {
  interface NuxtApp {
    $formatDate: (date: string | Date, locale?: string) => string
    $readingTime: (content: string, wordsPerMinute?: number) => number
    $excerpt: (content: string, length?: number) => string
    $slugify: (text: string) => string
  }
}
```

**Augmentation runtimeConfig :**

```typescript
// types/runtime-config.d.ts
declare module 'nuxt/schema' {
  interface PublicRuntimeConfig {
    apiBase: string
    siteUrl: string
    gaTrackingId?: string
  }

  interface RuntimeConfig {
    // Secrets serveur uniquement
    contentfulToken?: string
  }
}
```

Ces déclarations permettent l'autocomplétion pour `useNuxtApp().$formatDate` et `useRuntimeConfig().public.siteUrl`.
