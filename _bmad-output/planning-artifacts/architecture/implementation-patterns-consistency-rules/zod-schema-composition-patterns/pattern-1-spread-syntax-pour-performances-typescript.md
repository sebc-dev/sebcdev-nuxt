# Pattern 1 : Spread syntax pour performances TypeScript

**Problème** : Chaîner plusieurs appels `.extend()` cause une complexité **O(n²)** dans les instantiations TypeScript, ralentissant significativement `tsc` et l'IDE.

## ❌ Anti-pattern : Chaining `.extend()`

```typescript
// MAUVAIS: Chaque .extend() multiplie les instantiations TypeScript
const TechArticle = BaseContent
  .extend(SEOFields.shape)
  .extend({ codeLanguage: z.string().optional() })
  .extend({ repository: z.string().url().optional() })
```

## ✅ Pattern recommandé : Object spread

```typescript
import { z } from 'zod/v4'

// Mixins réutilisables
const TimestampFields = {
  publishedAt: z.iso.date(),
  updatedAt: z.iso.date().optional()
}

const SEOFields = {
  description: z.string().max(160).optional(),
  ogImage: z.string().optional()
}

// Composition via spread = 100x moins d'instantiations TypeScript
const TechArticle = z.object({
  ...BaseContent.shape,
  ...SEOFields,
  ...TimestampFields,
  codeLanguage: z.string(),
  repository: z.string().url().optional()
})
```

## Quand utiliser quoi

| Méthode | Usage | Performance tsc |
|---------|-------|-----------------|
| `.extend({ ... })` | Extension simple, 1 niveau | ✅ OK |
| `z.object({ ...A.shape, ...B.shape })` | Composition multiple | ✅ Optimal |
| `.extend().extend().extend()` | Jamais | ❌ Quadratique |

---
