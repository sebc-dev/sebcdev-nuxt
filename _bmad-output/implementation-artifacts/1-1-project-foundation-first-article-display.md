# Story 1.1: Project Foundation & First Article Display

Status: ready-for-dev

## Story

As a visiteur,
I want to see a blog article displayed on a dedicated page,
So that I can read technical content.

## Acceptance Criteria

1. **AC1 - Projet Nuxt 4 initialisé** : Le projet Nuxt 4.2.2 est initialisé avec la structure `app/` et pnpm 10.26+
2. **AC2 - Navigation article** : Je peux naviguer vers `/articles/test-article` et voir l'article
3. **AC3 - Rendu Markdown** : L'article est affiché avec le contenu Markdown rendu (titres, paragraphes, listes, liens)
4. **AC4 - Images optimisées** : Les images ont un attribut alt et sont lazy-loaded
5. **AC5 - Performance** : Le temps de chargement initial est < 2s (LCP)
6. **AC6 - Configuration D1** : Cloudflare D1 est configuré correctement dans wrangler.toml

## Tasks / Subtasks

- [ ] **Task 1: Initialisation projet Nuxt 4** (AC: #1)
  - [ ] 1.1 Créer le projet avec `pnpm dlx nuxi@latest init sebc-dev`
  - [ ] 1.2 Installer les dépendances core : `@nuxt/content`, `@nuxt/image`
  - [ ] 1.3 Installer TailwindCSS via `@tailwindcss/vite` (v4.1.18)
  - [ ] 1.4 Créer le fichier `.nvmrc` avec version `22`
  - [ ] 1.5 Configurer pnpm-workspace.yaml pour les lifecycle scripts

- [ ] **Task 2: Configuration nuxt.config.ts** (AC: #1, #5)
  - [ ] 2.1 Configurer les modules dans l'ordre correct : `@nuxt/image`, `@nuxt/content`
  - [ ] 2.2 Configurer `features.inlineStyles: true` pour réduire CLS
  - [ ] 2.3 Configurer `nitro.preset: 'cloudflare_pages'`
  - [ ] 2.4 Ajouter `css: ['~/assets/css/main.css']`
  - [ ] 2.5 Configurer Vite avec `@tailwindcss/vite` plugin

- [ ] **Task 3: Configuration Cloudflare D1** (AC: #6)
  - [ ] 3.1 Créer la base D1 avec `wrangler d1 create content-db`
  - [ ] 3.2 Créer wrangler.toml avec le database_id
  - [ ] 3.3 Configurer content.database dans nuxt.config.ts (type: 'd1', bindingName: 'DB')

- [ ] **Task 4: Structure app/ et content/** (AC: #1, #2)
  - [ ] 4.1 Créer `app/assets/css/main.css` avec imports TailwindCSS
  - [ ] 4.2 Créer le dossier `app/pages/articles/`
  - [ ] 4.3 Créer `app/pages/articles/[slug].vue` - page article dynamique
  - [ ] 4.4 Créer `content/fr/articles/` et `content/en/articles/`
  - [ ] 4.5 Créer `shared/types/article.ts` pour les types partagés

- [ ] **Task 5: Configuration Nuxt Content 3** (AC: #2, #3)
  - [ ] 5.1 Créer `content.config.ts` avec collections articles_fr et articles_en
  - [ ] 5.2 Définir le schema Zod pour le frontmatter (title, slug, pillar, etc.)
  - [ ] 5.3 Configurer les indexes D1 pour optimisation

- [ ] **Task 6: Article de test** (AC: #2, #3, #4)
  - [ ] 6.1 Créer `content/fr/articles/test-article.md` avec frontmatter complet
  - [ ] 6.2 Inclure des exemples de titres H1, H2, H3
  - [ ] 6.3 Inclure des paragraphes, listes, liens
  - [ ] 6.4 Inclure au moins une image avec alt text

- [ ] **Task 7: Page article [slug].vue** (AC: #2, #3, #4, #5)
  - [ ] 7.1 Implémenter `queryCollection()` pour récupérer l'article par slug
  - [ ] 7.2 Utiliser `<ContentRenderer>` pour afficher le contenu MDC
  - [ ] 7.3 Utiliser `<NuxtImg>` pour les images (lazy loading natif)
  - [ ] 7.4 Gérer le cas 404 si article non trouvé

- [ ] **Task 8: Validation et test** (AC: #1-6)
  - [ ] 8.1 Lancer `pnpm dev` et vérifier que le site démarre
  - [ ] 8.2 Naviguer vers `/articles/test-article` et vérifier le rendu
  - [ ] 8.3 Vérifier le lazy loading des images
  - [ ] 8.4 Vérifier la console pour erreurs

## Dev Notes

### Critical Architecture Requirements

**OBLIGATOIRE - Cloudflare D1:**
- Le runtime Cloudflare Worker n'a PAS accès au filesystem
- Sans D1 configuré → Erreur 500 au runtime
- Créer la DB avec `wrangler d1 create content-db` et copier l'ID dans wrangler.toml

**Ordre des modules dans nuxt.config.ts:**
```typescript
modules: [
  '@nuxt/image',
  '@nuxt/content',  // Modules SEO ajoutés dans stories suivantes
]
```

**Configuration wrangler.toml (CRITIQUE):**
```toml
name = "sebc-dev"
compatibility_date = "2024-09-19"

[[d1_databases]]
binding = "DB"
database_name = "content-db"
database_id = "VOTRE_DATABASE_ID"  # Généré par wrangler d1 create
```

### Project Structure Notes

Structure Nuxt 4 avec `app/` comme srcDir:

```
sebc-dev/
├── app/
│   ├── assets/css/main.css
│   ├── pages/articles/[slug].vue
│   └── ...
├── content/
│   ├── fr/articles/
│   └── en/articles/
├── shared/types/article.ts
├── server/
├── public/
├── content.config.ts
├── nuxt.config.ts
├── wrangler.toml
└── pnpm-workspace.yaml
```

**⚠️ content/ et server/ sont à la RACINE, pas dans app/**

### Technical Implementation Details

**TailwindCSS 4 - Configuration CSS-native:**
```css
/* app/assets/css/main.css */
@import "tailwindcss";
```

**Nuxt Config de base:**
```typescript
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  compatibilityDate: '2025-07-15',

  modules: [
    '@nuxt/image',
    '@nuxt/content',
  ],

  css: ['~/assets/css/main.css'],

  vite: {
    plugins: [tailwindcss()],
  },

  content: {
    database: {
      type: 'd1',
      bindingName: 'DB'
    }
  },

  nitro: {
    preset: 'cloudflare_pages',
  },

  features: {
    inlineStyles: true,
  },
})
```

**content.config.ts - Collections avec Zod:**
```typescript
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { z } from 'zod'

const articleSchema = z.object({
  title: z.string(),
  description: z.string(),
  slug: z.string(),
  pillar: z.enum(['ai', 'engineering', 'ux']),
  category: z.enum(['news', 'tutorial', 'deep-dive', 'case-study', 'retrospective']),
  level: z.enum(['all', 'beginner', 'intermediate', 'advanced']),
  tags: z.array(z.string()).default([]),
  publishedAt: z.string(),  // ISO date string
  updatedAt: z.string().optional(),
  image: z.string().optional(),
  draft: z.boolean().default(false),
  translationKey: z.string().optional(),
})

export default defineContentConfig({
  collections: {
    articles_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/**/*.md', prefix: '/blog' },
      schema: articleSchema,
    }),
    articles_en: defineCollection({
      type: 'page',
      source: { include: 'en/**/*.md', prefix: '/blog' },
      schema: articleSchema,
    }),
  }
})
```

**Page [slug].vue - Requête Content:**
```vue
<script setup lang="ts">
const route = useRoute()
const slug = route.params.slug as string

// Récupérer l'article par path (fr par défaut pour cette story)
const { data: article } = await useAsyncData(
  `article-${slug}`,
  () => queryCollection('articles_fr')
    .where('slug', '=', slug)
    .first()
)

if (!article.value) {
  throw createError({ statusCode: 404, message: 'Article not found' })
}
</script>

<template>
  <article v-if="article">
    <h1>{{ article.title }}</h1>
    <ContentRenderer :value="article" />
  </article>
</template>
```

### Package Versions (Décembre 2025)

| Package | Version |
|---------|---------|
| nuxt | 4.2.2 |
| @nuxt/content | 3.10.0+ |
| @nuxt/image | latest |
| tailwindcss | 4.1.17 |
| @tailwindcss/vite | 4.1.18 |
| zod | 4.x |
| pnpm | 10.26+ |
| node | 22 LTS |

### Commandes d'initialisation

```bash
# Créer le projet
pnpm dlx nuxi@latest init sebc-dev
cd sebc-dev && pnpm install

# Modules core
pnpm add @nuxt/content @nuxt/image
pnpm add -D @tailwindcss/vite tailwindcss wrangler

# Validation
pnpm add zod

# Créer .nvmrc
echo "22" > .nvmrc

# Créer D1 (OBLIGATOIRE)
wrangler d1 create content-db
# → Copier le database_id dans wrangler.toml
```

### Anti-patterns à éviter

- ❌ Ne PAS mettre content/ ou server/ dans app/
- ❌ Ne PAS utiliser `@nuxtjs/tailwindcss` module - utiliser `@tailwindcss/vite`
- ❌ Ne PAS oublier wrangler.toml - erreur 500 garantie en production
- ❌ Ne PAS utiliser `experimental.inlineSSRStyles` - renommé en `features.inlineStyles`
- ❌ Ne PAS utiliser radix-vue - rebrandé en reka-ui (février 2025)

### References

- [Source: architecture/project-structure-boundaries.md] - Structure complète du projet
- [Source: architecture/starter-template-evaluation/cloudflare-d1-requis-pour-nuxt-content-3.md] - Configuration D1 obligatoire
- [Source: architecture/starter-template-evaluation/configuration-nuxtconfigts-de-base.md] - nuxt.config.ts
- [Source: architecture/core-architectural-decisions/content-architecture.md] - Collections et schema
- [Source: architecture/starter-template-evaluation/arborescence-projet-nuxt-4.md] - Structure dossiers
- [Source: architecture/starter-template-evaluation/versions-des-packages-decembre-2025.md] - Versions packages
- [Source: prd/13-functional-requirements-mvp.md#FR1] - Requirement FR1

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

### File List
