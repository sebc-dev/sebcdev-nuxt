# Pattern 4 : Pick et Omit pour schemas dérivés

`.pick()` et `.omit()` créent des sous-schemas sans dupliquer la définition. Utile pour formulaires, API responses, ou previews.

## Extraire des champs spécifiques

```typescript
const FullArticle = z.object({
  id: z.string().uuid(),
  title: z.string(),
  content: z.string(),
  author: z.string(),
  publishedAt: z.iso.date(),
  updatedAt: z.iso.date().optional()
})

// Preview pour listes (sans contenu complet)
const ArticlePreview = FullArticle.pick({
  id: true,
  title: true,
  author: true,
  publishedAt: true
})
// { id: string; title: string; author: string; publishedAt: string }

// Schema formulaire (sans champs auto-générés)
const ArticleForm = FullArticle.omit({
  id: true,
  publishedAt: true,
  updatedAt: true
})
// { title: string; content: string; author: string }
```

## Combiner avec `.partial()` pour updates

```typescript
// Tous les champs deviennent optionnels
const ArticleUpdate = FullArticle
  .omit({ id: true })
  .partial()

type ArticleUpdate = z.infer<typeof ArticleUpdate>
// { title?: string; content?: string; author?: string; ... }

// Partial sur champs spécifiques uniquement
const PartialTitle = FullArticle.partial({ title: true, content: true })
// { id: string; title?: string; content?: string; author: string; ... }
```

## ⚠️ Limitation critique : Refinements perdus

**Les refinements ne survivent PAS à `.pick()` ou `.omit()`** :

```typescript
const WithRefinement = z.object({
  min: z.number(),
  max: z.number()
}).refine(d => d.max > d.min, { error: 'max doit être > min' })

const Picked = WithRefinement.pick({ min: true, max: true })
// ⚠️ Le refinement est SILENCIEUSEMENT PERDU !

// Solution : Réappliquer le refinement
const PickedWithValidation = WithRefinement
  .pick({ min: true, max: true })
  .refine(d => d.max > d.min, { error: 'max doit être > min' })
```

---
