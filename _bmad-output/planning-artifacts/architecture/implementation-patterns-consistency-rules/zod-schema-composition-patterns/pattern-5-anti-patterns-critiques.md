# Pattern 5 : Anti-patterns critiques

## ❌ Utiliser `.merge()` (déprécié Zod 4)

```typescript
// ❌ DÉPRÉCIÉ: .merge() supprimé en Zod 4
const Combined = SchemaA.merge(SchemaB)

// ✅ CORRECT: .extend() avec .shape
const Combined = SchemaA.extend(SchemaB.shape)

// ✅ OPTIMAL: Spread syntax
const Combined = z.object({
  ...SchemaA.shape,
  ...SchemaB.shape
})
```

## ❌ Chaîner plusieurs `.extend()`

```typescript
// ❌ MAUVAIS: Performance TypeScript quadratique
const Final = Base
  .extend({ a: z.string() })
  .extend({ b: z.string() })
  .extend({ c: z.string() })

// ✅ CORRECT: Spread unique
const Final = z.object({
  ...Base.shape,
  a: z.string(),
  b: z.string(),
  c: z.string()
})
```

## ❌ Utiliser `z.intersection()` pour objets

```typescript
// ❌ MAUVAIS: ZodIntersection n'a pas les méthodes objet
const Combined = z.intersection(SchemaA, SchemaB)
Combined.pick({ field: true })  // ❌ Method doesn't exist
Combined.extend({ ... })        // ❌ Method doesn't exist

// ✅ CORRECT: Utiliser extend/spread
const Combined = z.object({
  ...SchemaA.shape,
  ...SchemaB.shape
})
Combined.pick({ field: true })  // ✅ Works
```

## ❌ Ignorer input vs output avec transforms

```typescript
const DateField = z.string().transform(val => new Date(val))

// ❌ MAUVAIS: z.infer donne le type OUTPUT uniquement
type DateType = z.infer<typeof DateField>  // Date (output)

// ✅ CORRECT: Tracker les deux types
type DateInput = z.input<typeof DateField>   // string
type DateOutput = z.output<typeof DateField> // Date
```

## ❌ Oublier que refinements sont perdus avec pick/omit

```typescript
// ❌ BUG SILENCIEUX: Le refinement disparaît
const Subset = SchemaWithRefine.pick({ field: true })

// ✅ CORRECT: Réappliquer après pick/omit
const Subset = SchemaWithRefine
  .pick({ field: true })
  .refine(/* réappliquer la validation */)
```

## ❌ Import depuis `@nuxt/content`

```typescript
// ❌ DÉPRÉCIÉ: Re-export obsolète
import { z } from '@nuxt/content'

// ✅ CORRECT: Import direct avec API Zod 4
import { z } from 'zod/v4'
```

## ❌ Recréer schemas dans les composants

```typescript
// ❌ MAUVAIS: 6 ops/ms - Schema recréé à chaque render
// Zod 4 utilise JIT compilation, le premier parse est plus lent
const MyComponent = () => {
  const schema = z.object({ name: z.string() })
  return schema.safeParse(data)
}

// ✅ CORRECT: 100+ ops/ms - JIT compilation réutilisée
const schema = z.object({ name: z.string() })  // Module-level
const MyComponent = () => schema.safeParse(data)
```

**Pourquoi :** Zod 4 compile le schema au premier `parse()`. Déplacer les schemas au niveau module permet de réutiliser cette compilation.

---
