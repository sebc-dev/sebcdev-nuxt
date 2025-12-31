# Process Patterns

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
- Validateur: Zod 4 (~5KB gzip / ~10KB min) ou `zod/mini` (~2KB gzip / ~5KB min, réduction 64%) ou Valibot v1.0 (~1KB gzip / ~2KB min tree-shaken)
- Import: `import { z } from 'zod'` ou `import * as z from 'zod/mini'` (⚠️ `@zod/mini` package npm **déplacé** vers `zod/mini` import path depuis mai 2025. `import { z } from '@nuxt/content'` **déprécié**)
- Validation au build time via Content 3 collections

```typescript
// content.config.ts - Configuration avec @nuxtjs/seo
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { asSeoCollection } from '@nuxtjs/seo/content'
import { z } from 'zod'  // ✅ Import direct depuis zod (pas @nuxt/content)

export default defineContentConfig({
  collections: {
    articles: defineCollection(
      asSeoCollection({  // ✅ Combine sitemap + schema.org + robots + og-image
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
    )
  }
})
```

**Fonctionnalités activées par asSeoCollection() :**
- `asSitemapCollection` - Génération sitemap XML avec hreflang
- `asSchemaOrgCollection` - JSON-LD Schema.org automatique
- `asRobotsCollection` - Contrôle robots via frontmatter
- `asOgImageCollection` - Génération images OG au build

## Sitemap i18n avec hreflang automatique

### Configuration de base

@nuxtjs/sitemap s'intègre automatiquement avec @nuxtjs/i18n pour générer des sitemaps par locale avec annotations hreflang :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/i18n', '@nuxtjs/sitemap'],

  site: {
    url: 'https://sebc.dev'
  },

  // Génère automatiquement :
  // - /sitemap_index.xml
  // - /en-sitemap.xml
  // - /fr-sitemap.xml
})
```

### Routes dynamiques avec _i18nTransform

Pour les routes dynamiques (articles de blog), créer une source sitemap avec `_i18nTransform: true` :

```typescript
// server/api/__sitemap__/urls.ts
import type { Collections } from '@nuxt/content'

export default defineSitemapEventHandler(async () => {
  // Récupérer les articles des deux collections
  const [articlesFr, articlesEn] = await Promise.all([
    queryCollection('articles_fr' as keyof Collections)
      .where('draft', '=', false)
      .select('path', 'updatedAt', 'publishedAt')
      .all(),
    queryCollection('articles_en' as keyof Collections)
      .where('draft', '=', false)
      .select('path', 'updatedAt', 'publishedAt')
      .all(),
  ])

  const allArticles = [...articlesFr, ...articlesEn]

  return allArticles.map(article => ({
    loc: article.path,
    lastmod: article.updatedAt || article.publishedAt,
    _i18nTransform: true  // ← Génère automatiquement les variantes hreflang
  }))
})
```

**Effet de `_i18nTransform: true` :**

Pour chaque URL, génère automatiquement dans le sitemap XML :

```xml
<url>
  <loc>https://sebc.dev/blog/my-article</loc>
  <lastmod>2025-01-15</lastmod>
  <xhtml:link rel="alternate" hreflang="en" href="https://sebc.dev/blog/my-article"/>
  <xhtml:link rel="alternate" hreflang="fr" href="https://sebc.dev/fr/blog/my-article"/>
  <xhtml:link rel="alternate" hreflang="x-default" href="https://sebc.dev/blog/my-article"/>
</url>
```

### Pattern articles avec slugs traduits

Si les articles ont des slugs différents par langue (ex: `/blog/getting-started` vs `/fr/blog/demarrage`), utiliser un UID commun :

```typescript
// server/api/__sitemap__/urls.ts
export default defineSitemapEventHandler(async () => {
  // Récupérer articles avec leur UID de traduction
  const articles = await queryCollection('articles_en')
    .select('path', 'updatedAt', 'translationUid')
    .all()

  return articles.map(article => ({
    loc: article.path,
    lastmod: article.updatedAt,
    // Désactiver _i18nTransform si slugs différents
    // Gérer manuellement les alternates
    alternatives: [
      { hreflang: 'en', href: article.path },
      { hreflang: 'fr', href: `/fr/blog/${article.translationUid}` },
      { hreflang: 'x-default', href: article.path },
    ]
  }))
})
```

---

## Definition of Done (DoD)

**Checklist qualité pour chaque story/tâche :**

| Critère | Obligatoire | Vérification |
|---------|:-----------:|--------------|
| **Code** | ✅ | Pas de violations ESLint critiques |
| **Types** | ✅ | `pnpm typecheck` passe sans erreur |
| **Tests** | ✅ | Coverage ≥ 80% sur le code modifié |
| **Build** | ✅ | `pnpm build` réussit |
| **A11y** | ✅ | Composants UI testés avec vitest-axe |
| **Review** | ✅ | Code review effectuée (ou pair programming) |
| **Docs** | ⚪ | Documentation mise à jour si API publique |
| **i18n** | ⚪ | Traductions ajoutées si nouveau texte UI |

**Critères techniques automatisés (CI) :**

```yaml
# Résumé des quality gates
- pnpm lint          # Aucune erreur ESLint
- pnpm typecheck     # TypeScript strict
- pnpm test          # Tests passent
- Coverage ≥ 80%     # Seuil de couverture
- pnpm build         # Build SSG réussit
```

**Règle :** Une story n'est "Done" que si **tous les critères obligatoires** sont validés. Les critères optionnels (⚪) s'appliquent selon le contexte.
