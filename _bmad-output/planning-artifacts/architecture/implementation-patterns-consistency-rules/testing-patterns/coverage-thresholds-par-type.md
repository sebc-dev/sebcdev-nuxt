# Coverage Thresholds par Type

Configuration tiered basée sur la testabilité de chaque type de code :

```typescript
// vitest.config.ts
coverage: {
  provider: 'v8',
  include: [
    'app/components/**/*.{vue,ts}',
    'app/composables/**/*.ts',
    'app/utils/**/*.ts',
    'app/stores/**/*.ts',
    'app/pages/**/*.vue',
  ],

  thresholds: {
    // Seuils globaux
    lines: 75,
    functions: 70,
    branches: 70,
    statements: 75,
    perFile: true,

    // Seuils par type (priorité haute)
    'app/composables/**/*.ts': {
      lines: 90, functions: 90, branches: 85, statements: 90,
    },
    'app/utils/**/*.ts': {
      lines: 90, functions: 90, branches: 85, statements: 90,
    },

    // Seuils par type (priorité moyenne)
    'app/stores/**/*.ts': {
      lines: 85, functions: 85, branches: 80, statements: 85,
    },
    'app/components/**/*.vue': {
      lines: 70, functions: 70, branches: 65, statements: 70,
    },

    // Seuils par type (priorité basse - E2E préférable)
    'app/pages/**/*.vue': {
      lines: 60, functions: 60, branches: 55, statements: 60,
    },
  },
}
```

| Type | Target | Rationale |
|------|--------|-----------|
| Composables | 90% | Logique pure, haute réutilisation |
| Utils | 90% | Fonctions isolées, critiques |
| Stores | 85% | State management testable |
| Components | 70% | Focus sur comportement utilisateur |
| Pages | 60% | Mieux couvertes par E2E |

---
