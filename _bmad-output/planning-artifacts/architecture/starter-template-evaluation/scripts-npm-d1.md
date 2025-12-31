# Scripts npm D1

**Clarification `nuxt generate` vs `nuxt build --prerender` :**

Les commandes `nuxt generate` et `nuxt build --prerender` produisent un résultat **identique** pour le SSG — les deux génèrent des fichiers statiques dans `.output/public/`. La différence avec `nuxt build` simple est que ce dernier inclut un serveur Nitro pour le SSR.

Pour ce projet avec D1, on utilise `nuxt build --preset=cloudflare_pages` qui combine SSG + Worker runtime pour les requêtes de contenu.

```json
{
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build --preset=cloudflare_pages",
    "generate": "nuxt generate",
    "postgenerate": "node scripts/generate-search-index.mjs",
    "preview": "wrangler pages dev .output/public --persist",
    "db:list": "wrangler d1 list",
    "db:info": "wrangler d1 info content-db",
    "db:local": "wrangler d1 execute DB --local --command",
    "db:remote": "wrangler d1 execute DB --remote --command",
    "db:export": "wrangler d1 export DB --remote --output backup.sql",
    "db:tables": "wrangler d1 execute DB --local --command \"SELECT name FROM sqlite_master WHERE type='table'\""
  }
}
```

**Script génération index MiniSearch :**

```javascript
// scripts/generate-search-index.mjs
import { readFileSync, writeFileSync, readdirSync } from 'fs'
import { join } from 'path'
import matter from 'gray-matter'

const CONTENT_DIR = './content'
const OUTPUT_PATH = './.output/public/search-index.json'

function extractArticles(dir, articles = []) {
  const files = readdirSync(dir, { withFileTypes: true })

  for (const file of files) {
    const path = join(dir, file.name)
    if (file.isDirectory()) {
      extractArticles(path, articles)
    } else if (file.name.endsWith('.md')) {
      const content = readFileSync(path, 'utf-8')
      const { data, content: body } = matter(content)

      if (!data.draft) {
        articles.push({
          id: data.slug || file.name.replace('.md', ''),
          title: data.title,
          description: data.description,
          content: body.replace(/<[^>]*>/g, '').substring(0, 500),
          pillar: data.pillar,
          tags: data.tags?.join(' ') || ''
        })
      }
    }
  }
  return articles
}

const articles = extractArticles(CONTENT_DIR)
writeFileSync(OUTPUT_PATH, JSON.stringify(articles, null, 2))
console.log(`✅ Search index generated: ${articles.length} articles`)
```
