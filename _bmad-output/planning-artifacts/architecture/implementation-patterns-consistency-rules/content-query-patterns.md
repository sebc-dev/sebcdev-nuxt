# Content Query Patterns

Patterns de requêtes @nuxt/content v3 pour le blog multilingue avec collections séparées par langue (`articles_fr`, `articles_en`).

## Contrainte fondamentale : un document = une collection

**Limitation critique** (issue nuxt/content#2966) : Un document Markdown ne peut appartenir qu'à **UNE SEULE collection**. Cette contrainte explique pourquoi nous utilisons des collections séparées par locale plutôt qu'une collection unique avec filtre.

| Approche | Faisabilité | Raison |
|----------|-------------|--------|
| Collection unique + filtre locale | ❌ Impossible | Document ne peut pas être dans 2 collections |
| Collections séparées (`articles_fr`, `articles_en`) | ✅ Recommandée | Chaque fichier dans sa collection dédiée |

**Conséquence** : Les fichiers `content/fr/` et `content/en/` appartiennent à des collections distinctes, même si leur structure est identique.

## Configuration source avec prefix pour coordination i18n

Le `prefix` dans la configuration source **retire le préfixe de langue** du path généré, permettant à @nuxtjs/i18n de gérer les URLs :

```typescript
// content.config.ts
export default defineContentConfig({
  collections: {
    articles_en: defineCollection({
      source: {
        include: 'en/blog/**',
        prefix: '/blog'  // Génère /blog/article au lieu de /en/blog/article
      },
    }),
    articles_fr: defineCollection({
      source: {
        include: 'fr/blog/**',
        prefix: '/blog'  // Génère /blog/article au lieu de /fr/blog/article
      },
    }),
  }
})
```

**Coordination Content + i18n :**

| Fichier source | Path Content | URL finale (i18n) |
|----------------|--------------|-------------------|
| `content/en/blog/my-post.md` | `/blog/my-post` | `/blog/my-post` (EN, défaut) |
| `content/fr/blog/mon-article.md` | `/blog/mon-article` | `/fr/blog/mon-article` |

Le prefix `/blog` dans source évite la duplication du préfixe de langue entre Content et i18n.

## Pattern 1 : Requête avec locale dynamique typée

Utiliser le cast TypeScript `as keyof Collections` pour les noms de collections dynamiques :

```typescript
// composables/useArticles.ts
import type { Collections } from '@nuxt/content'

export function useArticles() {
  const { locale } = useI18n()

  const getCollection = () => {
    // Cast sécurisé pour TypeScript - collection dynamique selon locale
    return `articles_${locale.value}` as keyof Collections
  }

  const { data: articles } = useAsyncData(
    `articles-${locale.value}`,
    () => queryCollection(getCollection())
      .where('draft', '=', false)
      .order('publishedAt', 'DESC')
      .all()
  )

  return { articles }
}
```

**Avantage collections séparées :** La requête interroge uniquement la table de la langue courante, réduisant les rows_read D1.

## Pattern 2 : Re-fetch automatique au changement de langue

L'option `watch: [locale]` déclenche automatiquement un re-fetch quand la locale change :

```typescript
// pages/blog/index.vue
<script setup lang="ts">
import type { Collections } from '@nuxt/content'

const { locale } = useI18n()

const { data: posts } = await useAsyncData(
  `blog-list-${locale.value}`,
  () => {
    const collection = `articles_${locale.value}` as keyof Collections
    return queryCollection(collection)
      .where('draft', '=', false)
      .order('publishedAt', 'DESC')
      .all()
  },
  {
    watch: [locale]  // Re-fetch automatique au changement de langue
  }
)
</script>
```

**Comportement :**
- Premier chargement : fetch des données de la collection courante
- Changement de locale (ex: FR → EN) : fetch de `articles_en` au lieu de `articles_fr`
- La clé `blog-list-${locale.value}` garantit un cache séparé par langue

## Pattern 3 : Fallback vers locale par défaut

Graceful degradation pour les articles non traduits :

```typescript
// pages/blog/[...slug].vue
<script setup lang="ts">
import { withLeadingSlash } from 'ufo'
import type { Collections } from '@nuxt/content'

const route = useRoute()
const { locale, defaultLocale } = useI18n()

const slug = computed(() => withLeadingSlash(String(route.params.slug || '')))

const { data: result } = await useAsyncData(
  `article-${locale.value}-${slug.value}`,
  async () => {
    // Tentative de récupération dans la collection de la locale courante
    const collection = `articles_${locale.value}` as keyof Collections
    const content = await queryCollection(collection)
      .path(slug.value)
      .first()

    if (content) {
      return { content, isFallback: false }
    }

    // Fallback vers locale par défaut si contenu manquant
    if (locale.value !== defaultLocale.value) {
      const fallbackCollection = `articles_${defaultLocale.value}` as keyof Collections
      const fallbackContent = await queryCollection(fallbackCollection)
        .path(slug.value)
        .first()

      if (fallbackContent) {
        console.warn(`[i18n] Article "${slug.value}" non traduit en ${locale.value}, fallback vers ${defaultLocale.value}`)
        return { content: fallbackContent, isFallback: true }
      }
    }

    return { content: null, isFallback: false }
  },
  { watch: [locale] }
)

const article = computed(() => result.value?.content)
const isFallback = computed(() => result.value?.isFallback ?? false)
</script>

<template>
  <div v-if="isFallback" class="bg-amber-100 dark:bg-amber-900/30 p-4 rounded-lg mb-6">
    <p class="text-amber-800 dark:text-amber-200">
      Cet article n'est pas encore traduit dans votre langue.
    </p>
  </div>

  <article v-if="article">
    <ContentRenderer :value="article" />
  </article>

  <div v-else>
    <!-- 404 ou message d'erreur -->
  </div>
</template>
```

**UX recommandée :**
- Afficher un bandeau "Cet article n'est pas encore traduit" si fallback utilisé
- Proposer un lien vers la version originale

## Pattern 4 : Composable réutilisable complet

```typescript
// composables/useLocalizedContent.ts
import type { Collections } from '@nuxt/content'

interface LocalizedContentResult<T> {
  content: T | null
  isFallback: boolean
  originalLocale: string
}

interface UseLocalizedContentOptions {
  fallbackToDefault?: boolean
}

export function useLocalizedContent(
  path: MaybeRef<string>,
  options: UseLocalizedContentOptions = { fallbackToDefault: true }
) {
  const { locale, defaultLocale } = useI18n()
  const resolvedPath = toRef(path)

  const { data, pending, error } = useAsyncData(
    `content-${locale.value}-${resolvedPath.value}`,
    async (): Promise<LocalizedContentResult<unknown>> => {
      // Collection de la locale courante
      const collection = `articles_${locale.value}` as keyof Collections
      const content = await queryCollection(collection)
        .path(resolvedPath.value)
        .first()

      if (content) {
        return { content, isFallback: false, originalLocale: locale.value }
      }

      // Fallback vers locale par défaut
      if (options.fallbackToDefault && locale.value !== defaultLocale.value) {
        const fallbackCollection = `articles_${defaultLocale.value}` as keyof Collections
        const fallbackContent = await queryCollection(fallbackCollection)
          .path(resolvedPath.value)
          .first()

        if (fallbackContent) {
          return {
            content: fallbackContent,
            isFallback: true,
            originalLocale: defaultLocale.value
          }
        }
      }

      return { content: null, isFallback: false, originalLocale: locale.value }
    },
    { watch: [locale, resolvedPath] }
  )

  const content = computed(() => data.value?.content ?? null)
  const isFallback = computed(() => data.value?.isFallback ?? false)
  const originalLocale = computed(() => data.value?.originalLocale ?? locale.value)

  return { content, isFallback, originalLocale, pending, error }
}
```

**Usage :**

```vue
<script setup lang="ts">
const route = useRoute()
const { content, isFallback, originalLocale } = useLocalizedContent(() => route.path)
</script>

<template>
  <div v-if="isFallback" class="bg-amber-100 p-4 rounded">
    Cet article est affiché en {{ originalLocale }} car il n'est pas encore traduit.
  </div>
  <ContentRenderer v-if="content" :value="content" />
</template>
```

## Pattern 5 : Helper pour nom de collection

Pour éviter la répétition du cast TypeScript :

```typescript
// composables/useContentCollection.ts
import type { Collections } from '@nuxt/content'

/**
 * Retourne le nom de collection articles pour la locale courante
 */
export function useArticlesCollection() {
  const { locale } = useI18n()

  const collectionName = computed(() =>
    `articles_${locale.value}` as keyof Collections
  )

  return collectionName
}

// Usage simplifié
const collection = useArticlesCollection()
const posts = await queryCollection(collection.value).all()
```

## Résumé des patterns

| Pattern | Cas d'usage | Avantage collections séparées |
|---------|-------------|-------------------------------|
| Locale dynamique typée | Requêtes multilingues | `articles_${locale}` - table isolée |
| Watch locale | Listes d'articles | Re-fetch sur collection différente |
| Fallback locale | Articles non traduits | Requête fallback explicite |
| Composable complet | Réutilisation projet-wide | Abstraction complète |
| Helper collection | DRY | Évite répétition du cast |

## Mapping collections ↔ routes

| Locale | Collection | Route générée |
|--------|------------|---------------|
| `fr` | `articles_fr` | `/blog/mon-article` |
| `en` | `articles_en` | `/en/blog/my-article` |

Le `prefix: '/blog'` dans `content.config.ts` retire le préfixe de langue du path.

## Pattern 6 : Property Editors pour Nuxt Studio

La fonction `property()` de `@nuxt/content` encapsule un schema Zod et expose la méthode `.editor()` pour configurer l'interface visuelle dans Nuxt Studio.

### Configuration des inputs visuels

```typescript
// content.config.ts
import { defineContentConfig, defineCollection, property } from '@nuxt/content'
import { z } from 'zod/v4'

const articleSchema = z.object({
  title: z.string().min(1).max(100),

  // Media picker - ouvre la bibliothèque de médias Studio
  coverImage: property(z.string()).editor({ input: 'media' }),

  // Icon picker - sélecteur d'icônes Iconify
  icon: property(z.string().optional()).editor({
    input: 'icon',
    iconLibraries: ['lucide', 'heroicons', 'simple-icons']
  }),

  // Champ masqué dans l'éditeur (calculé automatiquement)
  slug: property(z.string()).editor({ hidden: true }),

  // Champ avec image et alt
  image: z.object({
    src: property(z.string()).editor({ input: 'media' }),
    alt: z.string(),
    width: z.number().optional(),
    height: z.number().optional()
  }).optional(),
})
```

### Types d'inputs disponibles

| Type Zod | Input Studio | Comportement |
|----------|--------------|--------------|
| `z.string()` | Texte | Input texte standard |
| `z.string().editor({ input: 'media' })` | Media picker | Ouvre bibliothèque médias |
| `z.string().editor({ input: 'icon' })` | Icon picker | Sélecteur icônes Iconify |
| `z.iso.date()` | Date picker | Calendrier (format `YYYY-MM-DD`) |
| `z.iso.datetime()` | Datetime picker | Calendrier + heure (ISO 8601) |
| `z.boolean()` | Toggle | Switch on/off |
| `z.enum([...])` | Select dropdown | Liste déroulante |
| `z.array(z.string())` | Tags | Liste de badges |
| `z.number()` | Number | Input numérique |

**⚠️ Zod 4** : Utiliser `z.iso.date()` (string `YYYY-MM-DD`) au lieu de `z.date()` (Date object) pour la compatibilité JSON Schema.

### Héritage de props de composants Vue

Pour un champ qui correspond aux props d'un composant Vue :

```typescript
// content.config.ts
const articleSchema = z.object({
  // Hérite automatiquement des props de HeroSection.vue
  hero: property(z.object({})).inherit('components/HeroSection.vue'),
})
```

### Exemple complet avec tous les types

```typescript
// content.config.ts
import { defineContentConfig, defineCollection, property } from '@nuxt/content'
import { asSeoCollection } from '@nuxtjs/seo/content'
import { z } from 'zod/v4'

const imageSchema = z.object({
  src: property(z.string()).editor({ input: 'media' }),
  alt: z.string(),
  width: z.number().optional(),
  height: z.number().optional()
})

const articleSchema = z.object({
  // Texte standard
  title: z.string().min(1).max(100),
  description: z.string().max(300).optional(),

  // Date picker (Zod 4 : z.iso.date() pour format YYYY-MM-DD)
  publishedAt: z.iso.date(),
  updatedAt: z.iso.date().optional(),

  // Select dropdown (enum)
  pillar: z.enum(['ai', 'engineering', 'ux']),
  category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
  level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),

  // Tags (array de strings)
  tags: z.array(z.string()).default([]),

  // Toggle boolean
  draft: z.boolean().default(false),
  featured: z.boolean().default(false),

  // Media picker
  image: imageSchema.optional(),

  // Icon picker
  icon: property(z.string().optional()).editor({
    input: 'icon',
    iconLibraries: ['lucide', 'heroicons']
  }),

  // Champ caché (auto-calculé)
  readingTime: property(z.number().optional()).editor({ hidden: true }),
})

export default defineContentConfig({
  collections: {
    articles_fr: defineCollection(
      asSeoCollection({
        type: 'page',
        source: { include: 'fr/**/*.md', prefix: '/blog' },
        schema: articleSchema,
      })
    ),
  }
})
```

## Pattern 7 : Validation cross-champs avec .refine()

La méthode `.refine()` de Zod permet des validations qui dépendent de plusieurs champs :

### Règle : date future interdite si publié

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

### Règle : image obligatoire si featured

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

### Chaîner plusieurs validations

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

### Error customization avancée (Zod 4)

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

### Recursive schemas avec getters (Zod 4)

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

### Date coercion avec `.pipe()` (Zod 4)

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

### Array unique et normalisé (tags)

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

### Build-time validation : limitations importantes

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

## Pattern 8 : Fallback component pour erreurs de validation

Gérer gracieusement les contenus avec erreurs de validation côté composant :

### Composant FallbackContent

```vue
<!-- components/content/FallbackContent.vue -->
<script setup lang="ts">
interface Props {
  message?: string
  showHomeLink?: boolean
}

withDefaults(defineProps<Props>(), {
  message: 'Ce contenu n\'est pas disponible.',
  showHomeLink: true
})

const localePath = useLocalePath()
</script>

<template>
  <div class="flex flex-col items-center justify-center py-16 text-center">
    <div class="rounded-lg bg-muted p-8 max-w-md">
      <Icon name="lucide:file-warning" class="w-12 h-12 text-muted-foreground mb-4" />
      <p class="text-muted-foreground mb-4">{{ message }}</p>
      <NuxtLink
        v-if="showHomeLink"
        :to="localePath('/')"
        class="text-primary hover:underline"
      >
        Retour à l'accueil
      </NuxtLink>
    </div>
  </div>
</template>
```

### Usage dans les pages

```vue
<!-- pages/blog/[...slug].vue -->
<script setup lang="ts">
import type { Collections } from '@nuxt/content'

const route = useRoute()
const { locale } = useI18n()

const { data: article } = await useAsyncData(
  `article-${locale.value}-${route.path}`,
  async () => {
    const collection = `articles_${locale.value}` as keyof Collections
    return queryCollection(collection).path(route.path).first()
  }
)

// 404 si article null (validation échouée ou inexistant)
if (!article.value) {
  throw createError({
    statusCode: 404,
    statusMessage: 'Article non trouvé'
  })
}
</script>

<template>
  <article v-if="article">
    <ContentRenderer :value="article" />
  </article>
  <FallbackContent v-else message="Cet article n'a pas pu être chargé." />
</template>
```

### Pattern avec état de chargement

```vue
<script setup lang="ts">
const { data: article, pending, error } = await useAsyncData(...)
</script>

<template>
  <!-- Loading state -->
  <div v-if="pending" class="animate-pulse">
    <div class="h-8 bg-muted rounded w-3/4 mb-4" />
    <div class="h-4 bg-muted rounded w-full mb-2" />
    <div class="h-4 bg-muted rounded w-5/6" />
  </div>

  <!-- Error state -->
  <FallbackContent
    v-else-if="error"
    message="Une erreur est survenue lors du chargement."
  />

  <!-- Empty state (validation failed) -->
  <FallbackContent
    v-else-if="!article"
    message="Ce contenu n'est pas disponible."
  />

  <!-- Success state -->
  <ContentRenderer v-else :value="article" />
</template>
```

## Pattern 9 : Lier les versions traduites (Language Switcher)

### Problème

Il n'existe **pas de connexion native** entre les versions localisées d'un même contenu dans Nuxt Content 3. Le language switcher ne peut pas automatiquement trouver la version traduite d'un article.

### Solution : UID de traduction dans le frontmatter

Ajouter un champ `translationUid` identique dans les deux versions :

```yaml
# content/en/blog/getting-started.md
---
title: "Getting Started with Nuxt"
slug: "getting-started"
translationUid: "nuxt-intro-001"  # ← UID partagé
---

# content/fr/blog/demarrage.md
---
title: "Démarrer avec Nuxt"
slug: "demarrage"
translationUid: "nuxt-intro-001"  # ← Même UID
---
```

### Schema Zod avec translationUid

```typescript
// content.config.ts
const articleSchema = z.object({
  title: z.string(),
  slug: z.string(),
  // ... autres champs
  translationUid: z.string().optional(),  // UID pour lier les traductions
})
```

### Composable pour trouver la version traduite

```typescript
// composables/useTranslatedArticle.ts
import type { Collections } from '@nuxt/content'

export async function useTranslatedArticle(
  translationUid: string | undefined,
  targetLocale: string
) {
  if (!translationUid) return null

  const collection = `articles_${targetLocale}` as keyof Collections

  const translated = await queryCollection(collection)
    .where('translationUid', '=', translationUid)
    .first()

  return translated
}
```

### Language Switcher avec lien vers traduction

```vue
<!-- app/components/content/ArticleLanguageSwitcher.vue -->
<script setup lang="ts">
import type { Collections } from '@nuxt/content'

interface Props {
  translationUid?: string
}

const props = defineProps<Props>()
const { locale, locales } = useI18n()
const localePath = useLocalePath()

// Trouver les traductions disponibles
const { data: translations } = await useAsyncData(
  `translations-${props.translationUid}`,
  async () => {
    if (!props.translationUid) return {}

    const results: Record<string, string> = {}

    for (const loc of locales.value) {
      if (loc.code === locale.value) continue

      const collection = `articles_${loc.code}` as keyof Collections
      const translated = await queryCollection(collection)
        .where('translationUid', '=', props.translationUid)
        .select('path')
        .first()

      if (translated) {
        results[loc.code] = translated.path
      }
    }

    return results
  }
)
</script>

<template>
  <nav v-if="translations && Object.keys(translations).length > 0" class="flex gap-2">
    <span class="text-muted-foreground">Lire en :</span>
    <NuxtLink
      v-for="(path, code) in translations"
      :key="code"
      :to="path"
      class="text-primary hover:underline"
    >
      {{ code.toUpperCase() }}
    </NuxtLink>
  </nav>
</template>
```

### Alternative : slugs identiques

Si les slugs sont identiques entre langues, `useSwitchLocalePath()` fonctionne directement :

```vue
<script setup>
const switchLocalePath = useSwitchLocalePath()
</script>

<template>
  <!-- Fonctionne si /blog/my-article et /fr/blog/my-article existent -->
  <NuxtLink :to="switchLocalePath('fr')">
    Lire en français
  </NuxtLink>
</template>
```

## Pattern 10 : Fallback locale avec message explicite

Afficher un message clair quand un article n'est pas traduit :

```vue
<!-- pages/blog/[...slug].vue -->
<script setup lang="ts">
import type { Collections } from '@nuxt/content'

const route = useRoute()
const { locale, defaultLocale, t } = useI18n()

const slug = computed(() => `/${(route.params.slug as string[]).join('/')}`)

const { data: result } = await useAsyncData(
  `article-${slug.value}`,  // Clé sans locale pour cache partagé
  async () => {
    const collection = `articles_${locale.value}` as keyof Collections
    let content = await queryCollection(collection).path(slug.value).first()

    if (content) {
      return { content, isFallback: false, displayLocale: locale.value }
    }

    // Fallback vers locale par défaut
    if (locale.value !== defaultLocale.value) {
      const fallbackCollection = `articles_${defaultLocale.value}` as keyof Collections
      content = await queryCollection(fallbackCollection).path(slug.value).first()

      if (content) {
        return { content, isFallback: true, displayLocale: defaultLocale.value }
      }
    }

    return { content: null, isFallback: false, displayLocale: locale.value }
  },
  { watch: [locale] }
)

const article = computed(() => result.value?.content)
const isFallback = computed(() => result.value?.isFallback ?? false)
const displayLocale = computed(() => result.value?.displayLocale ?? locale.value)
</script>

<template>
  <!-- Bandeau fallback -->
  <div
    v-if="isFallback"
    class="bg-amber-50 dark:bg-amber-950 border-l-4 border-amber-500 p-4 mb-6"
    role="alert"
  >
    <div class="flex items-start gap-3">
      <Icon name="lucide:globe" class="w-5 h-5 text-amber-600 flex-shrink-0 mt-0.5" />
      <div>
        <p class="font-medium text-amber-800 dark:text-amber-200">
          {{ t('content.notTranslated') }}
        </p>
        <p class="text-sm text-amber-700 dark:text-amber-300 mt-1">
          {{ t('content.showingIn', { locale: displayLocale }) }}
        </p>
      </div>
    </div>
  </div>

  <!-- Contenu article -->
  <article v-if="article">
    <ContentRenderer :value="article" />
  </article>

  <!-- 404 -->
  <FallbackContent v-else :message="t('content.notFound')" />
</template>
```

### Traductions pour le message fallback

```json
// locales/fr.json
{
  "content": {
    "notTranslated": "Cet article n'est pas encore traduit en français.",
    "showingIn": "Affichage en {locale}.",
    "notFound": "Article introuvable."
  }
}

// locales/en.json
{
  "content": {
    "notTranslated": "This article is not yet translated to English.",
    "showingIn": "Showing in {locale}.",
    "notFound": "Article not found."
  }
}
```

## Pattern 11 : Opérateurs SQL dans `.where()`

La méthode `.where()` utilise des opérateurs SQL. Liste complète des opérateurs disponibles :

### Opérateurs de comparaison

| Opérateur | Usage | Exemple |
|-----------|-------|---------|
| `=` | Égalité exacte | `.where('category', '=', 'news')` |
| `>` | Supérieur à | `.where('date', '>', '2024-01-01')` |
| `<` | Inférieur à | `.where('views', '<', 1000)` |
| `<>` | Différent de | `.where('status', '<>', 'draft')` |
| `IN` | Dans une liste | `.where('pillar', 'IN', ['ai', 'engineering'])` |
| `BETWEEN` | Dans un intervalle | `.where('publishedAt', 'BETWEEN', ['2024-01-01', '2024-12-31'])` |
| `LIKE` | Pattern matching | `.where('path', 'LIKE', '/blog%')` |
| `IS NULL` | Valeur nulle | `.where('image', 'IS NULL', true)` |
| `IS NOT NULL` | Valeur non-nulle | `.where('featured', 'IS NOT NULL', true)` |

### Exemples de requêtes

```typescript
// Filtrage par date et catégorie
queryCollection(collection)
  .where('publishedAt', '>', '2024-01-01')
  .where('category', '=', 'tutorial')
  .all()

// Pattern matching avec LIKE (% = wildcard)
queryCollection(collection)
  .where('path', 'LIKE', '/blog%')  // Tous les articles dans /blog/*
  .all()

// Recherche dans les tags (tableau sérialisé)
queryCollection(collection)
  .where('tags', 'LIKE', '%"vue"%')  // Guillemets pour match exact
  .all()

// Vérification de valeur non-nulle
queryCollection(collection)
  .where('image', 'IS NOT NULL', true)
  .where('featured', '=', true)
  .all()
```

## Pattern 12 : Conditions complexes `.andWhere()` / `.orWhere()`

Pour des requêtes avec conditions imbriquées, utiliser `.andWhere()` et `.orWhere()` :

### Syntaxe avec callback

```typescript
// AND imbriqué
queryCollection(collection)
  .where('published', '=', true)
  .andWhere(q => q
    .where('date', '>', '2024-01-01')
    .where('category', '=', 'news')
  )
  .all()
// SQL: WHERE published = true AND (date > '2024-01-01' AND category = 'news')

// OR logique
queryCollection(collection)
  .where('featured', '=', true)
  .orWhere(q => q
    .where('pillar', '=', 'ai')
    .where('level', '=', 'advanced')
  )
  .all()
// SQL: WHERE featured = true OR (pillar = 'ai' AND level = 'advanced')
```

### Pattern pour filtrage multi-tags

```typescript
// Tous les articles qui ont TOUS les tags demandés
function filterByAllTags(tags: string[]) {
  let query = queryCollection(collection)
    .where('draft', '=', false)

  if (tags.length > 0) {
    query = query.andWhere(q => {
      tags.forEach(tag => {
        q.where('tags', 'LIKE', `%"${tag}"%`)
      })
      return q
    })
  }

  return query.all()
}
```

## Pattern 13 : Caveat `.order()` - Tri alphabétique

⚠️ **Attention** : `.order()` effectue un **tri alphabétique**, pas numérique.

### Le problème

```
# Fichiers avec préfixes numériques
content/
├── 1.intro.md
├── 2.basics.md
├── 10.advanced.md
├── 11.expert.md

# Résultat du tri alphabétique : 1, 10, 11, 2 ❌
```

### La solution : zero-padding

```
# Fichiers avec zero-padding
content/
├── 01.intro.md
├── 02.basics.md
├── 10.advanced.md
├── 11.expert.md

# Résultat du tri alphabétique : 01, 02, 10, 11 ✅
```

### Recommandation

Toujours utiliser le **zero-padding** pour les préfixes numériques dans les noms de fichiers :

| ❌ Mauvais | ✅ Correct |
|-----------|-----------|
| `1.intro.md` | `01.intro.md` |
| `9.chapter.md` | `09.chapter.md` |
| `10.finale.md` | `10.finale.md` |

## Pattern 14 : Pagination SSG complète

Pattern combinant `.skip()`, `.limit()` et `.count()` pour une pagination fonctionnelle en SSG.

### Composable de pagination

```typescript
// composables/usePaginatedContent.ts
import type { Collections } from '@nuxt/content'

interface PaginationOptions {
  perPage?: number
}

export function usePaginatedContent(options: PaginationOptions = {}) {
  const { locale } = useI18n()
  const route = useRoute()
  const perPage = options.perPage ?? 10

  const currentPage = computed(() =>
    parseInt(route.query.page as string) || 1
  )

  const collection = computed(() =>
    `articles_${locale.value}` as keyof Collections
  )

  // Requête paginée
  const { data: items, pending } = useAsyncData(
    `paginated-${locale.value}-${currentPage.value}`,
    () => queryCollection(collection.value)
      .where('draft', '=', false)
      .order('publishedAt', 'DESC')
      .skip((currentPage.value - 1) * perPage)
      .limit(perPage)
      .all(),
    { watch: [locale, currentPage] }
  )

  // Compte total (clé stable pour cache)
  const { data: totalCount } = useAsyncData(
    `count-${locale.value}`,
    () => queryCollection(collection.value)
      .where('draft', '=', false)
      .count(),
    { watch: [locale] }
  )

  const totalPages = computed(() =>
    Math.ceil((totalCount.value ?? 0) / perPage)
  )

  const hasNextPage = computed(() =>
    currentPage.value < totalPages.value
  )

  const hasPrevPage = computed(() =>
    currentPage.value > 1
  )

  return {
    items,
    pending,
    currentPage,
    totalPages,
    totalCount,
    hasNextPage,
    hasPrevPage,
    perPage
  }
}
```

### Composant Pagination

```vue
<!-- components/content/Pagination.vue -->
<script setup lang="ts">
interface Props {
  currentPage: number
  totalPages: number
  baseUrl?: string
}

const props = withDefaults(defineProps<Props>(), {
  baseUrl: ''
})

const pageNumbers = computed(() => {
  const pages: (number | '...')[] = []
  const current = props.currentPage
  const total = props.totalPages

  if (total <= 7) {
    return Array.from({ length: total }, (_, i) => i + 1)
  }

  pages.push(1)
  if (current > 3) pages.push('...')
  for (let i = Math.max(2, current - 1); i <= Math.min(total - 1, current + 1); i++) {
    pages.push(i)
  }
  if (current < total - 2) pages.push('...')
  pages.push(total)

  return pages
})
</script>

<template>
  <nav class="flex justify-center gap-2" aria-label="Pagination">
    <NuxtLink
      v-if="currentPage > 1"
      :to="`${baseUrl}?page=${currentPage - 1}`"
      class="px-3 py-2 rounded hover:bg-muted"
    >
      ←
    </NuxtLink>

    <template v-for="page in pageNumbers" :key="page">
      <span v-if="page === '...'" class="px-3 py-2">...</span>
      <NuxtLink
        v-else
        :to="`${baseUrl}?page=${page}`"
        class="px-3 py-2 rounded"
        :class="page === currentPage ? 'bg-primary text-primary-foreground' : 'hover:bg-muted'"
      >
        {{ page }}
      </NuxtLink>
    </template>

    <NuxtLink
      v-if="currentPage < totalPages"
      :to="`${baseUrl}?page=${currentPage + 1}`"
      class="px-3 py-2 rounded hover:bg-muted"
    >
      →
    </NuxtLink>
  </nav>
</template>
```

### Configuration prerender pour pagination

```typescript
// nuxt.config.ts
nitro: {
  prerender: {
    crawlLinks: true,
    // Expliciter les pages paginées si non découvertes automatiquement
    routes: [
      '/blog',
      '/blog?page=1',
      '/blog?page=2',
      '/blog?page=3'
    ]
  }
}
```

## Pattern 15 : `.select()` pour optimisation des listes

Pour les pages de listing, **ne pas charger le contenu complet** des articles. Utiliser `.select()` pour réduire drastiquement le payload.

### Pattern listing optimisé

```typescript
// pages/blog/index.vue
const { data: articles } = await useAsyncData(
  `blog-list-${locale.value}`,
  () => queryCollection(collection.value)
    .where('draft', '=', false)
    .order('publishedAt', 'DESC')
    // Sélectionner uniquement les champs nécessaires à l'affichage
    .select('title', 'path', 'description', 'publishedAt', 'pillar', 'image')
    .all()
)
```

### Comparaison payload

| Sans `.select()` | Avec `.select()` |
|------------------|------------------|
| ~50KB pour 10 articles | ~5KB pour 10 articles |
| Contenu MDC complet inclus | Métadonnées uniquement |
| Parsing MDC côté client | Pas de parsing nécessaire |

### Champs recommandés par contexte

| Contexte | Champs à sélectionner |
|----------|----------------------|
| **Liste blog** | `title`, `path`, `description`, `publishedAt`, `image` |
| **Sidebar récents** | `title`, `path`, `publishedAt` |
| **Articles liés** | `title`, `path`, `pillar`, `tags` |
| **Sitemap** | `path`, `publishedAt`, `updatedAt` |

## Pattern 16 : Anti-patterns à éviter

### ❌ Requête dans `onMounted()`

```typescript
// ❌ MAUVAIS - Exécute uniquement côté client, casse le SSG
onMounted(async () => {
  const data = await queryCollection('blog').all()
})

// ✅ BON - useAsyncData au top-level
const { data } = await useAsyncData('blog', () =>
  queryCollection('blog').all()
)
```

**Pourquoi** : `onMounted()` s'exécute uniquement côté client. Le prerendering SSG ne verra pas ces données.

### ❌ Requête sur champs imbriqués

```typescript
// ❌ MAUVAIS - SQL ne supporte pas la notation pointée
queryCollection('blog')
  .where('meta.published', '=', true)
  .all()

// ✅ BON - Aplatir le schema
// content.config.ts
schema: z.object({
  published: z.boolean()  // Au lieu de meta.published
})

// Requête
queryCollection('blog')
  .where('published', '=', true)
  .all()
```

**Pourquoi** : La base SQLite stocke les données à plat. Les objets imbriqués sont sérialisés en JSON.

### ❌ Clés `useAsyncData` dupliquées

```typescript
// ❌ MAUVAIS - Même clé pour requêtes différentes (cache collision)
await useAsyncData('blog', () =>
  queryCollection('blog').where('tag', '=', 'vue').all()
)
await useAsyncData('blog', () =>
  queryCollection('blog').where('tag', '=', 'react').all()
)

// ✅ BON - Clés uniques incluant les paramètres
await useAsyncData(`blog-tag-vue`, () =>
  queryCollection('blog').where('tag', '=', 'vue').all()
)
await useAsyncData(`blog-tag-react`, () =>
  queryCollection('blog').where('tag', '=', 'react').all()
)
```

**Pourquoi** : Nuxt utilise la clé pour la déduplication et le cache. Des clés identiques provoquent des collisions silencieuses.

### ❌ Préfixes numériques sans zero-padding

```
# ❌ MAUVAIS - Tri alphabétique incorrect
content/
├── 1.intro.md
├── 10.advanced.md
├── 2.basics.md

# ✅ BON - Zero-padding pour tri correct
content/
├── 01.intro.md
├── 02.basics.md
├── 10.advanced.md
```

**Pourquoi** : `.order()` utilise le tri alphabétique. `"10"` vient avant `"2"` alphabétiquement.

### ❌ Oublier `watch` pour les paramètres dynamiques

```typescript
// ❌ MAUVAIS - Ne se met pas à jour quand locale change
const { data } = await useAsyncData('articles', () =>
  queryCollection(`articles_${locale.value}`).all()
)

// ✅ BON - Re-fetch automatique
const { data } = await useAsyncData('articles', () =>
  queryCollection(`articles_${locale.value}`).all(),
  { watch: [locale] }
)
```

**Pourquoi** : Sans `watch`, la requête ne se ré-exécute pas quand la dépendance change.

---

## Content Validation & Testing

### Pattern 17 : Script de validation pré-build

**Problème critique** : Nuxt Content 3 ne fait **pas échouer le build** sur erreurs de validation frontmatter. Les champs invalides sont silencieusement omis.

**Solution** : Script autonome exécuté avant le build avec exit code 1 sur erreur.

```typescript
// scripts/validate-content.ts
import { z } from 'zod/v4'
import matter from 'gray-matter'
import { glob } from 'glob'
import { readFileSync } from 'fs'

// Importer les schemas depuis content.config.ts ou les redéfinir
const articleSchema = z.object({
  title: z.string().min(1),
  description: z.string().max(300),
  publishedAt: z.iso.date(),
  pillar: z.enum(['ai', 'engineering', 'ux']),
  category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
  level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),
  tags: z.array(z.string()).default([]),
  draft: z.boolean().default(false),
})

interface ValidationError {
  file: string
  issues: z.ZodIssue[]
}

async function validateCollection(pattern: string, schema: z.ZodSchema): Promise<ValidationError[]> {
  const files = await glob(pattern)
  const errors: ValidationError[] = []

  for (const file of files) {
    const content = readFileSync(file, 'utf-8')
    const { data: frontmatter } = matter(content)
    const result = schema.safeParse(frontmatter)

    if (!result.success) {
      errors.push({ file, issues: result.error.issues })
    }
  }

  return errors
}

async function main() {
  console.log('🔍 Validation du contenu...\n')

  const collections = [
    { pattern: 'content/fr/**/*.md', schema: articleSchema, name: 'articles_fr' },
    { pattern: 'content/en/**/*.md', schema: articleSchema, name: 'articles_en' },
  ]

  let hasErrors = false

  for (const { pattern, schema, name } of collections) {
    const errors = await validateCollection(pattern, schema)

    if (errors.length > 0) {
      hasErrors = true
      console.error(`❌ Collection "${name}" - ${errors.length} erreur(s):\n`)

      for (const { file, issues } of errors) {
        console.error(`  📄 ${file}`)
        for (const issue of issues) {
          console.error(`     → ${issue.path.join('.')}: ${issue.message}`)
        }
        console.error('')
      }
    } else {
      console.log(`✅ Collection "${name}" - valide`)
    }
  }

  if (hasErrors) {
    console.error('\n💥 Validation échouée. Corrigez les erreurs avant le build.')
    process.exit(1)
  }

  console.log('\n✨ Tout le contenu est valide!')
}

main().catch((err) => {
  console.error('Erreur inattendue:', err)
  process.exit(1)
})
```

**Scripts npm :**

```json
{
  "scripts": {
    "validate:content": "tsx scripts/validate-content.ts",
    "prebuild": "pnpm validate:content",
    "build": "nuxt build --preset=cloudflare_pages"
  }
}
```

**Installation :**

```bash
pnpm add -D tsx gray-matter glob
```

### Pattern 18 : Configuration nuxt-link-checker

Le module `nuxt-link-checker` (inclus dans `@nuxtjs/seo`) doit être configuré explicitement pour faire échouer le build :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  linkChecker: {
    // Fail le build sur liens cassés
    failOnError: true,

    // Activer pendant le build SSG
    runOnBuild: true,

    // Générer des rapports
    report: {
      html: true,
      markdown: true,
    },

    // Exclure les patterns problématiques
    exclude: [
      '/api/**',           // Routes API (pas de HTML)
      '/__nuxt_island/**', // Internals Nuxt
    ],

    // Timeout pour liens externes (ms)
    fetchTimeout: 5000,

    // Retries sur erreurs réseau
    retries: 2,
  },

  nitro: {
    prerender: {
      // REQUIS : découvre les liens à vérifier
      crawlLinks: true,
      routes: ['/'],
    },
  },
})
```

| Option | Défaut | Recommandé | Effet |
|--------|--------|------------|-------|
| `failOnError` | `false` | `true` | Bloque le build sur lien cassé |
| `runOnBuild` | `false` | `true` | Vérifie pendant `nuxt generate` |
| `fetchTimeout` | `10000` | `5000` | Timeout liens externes |

### Pattern 19 : lychee en CI pour liens externes

Complément à nuxt-link-checker pour une vérification exhaustive post-build des liens externes :

```yaml
# .github/workflows/deploy.yml
jobs:
  check-links:
    needs: [build]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/download-artifact@v4
        with:
          name: build-output
          path: .output/public

      - name: Link Checker
        uses: lycheeverse/lychee-action@v2
        with:
          args: >-
            --verbose
            --cache
            --max-cache-age 1d
            --exclude linkedin\.com
            --exclude twitter\.com
            --exclude x\.com
            --timeout 30
            --max-retries 3
            '.output/public/**/*.html'
          fail: true
```

**Avantages lychee vs nuxt-link-checker seul :**

| Aspect | nuxt-link-checker | lychee |
|--------|-------------------|--------|
| **Vitesse** | Moyen | Très rapide (Rust) |
| **Cache** | Non | Oui (1 jour) |
| **Parallélisme** | Limité | Massif |
| **Liens externes** | Timeout fréquents | Robuste |

**Exclusions recommandées** (rate limiting agressif) :
- `linkedin.com` - bloque les bots
- `twitter.com` / `x.com` - bloque les bots
- `github.com/*/edit/*` - liens dynamiques

### Pattern 20 : markdownlint-cli2 pour MDC

**Aucun linter MDC natif n'existe**. Adapter markdownlint avec le parser `markdown-it-mdc` :

**Installation :**

```bash
pnpm add -D markdownlint-cli2 markdown-it-mdc
```

**Configuration `.markdownlint-cli2.jsonc` :**

```jsonc
{
  // Parser MDC pour comprendre la syntaxe ::component{}
  "markdownItPlugins": [["markdown-it-mdc"]],

  // Pattern frontmatter YAML
  "frontMatter": "^---\\s*$[\\s\\S]*?^---\\s*$",

  "config": {
    // Désactiver les règles incompatibles MDC
    "MD033": false,  // Allow inline HTML (composants Vue)
    "MD041": false,  // First line heading (frontmatter interfère)

    // Règles adaptées
    "MD013": {
      "line_length": 120,
      "code_blocks": false,
      "tables": false
    },

    // Headings
    "MD022": { "lines_above": 1, "lines_below": 1 },
    "MD024": { "siblings_only": true },

    // Listes
    "MD004": { "style": "dash" },
    "MD007": { "indent": 2 }
  },

  // Fichiers à linter
  "globs": ["content/**/*.{md,mdc}"],

  // Ignorer certains fichiers
  "ignores": ["content/**/drafts/**"]
}
```

**Script npm :**

```json
{
  "scripts": {
    "lint:md": "markdownlint-cli2 'content/**/*.{md,mdc}'",
    "lint:md:fix": "markdownlint-cli2 --fix 'content/**/*.{md,mdc}'"
  }
}
```

**Règles désactivées expliquées :**

| Règle | Raison désactivation |
|-------|---------------------|
| `MD033` | Composants Vue inline `<Badge>`, `<Alert>` |
| `MD041` | Frontmatter YAML avant le premier heading |

### Pattern 21 : CI séparé validate → build

Structure de jobs pour fail-fast sans gaspiller le build :

```yaml
# .github/workflows/deploy.yml
jobs:
  # ════════════════════════════════════════════════
  # VALIDATION CONTENU - Fail fast
  # ════════════════════════════════════════════════
  validate-content:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: '10'
          cache: true

      - uses: actions/setup-node@v4
        with:
          node-version: '22'

      - run: pnpm install --frozen-lockfile

      # Validation frontmatter Zod
      - run: pnpm validate:content

      # Linting markdown MDC
      - run: pnpm lint:md

  # ════════════════════════════════════════════════
  # LINT & TYPECHECK - Parallèle
  # ════════════════════════════════════════════════
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: '10', cache: true }
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: '10', cache: true }
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck

  # ════════════════════════════════════════════════
  # BUILD - Après validations
  # ════════════════════════════════════════════════
  build:
    needs: [validate-content, lint, typecheck]
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: '10', cache: true }
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
      # ... upload artifact, deploy
```

**Avantages :**

| Sans séparation | Avec séparation |
|-----------------|-----------------|
| Erreur contenu → build complet gaspillé | Erreur contenu → fail en ~30s |
| Feedback lent | Feedback rapide |
| CI coûteux | CI économique |

### Pattern 22 : experimental.nativeSqlite (Node 22)

Éviter les erreurs de compilation `better-sqlite3` en CI avec le module SQLite natif de Node 22 :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  content: {
    experimental: {
      // Utilise le module sqlite natif de Node.js 22+
      // Évite la compilation native de better-sqlite3
      nativeSqlite: true,
    },
  },
})
```

**Quand l'activer :**

| Contexte | Activer ? | Raison |
|----------|-----------|--------|
| Node 22 LTS | ✅ Oui | Module sqlite intégré |
| Node 20 | ❌ Non | Pas de module sqlite natif |
| CI GitHub Actions | ✅ Recommandé | Évite compilation native |
| Docker Alpine | ✅ Recommandé | Évite dépendances build |

**Note** : Cette option est expérimentale (décembre 2025). Tester en local avant d'activer en production.
