# Validation Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Schema Validator** | Zod 4 ou Valibot v1 | Standard Schema natif; Valibot ~1KB gzip / ~2KB min, Zod ~5KB gzip / ~10KB min, zod/mini ~2KB gzip / ~5KB min (réduction 64%) |
| **Import Source** | `import { z } from 'zod/v4'` | ⚠️ Utiliser `zod/v4` pour l'API Zod 4. `@zod/mini` (package npm) **déplacé** vers `zod/mini` (import path) depuis mai 2025 |
| **Validation Scope** | Build time | Erreurs détectées au build via Content 3 collections |
| **Type Inference** | `z.infer<typeof schema>` | Types TypeScript générés automatiquement |
| **Sitemap Integration** | `asSitemapCollection()` | Wrapper requis pour @nuxtjs/sitemap v7.5+ avec Content v3 |
| **SEO Unified** | `asSeoCollection()` | Combine sitemap + schema.org + OG Image + Robots via `@nuxtjs/seo/content` |

## Top-Level Validators Zod 4

Zod 4 promeut les validateurs de format au niveau top-level pour un meilleur tree-shaking et des performances accrues :

| Zod 3 (déprécié) | Zod 4 (recommandé) | Notes |
|------------------|-------------------|-------|
| `z.string().email()` | `z.email()` | Supporte patterns : `z.email({ pattern: z.regexes.unicodeEmail })` |
| `z.string().url()` | `z.url()` | Validation WHATWG URL |
| `z.string().uuid()` | `z.uuid()` | ⚠️ Strict RFC 9562/4122 (variant bits) |
| `z.string().uuid()` | `z.guid()` | Validation permissive (UUIDs legacy) |
| `z.date()` | `z.iso.date()` | Format `YYYY-MM-DD` (string, compatible JSON Schema) |
| `z.date()` | `z.iso.datetime()` | Format ISO 8601 complet avec options |
| `z.string().ip()` | `z.ipv4()` / `z.ipv6()` | `z.string().ip()` supprimé en Zod 4 |

**Options `z.iso.datetime()` :**
```typescript
z.iso.datetime({ offset: true })     // Autorise timezone offsets
z.iso.datetime({ local: true })      // Permet datetimes sans timezone
z.iso.datetime({ precision: 3 })     // Contraint à millisecondes
```

**⚠️ Migration UUID importante :** La validation UUID Zod 4 est stricte (RFC 9562/4122). Les UUIDs qui passaient en Zod 3 peuvent échouer. Utiliser `z.guid()` pour une validation permissive.

## Breaking Changes Critiques Zod 4

### `.default()` dans `.optional()` - Comportement inversé

**⚠️ CRITIQUE** : En Zod 4, `.default()` s'applique maintenant DANS les champs `.optional()` :

```typescript
const schema = z.object({
  draft: z.string().default("untitled").optional()
})

schema.parse({})
// Zod 3: {}                        ← undefined préservé
// Zod 4: { draft: "untitled" }     ← default appliqué !
```

**Impact frontmatter** : Les champs optionnels avec defaults auront toujours une valeur en Zod 4.

### `.prefault()` - Defaults pré-transform

Nouveau en Zod 4, applique le default AVANT les transforms (comportement Zod 3) :

```typescript
const titleSchema = z.string()
  .transform(val => val.toUpperCase())
  .prefault('untitled')  // Default sur le type INPUT

titleSchema.parse(undefined) // => "UNTITLED"
```

| Méthode | Appliqué | Type cible |
|---------|----------|------------|
| `.default(x)` | Après transform | Output type |
| `.prefault(x)` | Avant transform | Input type |

### `.nullish()` pour YAML null

Support des valeurs `null` explicites en YAML frontmatter :

```typescript
// Accepte: undefined, null, ou string URL
coverImage: z.string().url().nullish().default(null)

// Hiérarchie complète
| Pattern | Accepte | Output |
|---------|---------|--------|
| `.optional()` | `T \| undefined` | `T \| undefined` |
| `.nullable()` | `T \| null` | `T \| null` |
| `.nullish()` | `T \| null \| undefined` | `T \| null \| undefined` |
| `.nullish().default(x)` | `T \| null \| undefined` | `T` |
```

### Enum `.extract()` et `.exclude()`

Créer des sous-ensembles d'enums sans redéfinition :

```typescript
const AllCategories = z.enum(['tutorial', 'news', 'opinion', 'review'])
const BlogOnly = AllCategories.extract(['tutorial', 'opinion'])  // Sous-ensemble
const NonNews = AllCategories.exclude(['news'])                   // Exclusion

// Accès programmatique aux valeurs (remplace .Enum et .Values)
AllCategories.enum // => { tutorial: "tutorial", news: "news", ... }
```

### Autres breaking changes API

| Zod 3 | Zod 4 | Notes |
|-------|-------|-------|
| `z.record(z.string())` | `z.record(z.string(), z.string())` | **Deux arguments requis** |
| `.merge(otherSchema)` | `.extend(otherSchema.shape)` | `.merge()` déprécié |
| `.format()` / `.flatten()` | `z.treeifyError()` | API erreurs changée |
| `.nonempty()` → `[T, ...T[]]` | `.nonempty()` → `T[]` | Type inference simplifié |
| `z.nativeEnum(TsEnum)` | `z.enum(TsEnum)` | Enums TS natifs supportés |

## asSeoCollection() vs asSitemapCollection()

| Wrapper | Fonctionnalités | Quand utiliser |
|---------|-----------------|----------------|
| `asSitemapCollection()` | Sitemap uniquement | Blog simple sans besoins SEO avancés |
| `asSeoCollection()` | Sitemap + Schema.org + OG Image + Robots | **Recommandé** - SEO complet |

**Configuration asSeoCollection() :**

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { asSeoCollection } from '@nuxtjs/seo/content'
import { z } from 'zod/v4'

export default defineContentConfig({
  collections: {
    articles_fr: defineCollection(
      asSeoCollection({  // ← Wrapper SEO complet
        type: 'page',
        source: { include: 'fr/**/*.md', prefix: '/blog' },
        schema: articleSchema,
        indexes: articleIndexes,
      })
    ),
  }
})
```

**Clés frontmatter SEO activées par asSeoCollection() :**

```yaml
---
title: "Mon article"
description: "Description SEO (max 160 caractères)"
image: "/images/cover.jpg"

# OG Image dynamique (optionnel)
ogImage:
  component: BlogOgImage
  props:
    title: "Mon article"
    readingMins: 5

# Schema.org structuré (optionnel)
schemaOrg:
  - "@type": "BlogPosting"
    headline: "Mon article"
    author:
      "@type": "Person"
      name: "Sébastien C."
    datePublished: "2025-01-15"

# Contrôle sitemap (optionnel)
sitemap:
  lastmod: 2025-01-16
  priority: 0.8

# Contrôle robots (optionnel)
robots:
  noindex: false
  nofollow: false
---
```

## Schema SEO imbriqué (optionnel)

Pour un contrôle granulaire du SEO dans le frontmatter, définir un schema dédié :

```typescript
// content.config.ts
const seoSchema = z.object({
  title: z.string().max(60).optional(),      // Titre OG/Twitter
  description: z.string().max(160).optional(), // Meta description
  image: z.string().optional(),               // OG image path
  canonical: z.string().url().optional(),     // URL canonique
  noIndex: z.boolean().default(false),        // Bloquer indexation
})

const articleSchema = z.object({
  title: z.string().min(1).max(100),
  description: z.string().max(300).optional(),
  // ... autres champs
  seo: seoSchema.optional(),  // ← SEO imbriqué
})
```

**Usage dans le frontmatter :**

```yaml
---
title: "Titre de l'article"
description: "Description générale"
seo:
  title: "Titre SEO optimisé (60 car.)"  # Override pour les moteurs
  description: "Meta description (160 car.)"
  canonical: "https://sebc.dev/blog/article-original"
  noIndex: false
---
```
