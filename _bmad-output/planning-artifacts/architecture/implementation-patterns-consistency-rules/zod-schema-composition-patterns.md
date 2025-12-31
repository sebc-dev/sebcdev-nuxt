# Zod 4 Schema Composition Patterns

Patterns de composition de schemas Zod 4 optimisés pour Nuxt Content 3.10+ avec collections multilingues.

## Pattern 1 : Spread syntax pour performances TypeScript

**Problème** : Chaîner plusieurs appels `.extend()` cause une complexité **O(n²)** dans les instantiations TypeScript, ralentissant significativement `tsc` et l'IDE.

### ❌ Anti-pattern : Chaining `.extend()`

```typescript
// MAUVAIS: Chaque .extend() multiplie les instantiations TypeScript
const TechArticle = BaseContent
  .extend(SEOFields.shape)
  .extend({ codeLanguage: z.string().optional() })
  .extend({ repository: z.string().url().optional() })
```

### ✅ Pattern recommandé : Object spread

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

### Quand utiliser quoi

| Méthode | Usage | Performance tsc |
|---------|-------|-----------------|
| `.extend({ ... })` | Extension simple, 1 niveau | ✅ OK |
| `z.object({ ...A.shape, ...B.shape })` | Composition multiple | ✅ Optimal |
| `.extend().extend().extend()` | Jamais | ❌ Quadratique |

---

## Pattern 2 : `.safeExtend()` pour schemas raffinés (Zod 4.1+)

**Problème** : `.extend()` **lance une erreur** quand utilisé sur un schema avec `.refine()` ou `.superRefine()`.

### ❌ Anti-pattern : Extend sur schema raffiné

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

### ✅ Pattern recommandé : `.safeExtend()`

```typescript
// ✅ WORKS: safeExtend préserve les refinements
const Extended = ValidatedContent.safeExtend({
  author: z.string()
})

// Type narrowing also enforced - empêche les overwrites incompatibles
ValidatedContent.safeExtend({ title: z.string().min(10) }) // ✅ OK (même type)
ValidatedContent.safeExtend({ title: z.number() })          // ❌ Error (type différent)
```

### Quand utiliser `.safeExtend()`

| Situation | Méthode |
|-----------|---------|
| Schema sans refinement | `.extend()` ou spread |
| Schema avec `.refine()` | `.safeExtend()` |
| Schema avec `.superRefine()` | `.safeExtend()` |
| Composition depuis mixins | Spread syntax |

---

## Pattern 3 : Discriminated unions pour types de contenu

Les discriminated unions utilisent un champ discriminant (ex: `type`) pour un lookup **O(1)** au lieu de **O(n)** avec `z.union()`. Elles permettent aussi le type narrowing automatique en TypeScript.

### Schema de base avec discriminant

```typescript
import { z } from 'zod/v4'

// Base partagée entre tous les types de contenu
const BaseArticle = {
  title: z.string().min(1).max(100),
  slug: z.string(),
  draft: z.boolean().default(false),
  publishedAt: z.iso.date(),
  tags: z.array(z.string()).default([])
}

// Schemas avec discriminant `type`
const ArticleSchema = z.object({
  ...BaseArticle,
  type: z.literal('article'),
  description: z.string(),
  readingTime: z.number().positive()
})

const TutorialSchema = z.object({
  ...BaseArticle,
  type: z.literal('tutorial'),
  difficulty: z.enum(['beginner', 'intermediate', 'advanced']),
  prerequisites: z.array(z.string()).optional()
})

const NoteSchema = z.object({
  ...BaseArticle,
  type: z.literal('note'),
  body: z.string()
})

// Discriminated union - lookup O(1) par le champ 'type'
const ContentSchema = z.discriminatedUnion('type', [
  ArticleSchema,
  TutorialSchema,
  NoteSchema
])

type Content = z.infer<typeof ContentSchema>
```

### Type narrowing automatique

```typescript
function processContent(content: Content) {
  switch (content.type) {
    case 'article':
      // TypeScript sait que content.description existe
      console.log(content.description, content.readingTime)
      break
    case 'tutorial':
      // TypeScript sait que content.difficulty existe
      console.log(`${content.difficulty}: ${content.prerequisites?.length ?? 0} prérequis`)
      break
    case 'note':
      // TypeScript sait que content.body existe
      console.log(content.body)
      break
  }
}
```

### Composition de discriminated unions

```typescript
// Unions partielles
const BlogContent = z.discriminatedUnion('type', [ArticleSchema, NoteSchema])
const TechContent = z.discriminatedUnion('type', [TutorialSchema])

// Combiner via .options
const AllContent = z.discriminatedUnion('type', [
  ...BlogContent.options,
  ...TechContent.options
])
```

### Performance : discriminatedUnion vs union

| Aspect | `z.union()` | `z.discriminatedUnion()` |
|--------|-------------|-------------------------|
| **Validation** | Essaie chaque schema séquentiellement O(n) | Lookup par discriminant O(1) |
| **Messages d'erreur** | "Invalid union" générique | Spécifique au discriminant |
| **Type narrowing TS** | Type guards manuels requis | Automatique via discriminant |
| **Cas d'usage** | Types hétérogènes | Objets tagués/typés |

### Intégration Nuxt Content

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { asSeoCollection } from '@nuxtjs/seo/content'
import { z } from 'zod/v4'

// ... définition des schemas ci-dessus ...

export default defineContentConfig({
  collections: {
    articles_fr: defineCollection(
      asSeoCollection({
        type: 'page',
        source: { include: 'fr/**/*.md', prefix: '/blog' },
        schema: ContentSchema,  // Discriminated union
      })
    ),
    articles_en: defineCollection(
      asSeoCollection({
        type: 'page',
        source: { include: 'en/**/*.md', prefix: '/blog' },
        schema: ContentSchema,
      })
    ),
  }
})
```

---

## Pattern 4 : Pick et Omit pour schemas dérivés

`.pick()` et `.omit()` créent des sous-schemas sans dupliquer la définition. Utile pour formulaires, API responses, ou previews.

### Extraire des champs spécifiques

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

### Combiner avec `.partial()` pour updates

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

### ⚠️ Limitation critique : Refinements perdus

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

## Pattern 5 : Anti-patterns critiques

### ❌ Utiliser `.merge()` (déprécié Zod 4)

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

### ❌ Chaîner plusieurs `.extend()`

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

### ❌ Utiliser `z.intersection()` pour objets

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

### ❌ Ignorer input vs output avec transforms

```typescript
const DateField = z.string().transform(val => new Date(val))

// ❌ MAUVAIS: z.infer donne le type OUTPUT uniquement
type DateType = z.infer<typeof DateField>  // Date (output)

// ✅ CORRECT: Tracker les deux types
type DateInput = z.input<typeof DateField>   // string
type DateOutput = z.output<typeof DateField> // Date
```

### ❌ Oublier que refinements sont perdus avec pick/omit

```typescript
// ❌ BUG SILENCIEUX: Le refinement disparaît
const Subset = SchemaWithRefine.pick({ field: true })

// ✅ CORRECT: Réappliquer après pick/omit
const Subset = SchemaWithRefine
  .pick({ field: true })
  .refine(/* réappliquer la validation */)
```

### ❌ Import depuis `@nuxt/content`

```typescript
// ❌ DÉPRÉCIÉ: Re-export obsolète
import { z } from '@nuxt/content'

// ✅ CORRECT: Import direct avec API Zod 4
import { z } from 'zod/v4'
```

### ❌ Recréer schemas dans les composants

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

## Pattern 6 : `safeParse()` vs `parse()` - Stratégie d'usage

Le choix entre `parse()` et `safeParse()` détermine votre architecture de gestion d'erreurs.

### Quand utiliser `parse()` (throws)

```typescript
// Build-time : on VEUT que le build échoue sur erreur
// Nuxt Content valide au build - parse() approprié
schema.parse(frontmatter)  // Throws ZodError si invalide
```

**Cas d'usage :**
- Validation Nuxt Content (build-time)
- Données de confiance (API interne)
- Situations où l'échec doit stopper l'exécution

### Quand utiliser `safeParse()` (never throws)

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

### Per-parse error customization (Zod 4)

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

## Pattern 7 : Utilitaires de formatage d'erreurs

Zod 4 fournit trois utilitaires natifs pour formatter les erreurs. Ils remplacent `.format()` de Zod 3 (déprécié).

### `z.flattenError()` - Formulaires plats

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

### `z.treeifyError()` - Objets imbriqués

```typescript
const result = nestedSchema.safeParse(data)
if (!result.success) {
  const tree = z.treeifyError(result.error)
  // Structure miroir du schema avec .properties et .errors
  tree.properties?.address?.properties?.city?.errors
}
```

### `z.prettifyError()` - Logs et debugging

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

### Migration depuis Zod 3

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

## Pattern 8 : Localization avec `z.config()`

Zod 4 inclut **40+ locales natives** dont `fr` (français) et `frCA` (canadien).

### Configuration globale

```typescript
import { z } from 'zod/v4'

// Charger la locale française
z.config(z.locales.fr())

// Maintenant tous les messages d'erreur sont en français
const result = z.string().min(5).safeParse('abc')
// "La chaîne doit contenir au moins 5 caractère(s)"
```

### Lazy-loading pour optimisation bundle

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

### Hiérarchie de précédence des messages

Du plus prioritaire au moins prioritaire :

1. **Schema-level** : `z.string({ error: 'Message spécifique' })`
2. **Per-parse** : `schema.safeParse(data, { error: ... })`
3. **Global config** : `z.config({ ... })`
4. **Locale** : `z.config(z.locales.fr())`

### Messages personnalisés par champ

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

## Pattern 9 : zod/mini pour validation client-side

`zod/mini` (1.88kb gzip) est idéal pour la validation côté client quand la taille du bundle est critique.

### Différences avec Zod 4 complet

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

### ⚠️ Caveat critique : Pas de locale par défaut

```typescript
import * as z from 'zod/mini'

// SANS configuration : TOUS les messages = "Invalid input"
z.string().min(5).safeParse('ab')  // "Invalid input"

// AVEC configuration : Messages lisibles
z.config(z.locales.en())  // ou z.locales.fr()
z.string().min(5).safeParse('ab')  // "String must contain at least 5 character(s)"
```

### Recommandation pour le projet

| Contexte | Package | Raison |
|----------|---------|--------|
| Nuxt Content schemas | `zod/v4` (complet) | Build-time, pas dans le bundle client |
| Validation formulaires client | `zod/mini` | 1.88kb vs 5.36kb |
| Server routes API | `zod/v4` (complet) | Pas de contrainte bundle |

---

## Pattern 10 : VeeValidate + shadcn-vue integration

Pattern complet pour intégrer Zod avec les formulaires shadcn-vue via VeeValidate.

### Installation

```bash
pnpm add vee-validate @vee-validate/zod
```

### Composant formulaire typé

```vue
<!-- components/ContactForm.vue -->
<script setup lang="ts">
import { useForm } from 'vee-validate'
import { toTypedSchema } from '@vee-validate/zod'
import { z } from 'zod/v4'
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage
} from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
import { Button } from '@/components/ui/button'

// Schema Zod 4 avec messages FR
const contactSchema = z.object({
  email: z.email({ error: 'Adresse email invalide' }),
  name: z.string().min(2, { error: 'Le nom doit contenir au moins 2 caractères' }),
  message: z.string().min(10, { error: 'Message trop court (min. 10 caractères)' })
})

type ContactForm = z.infer<typeof contactSchema>

// Conversion pour VeeValidate
const validationSchema = toTypedSchema(contactSchema)

const { handleSubmit, errors, isSubmitting } = useForm<ContactForm>({
  validationSchema,
  initialValues: {
    email: '',
    name: '',
    message: ''
  }
})

const onSubmit = handleSubmit(async (values) => {
  // values est typé : { email: string; name: string; message: string }
  await $fetch('/api/contact', { method: 'POST', body: values })
})
</script>

<template>
  <Form @submit="onSubmit" class="space-y-4">
    <FormField v-slot="{ componentField }" name="email">
      <FormItem>
        <FormLabel>Email</FormLabel>
        <FormControl>
          <Input type="email" placeholder="vous@exemple.com" v-bind="componentField" />
        </FormControl>
        <FormMessage />
      </FormItem>
    </FormField>

    <FormField v-slot="{ componentField }" name="name">
      <FormItem>
        <FormLabel>Nom</FormLabel>
        <FormControl>
          <Input type="text" placeholder="Votre nom" v-bind="componentField" />
        </FormControl>
        <FormMessage />
      </FormItem>
    </FormField>

    <FormField v-slot="{ componentField }" name="message">
      <FormItem>
        <FormLabel>Message</FormLabel>
        <FormControl>
          <Textarea placeholder="Votre message..." v-bind="componentField" />
        </FormControl>
        <FormMessage />
      </FormItem>
    </FormField>

    <Button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Envoi...' : 'Envoyer' }}
    </Button>
  </Form>
</template>
```

### ⚠️ Caveat VeeValidate + Zod

Les méthodes `.refine()` et `.superRefine()` **ne s'exécutent pas** quand des clés d'objet sont manquantes. Solution :

```typescript
// ❌ refine() ignoré si passwordConfirm est undefined
const schema = z.object({
  password: z.string(),
  passwordConfirm: z.string()
}).refine(d => d.password === d.passwordConfirm)

// ✅ Toujours fournir des valeurs initiales
const { handleSubmit } = useForm({
  validationSchema: toTypedSchema(schema),
  initialValues: {
    password: '',
    passwordConfirm: ''  // REQUIS pour que refine() s'exécute
  }
})
```

---

## Pattern 11 : Vue `defineProps` et limitations du compilateur SFC

Le compilateur Vue SFC ne peut pas résoudre les types complexes importés depuis des fichiers externes. Utiliser `z.infer<typeof Schema>` importé cause l'erreur : `[@vue/compiler-sfc] Unresolvable type reference`.

### ❌ Anti-pattern : Import de type inféré

```typescript
// schemas/article.ts
export const ArticlePropsSchema = z.object({
  title: z.string(),
  slug: z.string()
})
export type ArticleProps = z.infer<typeof ArticlePropsSchema>
```

```vue
<!-- components/ArticleCard.vue -->
<script setup lang="ts">
// ❌ ERREUR: [@vue/compiler-sfc] Unresolvable type reference
import type { ArticleProps } from '@/schemas/article'
const props = defineProps<ArticleProps>()
</script>
```

### ✅ Solution 1 : Type alias dans le même fichier

```vue
<script setup lang="ts">
import { z } from 'zod/v4'

// Schema et type DANS LE MÊME FICHIER
const PropsSchema = z.object({
  title: z.string(),
  slug: z.string(),
  publishedAt: z.string()  // ISO date string après parsing Content
})

type Props = z.infer<typeof PropsSchema>
const props = defineProps<Props>()
</script>
```

### ✅ Solution 2 : Interface explicite séparée du schema

```typescript
// lib/content-schemas.ts
import { z } from 'zod/v4'

export const ArticleSchema = z.object({
  title: z.string(),
  slug: z.string(),
  publishedAt: z.iso.date()
})

// Interface EXPLICITE (pas z.infer) - résolvable par le compilateur Vue
export interface ArticleProps {
  title: string
  slug: string
  publishedAt: string
}
```

```vue
<script setup lang="ts">
// ✅ Interface explicite fonctionne
import type { ArticleProps } from '@/lib/content-schemas'
const props = defineProps<ArticleProps>()
</script>
```

### Quand utiliser quelle solution

| Situation | Solution recommandée |
|-----------|---------------------|
| Props simples (2-3 champs) | Solution 1 : Inline |
| Props réutilisées dans plusieurs composants | Solution 2 : Interface explicite |
| Schema avec transforms complexes | Solution 2 : Interface reflétant l'output |
| Données venant de Nuxt Content | Solution 2 : Interface matchant le type queryCollection |

### Pattern pour props venant de Nuxt Content

```typescript
// lib/content-schemas.ts
import { z } from 'zod/v4'

export const ArticleSchema = z.object({
  title: z.string(),
  description: z.string().optional(),
  publishedAt: z.iso.date(),
  pillar: z.enum(['ai', 'engineering', 'ux'])
})

// Interface pour les props de composants (après parsing Content)
export interface ArticleCardProps {
  title: string
  description?: string
  publishedAt: string  // ISO string après queryCollection
  pillar: 'ai' | 'engineering' | 'ux'
  path: string  // Ajouté par Content, pas dans le schema frontmatter
}
```

```vue
<!-- components/content/ArticleCard.vue -->
<script setup lang="ts">
import type { ArticleCardProps } from '@/lib/content-schemas'

const props = defineProps<ArticleCardProps>()
</script>

<template>
  <NuxtLink :to="props.path" class="block p-4 rounded-lg hover:bg-muted">
    <h3 class="font-semibold">{{ props.title }}</h3>
    <p v-if="props.description" class="text-muted-foreground">
      {{ props.description }}
    </p>
    <time :datetime="props.publishedAt" class="text-sm">
      {{ new Date(props.publishedAt).toLocaleDateString() }}
    </time>
  </NuxtLink>
</template>
```

---

## Référence : Schema complet blog multilingue

```typescript
// lib/content-schemas.ts
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

// Base pour tout le contenu blog
const BaseBlog = z.object({
  title: z.string().min(1).max(100),
  slug: z.string(),
  draft: z.boolean().default(false),
  author: z.string(),
  tags: z.array(z.string()).default([]),
  ...TimestampFields,
  ...SEOFields
})

// Schemas par type de contenu
export const ArticleSchema = z.object({
  ...BaseBlog.shape,
  type: z.literal('article'),
  pillar: z.enum(['ai', 'engineering', 'ux']),
  category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
  level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),
  readingTime: z.number().positive()
})

export const TutorialSchema = z.object({
  ...BaseBlog.shape,
  type: z.literal('tutorial'),
  difficulty: z.enum(['beginner', 'intermediate', 'advanced']),
  prerequisites: z.array(z.string()).optional(),
  estimatedTime: z.string().optional()
})

export const NoteSchema = z.object({
  ...BaseBlog.shape,
  type: z.literal('note')
})

// Discriminated union principale
export const BlogContentSchema = z.discriminatedUnion('type', [
  ArticleSchema,
  TutorialSchema,
  NoteSchema
])

// Types exportés
export type Article = z.infer<typeof ArticleSchema>
export type Tutorial = z.infer<typeof TutorialSchema>
export type Note = z.infer<typeof NoteSchema>
export type BlogContent = z.infer<typeof BlogContentSchema>

// Schemas dérivés pour différents usages
export const ArticlePreview = ArticleSchema.pick({
  title: true,
  slug: true,
  description: true,
  publishedAt: true,
  pillar: true,
  tags: true
})

export const ArticleForm = ArticleSchema.omit({
  type: true,
  slug: true,
  readingTime: true
})
```

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { asSeoCollection } from '@nuxtjs/seo/content'
import { BlogContentSchema } from './lib/content-schemas'

const articleIndexes = [
  { columns: ['path'], unique: true },
  { columns: ['pillar'] },
  { columns: ['publishedAt'] },
  { columns: ['draft'] },
  { columns: ['draft', 'publishedAt'], name: 'idx_published' },
]

export default defineContentConfig({
  collections: {
    articles_fr: defineCollection(
      asSeoCollection({
        type: 'page',
        source: { include: 'fr/**/*.md', prefix: '/blog' },
        schema: BlogContentSchema,
        indexes: articleIndexes,
      })
    ),
    articles_en: defineCollection(
      asSeoCollection({
        type: 'page',
        source: { include: 'en/**/*.md', prefix: '/blog' },
        schema: BlogContentSchema,
        indexes: articleIndexes,
      })
    ),
  }
})
```
