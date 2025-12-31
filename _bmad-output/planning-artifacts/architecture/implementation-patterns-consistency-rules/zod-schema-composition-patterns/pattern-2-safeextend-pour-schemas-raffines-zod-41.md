# Pattern 2 : `.safeExtend()` pour schemas raffinés (Zod 4.1+)

**Problème** : `.extend()` **lance une erreur** quand utilisé sur un schema avec `.refine()` ou `.superRefine()`.

## ❌ Anti-pattern : Extend sur schema raffiné

```typescript
const ValidatedContent = z.object({
  title: z.string(),
  publishDate: z.iso.date(),
  expiryDate: z.iso.date().optional()
}).refine(
  data => !data.expiryDate || data.expiryDate > data.publishDate,
  { error: 'La date d\'expiration doit être après la publication' }
)

// ❌ THROWS: Cannot extend a refined schema
const Extended = ValidatedContent.extend({ author: z.string() })
```

## ✅ Pattern recommandé : `.safeExtend()`

```typescript
// ✅ WORKS: safeExtend préserve les refinements
const Extended = ValidatedContent.safeExtend({
  author: z.string()
})

// Type narrowing also enforced - empêche les overwrites incompatibles
ValidatedContent.safeExtend({ title: z.string().min(10) }) // ✅ OK (même type)
ValidatedContent.safeExtend({ title: z.number() })          // ❌ Error (type différent)
```

## Quand utiliser `.safeExtend()`

| Situation | Méthode |
|-----------|---------|
| Schema sans refinement | `.extend()` ou spread |
| Schema avec `.refine()` | `.safeExtend()` |
| Schema avec `.superRefine()` | `.safeExtend()` |
| Composition depuis mixins | Spread syntax |

---
