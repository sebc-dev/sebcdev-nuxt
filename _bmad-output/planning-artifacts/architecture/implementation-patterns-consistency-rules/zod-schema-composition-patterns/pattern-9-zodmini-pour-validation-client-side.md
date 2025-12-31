# Pattern 9 : zod/mini pour validation client-side

`zod/mini` (1.88kb gzip) est idéal pour la validation côté client quand la taille du bundle est critique.

## Différences avec Zod 4 complet

```typescript
import * as z from 'zod/mini'

// API fonctionnelle avec .check() au lieu du chaining
const contactSchema = z.object({
  email: z.string().check(z.email()),
  message: z.string().check(z.minLength(10), z.maxLength(500))
})

// Wrappers fonctionnels au lieu de méthodes
const optionalEmail = z.optional(z.email())
```

## ⚠️ Caveat critique : Pas de locale par défaut

```typescript
import * as z from 'zod/mini'

// SANS configuration : TOUS les messages = "Invalid input"
z.string().min(5).safeParse('ab')  // "Invalid input"

// AVEC configuration : Messages lisibles
z.config(z.locales.en())  // ou z.locales.fr()
z.string().min(5).safeParse('ab')  // "String must contain at least 5 character(s)"
```

## Recommandation pour le projet

| Contexte | Package | Raison |
|----------|---------|--------|
| Nuxt Content schemas | `zod/v4` (complet) | Build-time, pas dans le bundle client |
| Validation formulaires client | `zod/mini` | 1.88kb vs 5.36kb |
| Server routes API | `zod/v4` (complet) | Pas de contrainte bundle |

---
