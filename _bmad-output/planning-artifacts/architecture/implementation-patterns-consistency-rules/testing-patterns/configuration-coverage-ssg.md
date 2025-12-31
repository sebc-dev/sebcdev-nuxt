# Configuration Coverage SSG

Pour les projets SSG, **définir explicitement `include`** pour éviter l'inflation due aux auto-imports :

```typescript
// vitest.config.ts
export default defineVitestConfig({
  test: {
    environment: 'nuxt',
    coverage: {
      provider: 'v8',
      enabled: true,

      // Inclusion explicite (évite inflation auto-imports)
      include: [
        'app/components/**/*.vue',
        'app/composables/**/*.ts',
        'app/utils/**/*.ts',
      ],

      exclude: [
        '**/.nuxt/**',
        '**/.output/**',
        '**/node_modules/**',
        '**/*.d.ts',
        'test/**',
      ],

      reporter: ['text', 'html', 'lcov'],
      reportsDirectory: './coverage',

      thresholds: {
        lines: 80,
        functions: 80,
        branches: 75,
        statements: 80,
      },
    },
  },
})
```

---
