# Script Génération Index (Build-time)

```javascript
// scripts/generate-search-index.mjs
import { readFileSync, writeFileSync, readdirSync, existsSync } from 'fs'
import { join } from 'path'
import matter from 'gray-matter'
import MiniSearch from 'minisearch'

const CONTENT_DIR = './content'
const OUTPUT_PATH = './.output/public/search-index.json'

// Configuration MiniSearch (doit matcher celle du client)
const MINISEARCH_OPTIONS = {
  idField: 'id',
  fields: ['title', 'description', 'content', 'tags'],
  storeFields: ['title', 'description', 'slug', 'locale', 'pillar'],
  processTerm: (term) => {
    const normalized = term
      .normalize('NFD')
      .replace(/[\u0300-\u036f]/g, '')
      .toLowerCase()
      .trim()
    if (normalized.length < 2) return null
    return normalized
  }
}

function stripMarkdown(content) {
  return content
    .replace(/```[\s\S]*?```/g, '')
    .replace(/`[^`]*`/g, '')
    .replace(/\[([^\]]+)\]\([^)]+\)/g, '$1')
    .replace(/!\[[^\]]*\]\([^)]+\)/g, '')
    .replace(/#{1,6}\s*/g, '')
    .replace(/[*_]{1,2}([^*_]+)[*_]{1,2}/g, '$1')
    .replace(/\s+/g, ' ')
    .trim()
}

function truncateContent(content, maxLength = 3000) {
  if (content.length <= maxLength) return content
  const truncated = content.slice(0, maxLength)
  const lastPeriod = truncated.lastIndexOf('.')
  return lastPeriod > maxLength * 0.8
    ? truncated.slice(0, lastPeriod + 1)
    : truncated
}

function extractArticles(dir, locale, articles = []) {
  if (!existsSync(dir)) return articles

  const files = readdirSync(dir, { withFileTypes: true })

  for (const file of files) {
    const path = join(dir, file.name)
    if (file.isDirectory()) {
      extractArticles(path, locale, articles)
    } else if (file.name.endsWith('.md')) {
      const raw = readFileSync(path, 'utf-8')
      const { data, content } = matter(raw)

      if (!data.draft) {
        const cleanContent = truncateContent(stripMarkdown(content))
        articles.push({
          id: `${locale}:${data.slug || file.name.replace('.md', '')}`,
          title: data.title || '',
          description: data.description || '',
          content: cleanContent,
          slug: data.path || `/${locale}/blog/${data.slug}`,
          locale,
          pillar: data.pillar || '',
          tags: Array.isArray(data.tags) ? data.tags.join(' ') : ''
        })
      }
    }
  }
  return articles
}

// Extraire articles FR et EN
const articles = [
  ...extractArticles(join(CONTENT_DIR, 'fr'), 'fr'),
  ...extractArticles(join(CONTENT_DIR, 'en'), 'en')
]

// Créer et peupler l'index
const miniSearch = new MiniSearch(MINISEARCH_OPTIONS)
miniSearch.addAll(articles)

// Sauvegarder l'index sérialisé
writeFileSync(OUTPUT_PATH, JSON.stringify(miniSearch))

console.log(`✅ Search index generated: ${articles.length} articles`)
console.log(`   - FR: ${articles.filter(a => a.locale === 'fr').length}`)
console.log(`   - EN: ${articles.filter(a => a.locale === 'en').length}`)
console.log(`   - Size: ${(JSON.stringify(miniSearch).length / 1024).toFixed(1)}KB`)
```

**Configuration package.json :**

```json
{
  "scripts": {
    "generate": "nuxt generate",
    "postgenerate": "node scripts/generate-search-index.mjs"
  }
}
```

## Alternative : Hook Nitro `nitro:build:public-assets`

Pour intégrer la génération d'index directement dans le build Nuxt (sans script externe) :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  hooks: {
    async 'nitro:build:public-assets'(nitro) {
      const { generateSearchIndexes } = await import('./scripts/build-search')
      await generateSearchIndexes(nitro.options.output.publicDir)
    }
  }
})
```

**Comparaison :**

| Approche | Avantages | Inconvénients |
|----------|-----------|---------------|
| `postgenerate` script | Simple, explicite, débugable | Fichier séparé |
| Hook Nitro | Intégré au build | Plus complexe, moins visible |

**Recommandation** : `postgenerate` pour sa simplicité. Le hook Nitro est utile si vous avez besoin d'accès aux options Nitro.

## API `queryCollectionSearchSections()` (Nuxt Content 3.10+)

API native pour extraire les sections recherchables depuis les collections SQLite :

```typescript
// server/api/search-sections/[locale].get.ts
import type { Collections } from '@nuxt/content'

export default defineEventHandler(async (event) => {
  const locale = getRouterParam(event, 'locale') || 'fr'
  const collectionName = `articles_${locale}` as keyof Collections

  return queryCollectionSearchSections(event, collectionName, {
    ignoredTags: ['code', 'pre', 'script'],  // Exclure blocs de code
    minHeading: 'h2',                         // Niveau min (v3.10.0+)
    maxHeading: 'h4',                         // Niveau max
  })
})
```

**Structure `Section` retournée :**

```typescript
interface Section {
  id: string       // "/blog/article#section" (utilisable comme URL)
  title: string    // Texte du heading
  titles: string[] // Breadcrumb des parents ["Article", "Introduction"]
  content: string  // Contenu textuel de la section
  level: number    // Niveau du heading (1-6)
}
```

**⚠️ Note SSG** : Cette API requiert une requête D1 runtime. Pour SSG pur, préférer le script build-time qui génère un index statique sans consommer le quota D1.
