# Pattern 7 : Validation cross-champs avec .refine()

La méthode `.refine()` de Zod permet des validations qui dépendent de plusieurs champs :

## Règle : date future interdite si publié

```typescript
const articleSchema = z.object({
  title: z.string().min(1),
  publishedAt: z.iso.date(),  // Zod 4 : format YYYY-MM-DD
  draft: z.boolean().default(false),
}).refine(
  (data) => data.draft || new Date(data.publishedAt) <= new Date(),
  {
    error: 'La date de publication ne peut pas être future pour un article publié',  // Zod 4 : 'error' remplace 'message'
    path: ['publishedAt']  // Indique quel champ est en erreur
  }
)
```

## Règle : image obligatoire si featured

```typescript
const articleSchema = z.object({
  title: z.string(),
  featured: z.boolean().default(false),
  image: z.object({
    src: z.string(),
    alt: z.string()
  }).optional(),
}).refine(
  (data) => !data.featured || data.image !== undefined,
  {
    error: 'Une image est requise pour les articles mis en avant',  // Zod 4 : 'error' remplace 'message'
    path: ['image']
  }
)
```

## Chaîner plusieurs validations

```typescript
const articleSchema = z.object({
  title: z.string(),
  publishedAt: z.iso.date(),           // Zod 4 : format YYYY-MM-DD
  updatedAt: z.iso.date().optional(),  // Zod 4 : format YYYY-MM-DD
  draft: z.boolean().default(false),
  featured: z.boolean().default(false),
  image: z.object({ src: z.string(), alt: z.string() }).optional(),
})
  .refine(
    (data) => data.draft || new Date(data.publishedAt) <= new Date(),
    { error: 'Date future interdite si publié', path: ['publishedAt'] }  // Zod 4 : 'error'
  )
  .refine(
    (data) => !data.updatedAt || data.updatedAt >= data.publishedAt,
    { error: 'updatedAt doit être >= publishedAt', path: ['updatedAt'] }  // Zod 4 : 'error'
  )
  .refine(
    (data) => !data.featured || data.image,
    { error: 'Image requise si featured', path: ['image'] }  // Zod 4 : 'error'
  )
```

**Notes Zod 4 :**
- Les erreurs de validation `.refine()` apparaissent en console au build time mais **ne font pas échouer le build par défaut**. Les valeurs invalides sont omises du résultat.
- `{ message: ... }` est remplacé par `{ error: ... }` en Zod 4
- `z.iso.date()` retourne une string `YYYY-MM-DD`, donc comparaison de dates nécessite `new Date()`

## Error customization avancée (Zod 4)

Le paramètre `error` accepte une fonction pour des messages dynamiques :

```typescript
z.string({
  error: (issue) => issue.input === undefined
    ? 'Ce champ est requis'
    : 'Valeur invalide'
})

// Ou avec .refine()
.refine(
  (data) => data.endDate >= data.startDate,
  {
    error: (issue) => `La date de fin doit être après ${issue.ctx.startDate}`,
    path: ['endDate']
  }
)
```

## Recursive schemas avec getters (Zod 4)

Zod 4 introduit une syntaxe plus propre pour les schemas récursifs, remplaçant `z.lazy()` :

```typescript
// ❌ Zod 3 : Verbose avec z.lazy() et annotation de type manuelle
interface Category {
  name: string;
  subcategories: Category[];
}

const CategorySchema: z.ZodType<Category> = z.object({
  name: z.string(),
  subcategories: z.lazy(() => CategorySchema.array()),
});

// ✅ Zod 4 : Getter natif - plus propre et typé automatiquement
const Category = z.object({
  name: z.string(),
  get subcategories() {
    return z.array(Category);
  }
});

type Category = z.infer<typeof Category>;
// Infère correctement : { name: string; subcategories: Category[] }
```

**Avantages :**
- Pas besoin d'annotation de type manuelle
- Accès à `.pick()`, `.omit()`, `.partial()`, `.extend()` sur le schema
- Syntaxe JavaScript native (getter)

**Cas spéciaux :**
- Pour les unions récursives hors objets ou `z.record()`, `z.lazy()` reste nécessaire
- Si TypeScript signale "implicitly has return type 'any'", ajouter une annotation :
  ```typescript
  get subcategories(): z.ZodArray<typeof Category> {
    return z.array(Category);
  }
  ```

## Date coercion avec `.pipe()` (Zod 4)

Pour obtenir des objets `Date` JavaScript à partir de strings ISO du frontmatter :

```typescript
// Pattern recommandé : validation format PUIS coercion
const articleSchema = z.object({
  // Valide le format ISO, puis convertit en Date object
  publishedAt: z.iso.datetime({ offset: true }).pipe(z.coerce.date()),

  // Avec contraintes temporelles
  createdAt: z.iso.date()
    .pipe(z.coerce.date())
    .pipe(z.date()
      .min(new Date("2020-01-01"), { error: "Trop ancien" })
      .max(new Date(), { error: "Date future interdite" })
    )
})
```

**Quand utiliser quel pattern :**

| Pattern | Output | Usage |
|---------|--------|-------|
| `z.iso.date()` | `string` (YYYY-MM-DD) | Stockage JSON Schema, comparaisons string |
| `z.iso.datetime()` | `string` (ISO 8601) | Timestamps avec timezone |
| `z.coerce.date()` | `Date` object | Calculs, formatage, manipulation |
| `.pipe(z.coerce.date())` | `Date` object validé | Validation format + conversion |

**⚠️ Attention** : `z.coerce.date()` seul produit des erreurs confuses si la coercion échoue. Toujours valider le format avec `z.iso.*` avant.

## Array unique et normalisé (tags)

Pattern pour valider l'unicité et normaliser les tags du frontmatter :

```typescript
// Pattern complet : normalisation + unicité + contraintes
const TagsSchema = z.array(z.string())
  // 1. Normaliser (trim, lowercase, dedupe)
  .transform(tags => [...new Set(tags.map(t => t.trim().toLowerCase()))])
  // 2. Valider le résultat normalisé
  .pipe(z.array(z.string()).min(1).max(10))

// Ou avec validation explicite de l'unicité (messages personnalisés)
const TagsWithErrors = z.array(z.string())
  .min(1, { error: 'Au moins un tag requis' })
  .max(10, { error: 'Maximum 10 tags' })
  .superRefine((tags, ctx) => {
    const unique = new Set(tags)
    if (tags.length !== unique.size) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: 'Les tags doivent être uniques'
      })
    }
  })
```

**Usage dans le schema complet :**

```typescript
const articleSchema = z.object({
  title: z.string().min(1).max(200),
  tags: z.array(z.string())
    .transform(tags => [...new Set(tags.map(t => t.trim().toLowerCase()))])
    .pipe(z.array(z.string()).min(1).max(10))
    .default(['general']),
})
```

## Build-time validation : limitations importantes

**⚠️ Comportement critique à connaître :**

Nuxt Content 3 valide le frontmatter uniquement au **build time** (`nuxt generate`). Les erreurs de validation sont **silencieuses** :

| Situation | Comportement | Conséquence |
|-----------|--------------|-------------|
| Champ requis manquant | Valeur = `undefined` | Pas d'erreur explicite |
| Valeur invalide | Champ omis du résultat | Données partielles |
| Type incorrect | Coercion ou omission | Comportement imprévisible |

**Stratégies de mitigation :**

1. **Defaults sensibles** : Toujours définir des `.default()` pour les champs critiques
2. **Messages d'erreur explicites** : Utiliser `{ error: "..." }` pour le debugging
3. **Validation externe** : Linter frontmatter en pre-commit hook
4. **Tests de contenu** : Vérifier les champs attendus dans les tests E2E

```typescript
// Schema défensif avec defaults
const articleSchema = z.object({
  title: z.string().min(1).default('Sans titre'),  // Jamais undefined
  category: z.enum(['tutorial', 'news'], {
    error: 'Catégorie invalide - doit être "tutorial" ou "news"'
  }).default('news'),
  draft: z.boolean().default(true),  // Sécurité : draft par défaut
})
```
