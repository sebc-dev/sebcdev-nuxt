# Implementation Patterns & Consistency Rules

## Pattern Categories Defined

**15 points de conflit potentiels** identifiés et résolus pour garantir la cohérence entre agents IA.

## Naming Patterns

**File Naming Conventions:**

| Type | Convention | Exemple |
|------|------------|---------|
| Composants Vue | PascalCase | `ArticleCard.vue` |
| Pages Nuxt | kebab-case | `[slug].vue` |
| Composables | camelCase + `use` | `useReadingTime.ts` |
| Utilitaires | camelCase | `formatDate.ts` |
| Types | PascalCase | `Article.ts` |
| Contenu MDC | kebab-case | `mon-premier-article.md` |

**i18n Key Naming:**
- Nested objects structure
- Groupé par feature/section

```typescript
// ✅ Correct
{
  header: {
    nav: { home: "Accueil", articles: "Articles" }
  },
  article: {
    readingTime: "{min} min de lecture"
  }
}
```

**CSS/Tailwind Naming:**
- Classes custom: kebab-case (`article-card`)
- CSS variables: `--color-primary`, `--spacing-lg`
- Animations: kebab-case (`fade-in`, `slide-up`)

## Structure Patterns

**Test Organization:**
- Co-located avec le code source
- `Component.vue` → `Component.test.ts` (même dossier)

**Composables Organization:**
- Un fichier par composable
- Flat structure dans `app/composables/`

```
app/composables/
├── useReadingTime.ts
├── useArticleFilters.ts
├── useTableOfContents.ts
└── useSeoMeta.ts
```

**TypeScript Types:**
- Groupés par domaine dans `types/`

```
types/
├── article.ts      # Article, ArticleMeta, Pillar, Category
├── search.ts       # SearchFilters, SearchResult
└── navigation.ts   # NavItem, Breadcrumb
```

## Format Patterns

**Date Formatting:**
- Frontmatter: ISO 8601 (`2025-01-15`)
- Display: `Intl.DateTimeFormat` natif (zéro dépendance)

```typescript
// ✅ Correct
const formatDate = (date: Date, locale: string) =>
  new Intl.DateTimeFormat(locale, {
    day: 'numeric',
    month: 'long',
    year: 'numeric'
  }).format(date)
```

**Error Format (server routes):**

```typescript
// Format simple
{ error: string, statusCode: number }

// Exemple
{ error: "Article not found", statusCode: 404 }
```

**JSON Conventions:**
- camelCase pour toutes les propriétés
- Cohérent avec TypeScript/JavaScript

## Communication Patterns

**Vue Events:**
- kebab-case: `@article-selected`, `@filter-changed`

```vue
<!-- ✅ Correct -->
<ArticleCard @article-selected="handleSelect" />

<!-- ❌ Incorrect -->
<ArticleCard @articleSelected="handleSelect" />
```

**Component Props:**
- Inline `defineProps<{}>` pour composants simples
- Interface exportée si props réutilisées ailleurs

```typescript
// Simple component
defineProps<{
  article: Article
  showExcerpt?: boolean
}>()

// Reusable props
export interface ArticleCardProps {
  article: Article
  variant: 'card' | 'hero' | 'list'
}
```

**Loading States:**
- Naming: `isLoading`, `isPending`
- UI: `<Skeleton />` de shadcn-vue
- Pattern: destructure depuis `useAsyncData`

## Process Patterns

**Error Handling Hierarchy:**

| Niveau | Approche | Exemple |
|--------|----------|---------|
| Composant | `try/catch` + état local | Form validation |
| Page | `useAsyncData` error | Article fetch |
| Global | `error.vue` | 404, 500 |

**Logging:**
- Console native (`console.log`, `console.error`)
- Pas de wrapper custom pour MVP

**Data Validation:**
- Validateur: Zod v4 (Standard Schema compatible)
- Import direct depuis `zod` (re-export `@nuxt/content` déprécié)
- Validation au build time via Content 3 collections

```typescript
// content.config.ts
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { z } from 'zod'  // ✅ Import direct - ne pas utiliser @nuxt/content

export default defineContentConfig({
  collections: {
    articles: defineCollection({
      type: 'page',
      source: '**/*.md',
      schema: z.object({
        title: z.string(),
        description: z.string(),
        slug: z.string(),
        pillar: z.enum(['ai', 'engineering', 'ux']),
        category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
        level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),
        tags: z.array(z.string()),
        publishedAt: z.coerce.date(),
        updatedAt: z.coerce.date().optional(),
        image: z.string().optional(),
        draft: z.boolean().default(false),
      })
    })
  }
})
```

**Intégration Schema.org avec Content 3:**
```typescript
// content.config.ts
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { asSchemaOrgCollection } from 'nuxt-schema-org/content'
import { z } from 'zod'

export default defineContentConfig({
  collections: {
    articles: defineCollection(
      asSchemaOrgCollection({
        type: 'page',
        source: '**/*.md',
        schema: z.object({
          title: z.string(),
          description: z.string(),
          pillar: z.enum(['ai', 'engineering', 'ux']),
          category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
          level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),
          tags: z.array(z.string()),
          publishedAt: z.date(),
          updatedAt: z.date().optional(),
          image: z.string().optional(),
          draft: z.boolean().default(false),
        })
      })
    )
  }
})
```

## Enforcement Guidelines

**All AI Agents MUST:**

1. Suivre les conventions de nommage définies ci-dessus
2. Placer les tests à côté des fichiers sources
3. Utiliser `Intl.DateTimeFormat` pour le formatage de dates
4. Émettre les events Vue en kebab-case
5. Valider le contenu via Content 3 schema

**Anti-Patterns à Éviter:**
```typescript
// ❌ snake_case dans JSON
{ published_at: "2025-01-15" }

// ❌ Tests dans dossier séparé
tests/components/ArticleCard.test.ts

// ❌ Date formatting avec lib externe
import { format } from 'date-fns'

// ❌ Events en camelCase
emit('articleSelected')

// ❌ Import zod depuis @nuxt/content (déprécié)
import { z } from '@nuxt/content'
```
