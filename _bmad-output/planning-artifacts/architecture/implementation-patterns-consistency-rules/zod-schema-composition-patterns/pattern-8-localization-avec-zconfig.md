# Pattern 8 : Localization avec `z.config()`

Zod 4 inclut **40+ locales natives** dont `fr` (français) et `frCA` (canadien).

## Configuration globale

```typescript
import { z } from 'zod/v4'

// Charger la locale française
z.config(z.locales.fr())

// Maintenant tous les messages d'erreur sont en français
const result = z.string().min(5).safeParse('abc')
// "La chaîne doit contenir au moins 5 caractère(s)"
```

## Lazy-loading pour optimisation bundle

```typescript
// plugins/zod-locale.ts
export default defineNuxtPlugin(async () => {
  const { locale } = useI18n()

  if (locale.value === 'fr') {
    const { default: fr } = await import('zod/v4/locales/fr.js')
    z.config(fr())
  }
})
```

## Hiérarchie de précédence des messages

Du plus prioritaire au moins prioritaire :

1. **Schema-level** : `z.string({ error: 'Message spécifique' })`
2. **Per-parse** : `schema.safeParse(data, { error: ... })`
3. **Global config** : `z.config({ ... })`
4. **Locale** : `z.config(z.locales.fr())`

## Messages personnalisés par champ

```typescript
const articleSchema = z.object({
  title: z.string({
    error: (iss) => iss.input === undefined
      ? 'Le titre est requis'
      : 'Titre invalide'
  }),
  slug: z.string().regex(/^[a-z0-9-]+$/, {
    error: 'Format URL slug invalide (lettres minuscules, chiffres, tirets)'
  })
})
```

---
