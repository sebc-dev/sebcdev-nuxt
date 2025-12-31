# Installation

```bash
pnpm add -D @nuxt/test-utils vitest @vue/test-utils happy-dom @vitest/coverage-v8 playwright-core
```

Ajouter le module dans `nuxt.config.ts` :

```typescript
export default defineNuxtConfig({
  modules: ['@nuxt/test-utils/module'],
})
```

---
