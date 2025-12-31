# Pattern 7 : Utilitaires de formatage d'erreurs

Zod 4 fournit trois utilitaires natifs pour formatter les erreurs. Ils remplacent `.format()` de Zod 3 (déprécié).

## `z.flattenError()` - Formulaires plats

```typescript
const result = schema.safeParse(formData)
if (!result.success) {
  const flat = z.flattenError(result.error)
  // {
  //   formErrors: ['Erreur globale'],
  //   fieldErrors: {
  //     email: ['Email invalide'],
  //     password: ['Trop court', 'Doit contenir un chiffre']
  //   }
  // }
}
```

## `z.treeifyError()` - Objets imbriqués

```typescript
const result = nestedSchema.safeParse(data)
if (!result.success) {
  const tree = z.treeifyError(result.error)
  // Structure miroir du schema avec .properties et .errors
  tree.properties?.address?.properties?.city?.errors
}
```

## `z.prettifyError()` - Logs et debugging

```typescript
if (!result.success) {
  console.log(z.prettifyError(result.error))
}
// Output:
// ✖ Invalid input: expected string, received number
//   → at username
// ✖ Invalid input: expected number, received string
//   → at favoriteNumbers[1]
```

## Migration depuis Zod 3

```typescript
// ❌ Zod 3 (déprécié)
const formatted = result.error.format()
formatted.email?._errors

// ✅ Zod 4
const tree = z.treeifyError(result.error)
tree.properties?.email?.errors
```

| Utilitaire | Structure | Cas d'usage |
|------------|-----------|-------------|
| `z.flattenError()` | `{ formErrors, fieldErrors }` | Formulaires simples |
| `z.treeifyError()` | Miroir schema avec `.properties` | Objets imbriqués |
| `z.prettifyError()` | String lisible | Logs, debugging |

---
