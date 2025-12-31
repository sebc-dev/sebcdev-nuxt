# Enforcement Guidelines

**All AI Agents MUST:**

1. Suivre les conventions de nommage définies ci-dessus
2. Placer les tests à côté des fichiers sources
3. Utiliser `Intl.DateTimeFormat` pour le formatage de dates
4. Émettre les events Vue en kebab-case
5. Valider le contenu via Content 3 schema
6. **Configurer Cloudflare D1** : OBLIGATOIRE avec preset `cloudflare_pages` (Worker sans accès filesystem)

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

// ❌ Package @zod/mini (déplacé depuis mai 2025)
import { z } from '@zod/mini'  // Utiliser 'zod' ou 'zod/mini' (import path, pas package)

// ❌ Import zod depuis @nuxt/content (sera supprimé)
import { z } from '@nuxt/content'

// ❌ nuxt-delay-hydration (obsolète depuis Nuxt 3.16+)
modules: ['nuxt-delay-hydration']

// ❌ experimental.inlineSSRStyles (renommé)
experimental: { inlineSSRStyles: true }  // Utiliser features.inlineStyles

// ❌ Import depuis radix-vue (rebrandé)
import { ... } from 'radix-vue'  // Utiliser reka-ui

// ❌ Index de recherche généré côté client (performance)
const index = MiniSearch.loadJSON(...)  // Dans composant, re-parse à chaque navigation
// ✅ Index pré-chargé dans composable avec cache
const { searchIndex } = useSearch()  // Fetch unique + cache

// ❌ Composants Content v2 (supprimés dans v3)
<ContentDoc />
<ContentList />
<ContentQuery />
// ✅ Utiliser <ContentRenderer> + queryCollection()

// ❌ queryContent() (Content v2)
const posts = await queryContent('blog').find()
// ✅ queryCollection() (Content v3)
const posts = await queryCollection('blog').all()

// ❌ Configuration pnpm dans .npmrc
pnpm.onlyBuiltDependencies[]=sharp  // Utiliser package.json (champ pnpm) ou pnpm-workspace.yaml

// ❌ Oublier D1 avec cloudflare_pages (runtime Worker sans filesystem)
nitro: { preset: 'cloudflare_pages' }  // Sans content.database.type: 'd1' → Erreur 500 au runtime
// ✅ Configuration complète avec D1
content: { database: { type: 'd1', bindingName: 'DB' } }  // + wrangler.toml avec [[d1_databases]]

// ❌ Preset avec tiret (invalide)
nitro: { preset: 'cloudflare-pages' }  // Tiret
// ✅ Preset avec underscore
nitro: { preset: 'cloudflare_pages' }  // Underscore

// ❌ Tiret dans le binding name (invalide)
binding = "my-db"
// ✅ Binding sans tiret
binding = "DB"
binding = "contentDB"

// ❌ Preview local sans persistence (données perdues)
wrangler pages dev .output/public
// ✅ Preview local avec persistence
wrangler pages dev .output/public --persist

// ❌ APIs navigateur dans plugin sans suffixe
// app/plugins/tracker.ts
window.addEventListener('scroll', ...)  // Crash SSR
// ✅ Utiliser suffixe .client.ts ou guard
// app/plugins/tracker.client.ts
window.addEventListener('scroll', ...)  // OK

// ❌ Propriétés plugin dynamiques (empêche optimisations build)
export default defineNuxtPlugin({
  parallel: import.meta.server ? true : false,  // Dynamique = pas d'optimisation
})
// ✅ Propriétés statiques
export default defineNuxtPlugin({
  parallel: true,  // Statique = optimisé
})

// ❌ Commandes D1 sans spécifier l'environnement (Wrangler 3.33.0+ = local par défaut)
wrangler d1 execute DB --command "SELECT * FROM table"
// ✅ Toujours spécifier --local ou --remote
wrangler d1 execute DB --local --command "SELECT * FROM table"
wrangler d1 execute DB --remote --command "SELECT * FROM table"
```
