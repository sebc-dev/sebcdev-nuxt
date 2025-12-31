# Pattern 6 : Property Editors pour Nuxt Studio

La fonction `property()` de `@nuxt/content` encapsule un schema Zod et expose la méthode `.editor()` pour configurer l'interface visuelle dans Nuxt Studio.

## Configuration des inputs visuels

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

## Types d'inputs disponibles

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

## Héritage de props de composants Vue

Pour un champ qui correspond aux props d'un composant Vue :

```typescript
// content.config.ts
const articleSchema = z.object({
  // Hérite automatiquement des props de HeroSection.vue
  hero: property(z.object({})).inherit('components/HeroSection.vue'),
})
```

## Exemple complet avec tous les types

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
