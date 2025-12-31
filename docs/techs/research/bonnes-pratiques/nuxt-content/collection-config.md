# Configuration des collections @nuxt/content v3.10+ : guide complet pour Cloudflare Pages

Pour un blog multilingue Nuxt 4 sur Cloudflare Pages, **Cloudflare D1 est obligatoire** (pas de SQLite fichier possible), les collections doivent être définies séparément par langue (`blog_fr`, `blog_en`), et les index SQLite configurés dans `defineCollection()` sont essentiels pour la performance des requêtes. La migration depuis Content v2 implique l'abandon total de `queryContent()` au profit de `queryCollection()` avec syntaxe SQL.

## Architecture fondamentale de content.config.ts

Le fichier `content.config.ts` à la racine du projet définit toutes les collections via `defineContentConfig()`. Chaque collection accepte quatre propriétés principales : `type` ('page' ou 'data'), `source` (pattern glob ou objet de configuration), `schema` (validation Zod), et `indexes` (optimisation SQLite).

```typescript
// content.config.ts - Configuration blog multilingue complète
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { z } from 'zod'  // Import direct depuis zod, PAS depuis @nuxt/content (déprécié)

const blogSchema = z.object({
  title: z.string(),
  description: z.string().optional(),
  date: z.date(),
  draft: z.boolean().default(false),
  tags: z.array(z.string()).default([]),
  image: z.string().optional(),
  author: z.string().optional(),
})

export default defineContentConfig({
  collections: {
    blog_fr: defineCollection({
      type: 'page',
      source: {
        include: 'fr/blog/**/*.md',
        prefix: '/blog'  // Génère /blog/mon-article au lieu de /fr/blog/mon-article
      },
      schema: blogSchema,
      indexes: [
        { columns: ['date'] },
        { columns: ['draft'] },
        { columns: ['draft', 'date'], name: 'idx_published_fr' },
        { columns: ['path'], unique: true },
      ],
    }),
    
    blog_en: defineCollection({
      type: 'page',
      source: {
        include: 'en/blog/**/*.md',
        prefix: '/blog'
      },
      schema: blogSchema,
      indexes: [
        { columns: ['date'] },
        { columns: ['draft'] },
        { columns: ['draft', 'date'], name: 'idx_published_en' },
        { columns: ['path'], unique: true },
      ],
    }),
  }
})
```

Le `type: 'page'` génère automatiquement les champs `path`, `title`, `description`, `seo`, `body` et `navigation`. Le `type: 'data'` offre un contrôle total du schéma pour les données structurées (auteurs, configurations). L'option `prefix: '/blog'` dans source retire le préfixe de langue du path final généré.

## Configuration Cloudflare Pages avec D1 obligatoire

Cloudflare Pages **ne supporte pas** SQLite fichier ni `:memory:` de manière fiable. **D1 est obligatoire** pour le runtime edge. La configuration complète dans `nuxt.config.ts` pour Nuxt 4.2.x requiert :

```typescript
// nuxt.config.ts - Configuration production Cloudflare Pages
export default defineNuxtConfig({
  compatibilityDate: '2025-05-15',  // Minimum 2024-09-19 pour Workers
  
  srcDir: 'app/',  // Structure Nuxt 4
  
  modules: ['@nuxt/content', '@nuxtjs/i18n'],
  
  nitro: {
    preset: 'cloudflare_pages',
  },
  
  content: {
    database: {
      type: 'd1',
      bindingName: 'DB'  // Nom du binding D1 dans Cloudflare dashboard
    },
    experimental: {
      sqliteConnector: 'better-sqlite3'  // Recommandé pour CI Cloudflare
    }
  },
  
  i18n: {
    locales: [
      { code: 'fr', language: 'fr-FR', name: 'Français' },
      { code: 'en', language: 'en-US', name: 'English' }
    ],
    defaultLocale: 'fr',
    strategy: 'prefix_except_default',
  },
  
  // Pre-rendering pour optimiser cold starts (budget 0€)
  routeRules: {
    '/': { prerender: true },
    '/blog/**': { prerender: true },
    '/en/blog/**': { prerender: true },
  }
})
```

L'architecture de stockage fonctionne ainsi : au build, les fichiers Markdown sont parsés en AST puis stockés dans des tables SQLite, générant un fichier dump. Au runtime, ce dump est restauré dans D1 à chaque cold start. Côté client, une version WASM SQLite permet les requêtes offline.

## Structure de dossiers optimale pour multilingue

La structure recommandée sépare le contenu par locale au premier niveau, avec `content/` restant à la racine (rootDir) conformément à Nuxt 4 :

```
project/
├── app/                          # srcDir (Nuxt 4)
│   ├── pages/
│   │   ├── blog/
│   │   │   ├── index.vue
│   │   │   └── [...slug].vue
│   │   └── [...slug].vue
│   └── components/
├── content/                      # Reste à rootDir
│   ├── fr/
│   │   ├── index.md
│   │   └── blog/
│   │       ├── 1.premier-article.md
│   │       └── 2.deuxieme-article.md
│   └── en/
│       ├── index.md
│       └── blog/
│           ├── 1.first-post.md
│           └── 2.second-post.md
├── server/                       # À rootDir
├── content.config.ts
└── nuxt.config.ts
```

Le préfixe numérique (`1.`, `2.`) contrôle l'ordre de navigation. Le séparateur doit être `.` (pas `-` ou `_`).

## Requêtes queryCollection() avec changement de locale

L'API `queryCollection()` remplace totalement `queryContent()` et utilise une syntaxe SQL. La construction dynamique du nom de collection permet le switch de langue :

```vue
<script setup lang="ts">
// pages/blog/index.vue
import type { Collections } from '@nuxt/content'

const { locale } = useI18n()

const { data: posts } = await useAsyncData(
  `blog-${locale.value}`,
  async () => {
    const collection = `blog_${locale.value}` as keyof Collections
    return await queryCollection(collection)
      .where('draft', '=', false)
      .order('date', 'DESC')
      .all()
  },
  { watch: [locale] }  // Re-fetch automatique au changement de langue
)
</script>
```

Les opérateurs disponibles incluent `=`, `<>`, `<`, `>`, `<=`, `>=`, `LIKE`, `IN`, `BETWEEN`, `IS NULL`. Pour les conditions complexes :

```typescript
// Conditions groupées avec AND/OR
queryCollection('blog_fr')
  .where('draft', '=', false)
  .andWhere(q => q.where('date', '>', '2024-01-01').where('category', '=', 'tech'))
  .orWhere(q => q.where('featured', '=', true))
  .order('date', 'DESC')
  .select('path', 'title', 'date', 'description')
  .limit(10)
  .all()
```

## Index SQLite pour performance optimale

Les index définis dans `defineCollection()` impactent directement les performances des requêtes. **Toute colonne utilisée dans `where()` ou `order()` devrait être indexée** :

| Colonne | Usage | Type d'index |
|---------|-------|--------------|
| `date` | Tri chronologique | Simple |
| `draft` | Filtrage publié/brouillon | Simple |
| `path` | Lookup par URL | Unique |
| `(draft, date)` | Liste publiés triés | Composite |

```typescript
indexes: [
  { columns: ['date'] },
  { columns: ['draft'] },
  { columns: ['draft', 'date'], name: 'idx_published' },
  { columns: ['path'], unique: true },
  { columns: ['tags'] },  // Si filtrage par tag fréquent
]
```

Pour D1 sur Cloudflare, un `WHERE` sur colonne indexée = **1 row read** au lieu d'un table scan complet, réduisant significativement les coûts et la latence.

## Breaking changes majeurs v2 vers v3

La migration nécessite des changements structurels importants. Les APIs et composants ont été entièrement remplacés :

- **`queryContent()` → `queryCollection()`** : syntaxe SQL obligatoire
- **`where({ field: value })` → `where('field', '=', value)`** : opérateurs explicites
- **`sort({ date: -1 })` → `order('date', 'DESC')`** : nomenclature SQL
- **`only(['field'])` → `select('field')`** : projection de colonnes
- **`find()` → `all()`**, **`findOne()` → `first()`** : terminaison de requête
- **Composants `<ContentDoc>`, `<ContentList>`** → `<ContentRenderer>` uniquement
- **Document Driven Mode** : supprimé, créer manuellement `pages/[...slug].vue`
- **Fichiers `_dir.yml`** → renommés `.navigation.yml`
- **Import Zod** : `import { z } from '@nuxt/content'` déprécié → `import { z } from 'zod'`

```typescript
// v2 (obsolète)
const posts = await queryContent('/blog').where({ published: true }).sort({ date: -1 }).find()

// v3 (actuel)
const posts = await queryCollection('blog').where('published', '=', true).order('date', 'DESC').all()
```

## Nouvelles fonctionnalités v3.10+

La version **3.10.0** (décembre 2024) introduit les **index de base de données optionnels** pour améliorer les performances, le **clone shallow des sources Git** pour un téléchargement plus rapide, et le support des **collections CSV single-file**. Les versions récentes ajoutent également :

- **v3.8** : Extension de schéma hérité, hot reload via Vite
- **v3.7** : Support Zod v4 et Valibot comme validateurs alternatifs
- **v3.6** : Support Cloudflare Workers, opérateurs `<=` et `>=`

Nouvelles APIs de requêtes disponibles :

```typescript
// Compter les résultats
const count = await queryCollection('blog_fr').where('draft', '=', false).count()

// Recherche full-text
const sections = await queryCollectionSearchSections('blog_fr', 'nuxt tutorial')

// Navigation prev/next
const [prev, next] = await queryCollectionItemSurroundings('blog_fr', route.path)
```

## Configuration i18n complète avec fallback

L'intégration avec `@nuxtjs/i18n` v10.2.1+ requiert la stratégie `prefix_except_default` pour un blog FR/EN où le français est la locale par défaut :

```vue
<script setup lang="ts">
// pages/[...slug].vue - Page catch-all multilingue avec fallback
import { withLeadingSlash } from 'ufo'
import type { Collections } from '@nuxt/content'

const route = useRoute()
const { locale, defaultLocale } = useI18n()

const slug = computed(() => withLeadingSlash(String(route.params.slug || '')))

const { data: page } = await useAsyncData(
  `page-${locale.value}-${slug.value}`,
  async () => {
    const collection = `content_${locale.value}` as keyof Collections
    const content = await queryCollection(collection).path(slug.value).first()
    
    // Fallback vers locale par défaut si contenu manquant
    if (!content && locale.value !== defaultLocale.value) {
      const fallbackCollection = `content_${defaultLocale.value}` as keyof Collections
      return await queryCollection(fallbackCollection).path(slug.value).first()
    }
    return content
  },
  { watch: [locale] }
)
</script>

<template>
  <ContentRenderer v-if="page" :value="page" />
  <div v-else>Contenu non disponible</div>
</template>
```

## Conclusion

La configuration optimale pour un blog multilingue Nuxt 4 sur Cloudflare Pages avec budget 0€ repose sur cinq piliers : **D1 obligatoire** pour le stockage serverless, **collections séparées par langue** (`blog_fr`, `blog_en`) avec pattern glob `include: 'fr/blog/**'` et `prefix: '/blog'`, **index SQLite sur toutes les colonnes de filtrage**, **pre-rendering via routeRules** pour minimiser les cold starts, et **migration complète vers `queryCollection()`** avec syntaxe SQL. L'option `watch: [locale]` dans `useAsyncData` garantit le rafraîchissement automatique lors du changement de langue. Pour la validation des schémas, privilégier Zod v4 importé directement (`import { z } from 'zod'`) plutôt que l'export déprécié de `@nuxt/content`.