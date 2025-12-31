# Content Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **File Organization** | By language (`content/fr/`, `content/en/`) | Natural fit with @nuxtjs/i18n, clear separation |
| **Collections** | Separate per language (`articles_fr`, `articles_en`) | Performance D1, typage précis, code auto-documenté |
| **Frontmatter Constants** | English (`ai`, `tutorial`, `beginner`) | Consistency, i18n-agnostic |
| **Reading Time** | Auto-calculated (200 words/min) | Less maintenance, always accurate |

**Frontmatter Schema:**

```yaml
title: string
description: string
slug: string
pillar: 'ai' | 'engineering' | 'ux'
category: 'news' | 'tutorial' | 'deep-dive' | 'case-study' | 'retrospective'
level: 'all' | 'beginner' | 'intermediate' | 'advanced'
tags: string[]
publishedAt: date
updatedAt: date
image: string
draft: boolean
translationKey: string          # Optionnel - Lie les traductions FR/EN d'un même article
```

**Champ `translationKey` (i18n Content) :**

Le champ `translationKey` permet de lier les versions française et anglaise d'un même article pour :
- Afficher le switcher de langue sur les pages article
- Générer les balises `hreflang` alternate correctes
- Proposer "Lire en français/anglais" automatiquement

```yaml
# content/fr/guide-nuxt-4.md
---
title: "Guide complet Nuxt 4"
translationKey: "nuxt-4-complete-guide"
---

# content/en/nuxt-4-guide.md
---
title: "Complete Nuxt 4 Guide"
translationKey: "nuxt-4-complete-guide"  # Même clé = même article
---
```

**Requête pour trouver la traduction :**

```typescript
// Trouver la version EN d'un article FR
const { data: translation } = await useAsyncData(
  `translation-${article.translationKey}`,
  () => queryCollection('articles_en')
    .where('translationKey', '=', article.translationKey)
    .first()
)
```

**Configuration Collections (content.config.ts) :**

```typescript
import { defineContentConfig, defineCollection, asSitemapCollection } from '@nuxt/content'
import { z } from 'zod/v4'  // ⚠️ Zod 4 : utiliser 'zod/v4' pour l'API moderne

// Schema partagé entre les collections
const articleSchema = z.object({
  title: z.string(),
  description: z.string(),
  slug: z.string(),
  pillar: z.enum(['ai', 'engineering', 'ux']),
  category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
  level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),
  tags: z.array(z.string()).default([]),
  publishedAt: z.iso.date(),           // ⚠️ Zod 4 : z.iso.date() pour format YYYY-MM-DD (compatible JSON Schema)
  updatedAt: z.iso.date().optional(),  // ⚠️ z.date() n'a PAS d'équivalent JSON Schema - utiliser z.iso.date()
  image: z.string().optional(),
  draft: z.boolean().default(false),
  translationKey: z.string().optional(),  // Lie les traductions FR/EN d'un même article
})

// Indexes partagés
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
      asSitemapCollection({
        type: 'page',
        source: { include: 'fr/**/*.md', prefix: '/blog' },
        schema: articleSchema,
        indexes: articleIndexes,
      })
    ),
    articles_en: defineCollection(
      asSitemapCollection({
        type: 'page',
        source: { include: 'en/**/*.md', prefix: '/blog' },
        schema: articleSchema,
        indexes: articleIndexes,
      })
    ),
  }
})
```

**Avantages des collections séparées :**

| Aspect | Bénéfice |
|--------|----------|
| **Performance D1** | Requêtes sur table plus petite = moins de rows_read |
| **Indexes isolés** | Index `idx_published` optimisé par langue |
| **Typage TypeScript** | `Collections['articles_fr']` vs filtrage runtime |
| **Code auto-documenté** | `queryCollection('articles_fr')` - intention claire |
| **Évolutivité** | Ajouter une 3e langue = nouvelle collection isolée |
