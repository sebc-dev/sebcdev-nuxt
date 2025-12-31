# Configuration Vitest pour Nuxt 4

**Fichier `vitest.config.ts` à la racine du projet :**

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import { defineVitestProject } from '@nuxt/test-utils/config'

export default defineConfig({
  test: {
    projects: [
      // Tests unitaires rapides - environnement Node
      {
        test: {
          name: 'unit',
          include: ['test/unit/**/*.{test,spec}.ts'],
          environment: 'node',
          globals: true,
        },
      },
      // Tests composants Nuxt - environnement complet
      await defineVitestProject({
        test: {
          name: 'nuxt',
          include: ['test/nuxt/**/*.{test,spec}.ts'],
          environment: 'nuxt',
          environmentOptions: {
            nuxt: {
              domEnvironment: 'happy-dom',  // Plus rapide que jsdom
              mock: {
                intersectionObserver: true,
                indexedDb: true,
              },
            },
          },
        },
      }),
    ],
  },
})
```

**Prérequis :** `"type": "module"` dans `package.json` ou renommer en `vitest.config.mts`.

**Scripts npm recommandés :**

```json
{
  "scripts": {
    "test": "vitest",
    "test:unit": "vitest --project unit",
    "test:nuxt": "vitest --project nuxt",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
}
```

---
