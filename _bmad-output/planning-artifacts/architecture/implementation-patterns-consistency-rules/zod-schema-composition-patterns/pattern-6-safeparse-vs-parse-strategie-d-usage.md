# Pattern 6 : `safeParse()` vs `parse()` - Stratégie d'usage

Le choix entre `parse()` et `safeParse()` détermine votre architecture de gestion d'erreurs.

## Quand utiliser `parse()` (throws)

```typescript
// Build-time : on VEUT que le build échoue sur erreur
// Nuxt Content valide au build - parse() approprié
schema.parse(frontmatter)  // Throws ZodError si invalide
```

**Cas d'usage :**
- Validation Nuxt Content (build-time)
- Données de confiance (API interne)
- Situations où l'échec doit stopper l'exécution

## Quand utiliser `safeParse()` (never throws)

```typescript
// Runtime : on veut afficher les erreurs gracieusement
const result = schema.safeParse(userInput)

if (!result.success) {
  // Discriminated union - TypeScript sait que result.error existe
  const errors = z.flattenError(result.error)
  // { formErrors: string[], fieldErrors: { email: string[], password: string[] } }
  displayErrors(errors.fieldErrors)
} else {
  // TypeScript sait que result.data est fully typed
  submitForm(result.data)
}
```

**Cas d'usage :**
- Formulaires utilisateur
- Input non fiable (API externe)
- Affichage d'erreurs dans l'UI

## Per-parse error customization (Zod 4)

```typescript
// Override messages contextuellement sans modifier le schema
schema.safeParse(data, {
  error: (iss) => {
    if (iss.code === 'invalid_type') return `Attendu ${iss.expected}`
    if (iss.code === 'too_small') return `Minimum: ${iss.minimum}`
  }
})
```

---
