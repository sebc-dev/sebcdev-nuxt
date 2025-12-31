# Cloudflare D1 + Nuxt Content 3 : guide des bonnes pratiques (2025)

**Clarification critique d'emblée** : D1 est requis pour Nuxt Content 3 sur Cloudflare Pages **en mode SSR uniquement**, pas en SSG pur. En mode SSG (`nuxt generate`), le contenu est pré-rendu au build et les requêtes client utilisent **WASM SQLite** dans le navigateur — D1 n'est jamais sollicité au runtime. Ce guide couvre le cas SSR avec le preset `cloudflare_pages`, qui est probablement votre configuration réelle si vous avez besoin de D1.

---

## Configuration D1 pour Nuxt Content 3

La configuration minimale requiert trois fichiers synchronisés : `nuxt.config.ts`, `wrangler.toml` (ou `.jsonc`), et les bindings dans le Dashboard Cloudflare. Le binding par défaut est **`DB`** — toute divergence de nommage provoque l'erreur "Missing Cloudflare DB binding (D1)".

**nuxt.config.ts — configuration de base :**
```typescript
export default defineNuxtConfig({
  compatibilityDate: '2025-01-01',
  modules: ['@nuxt/content'],
  
  nitro: {
    preset: 'cloudflare_pages', // Obligatoire pour D1
  },
  
  content: {
    database: {
      type: 'd1',
      bindingName: 'DB' // Doit correspondre au binding Cloudflare
    }
  }
})
```

**wrangler.toml — pour le développement local uniquement :**
```toml
[[d1_databases]]
binding = "DB"
database_name = "my-content-db"
database_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
preview_database_id = "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"
migrations_dir = "./migrations"
```

**Point crucial** : `wrangler.toml` n'est utilisé que pour le développement local. Les bindings de production **doivent** être configurés dans Cloudflare Dashboard → Pages → Settings → Bindings, séparément pour Production et Preview.

---

## Workflow de déploiement et architecture de données

Nuxt Content 3 utilise une **architecture dump/restore** plutôt que des migrations traditionnelles. Au build, le module parse tout le contenu, génère un fichier dump SQL, et l'inclut dans le bundle. Au premier cold start en production, ce dump est restauré dans D1.

**Processus en 4 étapes :**
1. `nuxt build --preset=cloudflare_pages` parse le contenu et génère le dump
2. Le build produit `.output/public` avec le dump intégré
3. Déploiement via `wrangler pages deploy .output/public`
4. Premier accès : Nuxt Content détecte le dump et le restaure dans D1

**content.config.ts — définition des collections :**
```typescript
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { z } from 'zod'

export default defineContentConfig({
  collections: {
    content: defineCollection({
      type: 'page',
      source: '**/*.md',
      schema: z.object({
        title: z.string(),
        description: z.string().optional(),
      }),
      // Indexes pour optimiser les requêtes et réduire les rows_read
      indexes: [
        { columns: ['path'] },
        { columns: ['title'] },
      ]
    })
  }
})
```

Les tables `_content_info` et `_content_content` sont créées automatiquement lors de la restauration — **aucune migration manuelle n'est nécessaire** pour Nuxt Content.

---

## Limites du free tier D1 — ce qui compte vraiment

Le free tier D1 impose des limites quotidiennes strictes, appliquées depuis le **10 février 2025**. En cas de dépassement, les requêtes échouent jusqu'à 00:00 UTC.

| Ressource | Free Tier | Workers Paid ($5/mois) |
|-----------|-----------|------------------------|
| Rows read | **5 millions/jour** | 25 milliards/mois |
| Rows written | **100 000/jour** | 50 millions/mois |
| Stockage total | **5 GB** | 1 TB |
| Taille max par DB | **500 MB** | 10 GB |
| Nombre de bases | **10** | 50 000 |
| Time Travel | 7 jours | 30 jours |

**Ce qui compte dans le quota :**
- ✅ Requêtes depuis Workers/Pages déployés
- ✅ `wrangler d1 execute --remote`
- ✅ Requêtes via Dashboard Cloudflare
- ❌ Développement local (`wrangler dev` sans `--remote`)
- ❌ Build-time (D1 n'est pas accessible pendant le build)

**Piège majeur des quotas** : D1 facture les **rows_read**, pas les rows_returned. Un `SELECT * FROM table` sur 5000 lignes sans index = 5000 rows_read même avec `LIMIT 1`. Un développeur a reçu une facture de **$3,200** pour des requêtes non indexées.

---

## Configuration multi-environnements recommandée

**wrangler.toml avec environnements :**
```toml
# Défaut (développement local)
[[d1_databases]]
binding = "DB"
database_name = "myapp-dev"
database_id = "dev-uuid"

# Staging
[env.staging]
[[env.staging.d1_databases]]
binding = "DB"
database_name = "myapp-staging"
database_id = "staging-uuid"

# Production
[env.production]
[[env.production.d1_databases]]
binding = "DB"
database_name = "myapp-prod"
database_id = "prod-uuid"
```

**Scripts npm utiles :**
```json
{
  "scripts": {
    "dev": "nuxi dev",
    "build": "nuxi build --preset=cloudflare_pages",
    "preview": "wrangler pages dev .output/public",
    "db:migrate:local": "wrangler d1 migrations apply DB --local",
    "db:migrate:remote": "wrangler d1 migrations apply DB --remote",
    "db:execute:local": "wrangler d1 execute DB --local --command",
    "db:export": "wrangler d1 export DB --remote --output backup.sql"
  }
}
```

---

## Debugging et commandes wrangler essentielles

**Commandes de diagnostic :**
```bash
# Lister toutes les bases D1 du compte
wrangler d1 list

# Infos sur une base spécifique
wrangler d1 info my-content-db

# Exécuter une requête locale
wrangler d1 execute DB --local --command "SELECT name FROM sqlite_master WHERE type='table'"

# Exécuter une requête sur la production
wrangler d1 execute DB --remote --command "SELECT * FROM _content_info LIMIT 5"

# Vérifier le statut des migrations
wrangler d1 migrations list DB --local
wrangler d1 migrations list DB --remote

# Exporter pour backup
wrangler d1 export DB --remote --output backup.sql

# Time Travel (récupération point-in-time)
wrangler d1 time-travel info DB --timestamp "2025-12-25T10:00:00Z"
```

**Localisation des données locales :**
```
.wrangler/state/v3/d1/miniflare-D1DatabaseObject/*.sqlite
```

Inspectable avec n'importe quel client SQLite ou l'extension VS Code SQLite.

**Erreurs courantes et solutions :**

| Erreur | Cause | Solution |
|--------|-------|----------|
| "Missing Cloudflare DB binding (D1)" | Binding manquant ou mal nommé | Vérifier que `DB` est identique dans nuxt.config.ts, wrangler.toml ET Dashboard |
| "no such table: _content_info" | Dump non restauré | Redéployer, vérifier que le build inclut le dump |
| "binding DB must have a database that already exists" | database_id incorrect | `wrangler d1 list` pour vérifier l'UUID |
| Tables vides après deploy | Preview vs Production | Dans Dashboard, scroller vers le haut pour vérifier l'environnement sélectionné |

---

## Pièges et anti-patterns à éviter absolument

**❌ Ne pas utiliser `--persist` en développement local :**
```bash
# MAUVAIS - données perdues entre sessions
wrangler pages dev .output/public

# BON - données persistantes
wrangler pages dev .output/public --persist
```

**❌ Ne pas mélanger les noms de preset :**
```typescript
// MAUVAIS - inconsistant
nitro: { preset: "cloudflare-pages" } // avec tiret

// BON
nitro: { preset: "cloudflare_pages" } // avec underscore
```

**❌ Ne pas utiliser de tirets dans le binding name :**
```toml
# MAUVAIS - tiret invalide
binding = "my-db"

# BON
binding = "DB"
binding = "contentDB"
```

**❌ Ne pas oublier de configurer les bindings Preview dans Dashboard :**
Le UI Cloudflare a un bug d'affichage — l'onglet Production/Preview scroll hors de vue. Toujours vérifier en scrollant vers le haut.

**❌ Ne pas utiliser Wrangler 4.2.0 avec NuxtHub :**
Version cassée pour D1. Épingler à `"wrangler": "4.0.0"` dans package.json.

**❌ Ne pas tenter `wrangler dev --remote` avec Pages + D1 :**
Actuellement impossible. Le développement local utilise uniquement la DB locale.

---

## Workflow CI/CD avec GitHub Actions

**Workflow complet :**
```yaml
name: Deploy to Cloudflare Pages

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      deployments: write
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Build Nuxt
        run: npm run build -- --preset=cloudflare_pages
        
      # Migrations (si vous utilisez Drizzle ou migrations custom)
      - name: Apply D1 Migrations
        if: github.ref == 'refs/heads/main'
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: d1 migrations apply DB --remote
          
      - name: Deploy to Pages
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy .output/public --project-name=my-nuxt-site
          gitHubToken: ${{ secrets.GITHUB_TOKEN }}

env:
  NO_D1_WARNING: true
```

**Secrets requis :**
- `CLOUDFLARE_API_TOKEN` : créer avec le template "Edit Cloudflare Workers"
- `CLOUDFLARE_ACCOUNT_ID` : visible dans Dashboard → Workers & Pages (sidebar droite)

**Pour les preview deployments :** Wrangler détecte automatiquement la branche et déploie en preview pour les PRs, en production pour main.

---

## Récapitulatif des breaking changes récents

**Nuxt Content v3** introduit des changements majeurs :
- `queryContent()` → `queryCollection()`
- `fetchContentNavigation()` → `queryCollectionNavigation()`
- `_dir.yml` → `.navigation.yml`
- Propriété `._path` → `.path` (sans underscore)
- `<ContentDoc>`, `<ContentList>` supprimés → utiliser `<ContentRenderer>`

**Wrangler 3.33.0+** : les commandes D1 utilisent maintenant **local par défaut**. Toujours spécifier `--local` ou `--remote` explicitement.

**D1 Free Tier (10 février 2025)** : les limites quotidiennes sont maintenant strictement appliquées — les requêtes échouent au-delà des quotas jusqu'au reset UTC.

---

## Conclusion

Pour un déploiement réussi de Nuxt Content 3 avec D1 sur Cloudflare Pages en free tier, trois points sont non négociables : **synchroniser les noms de binding** entre les trois emplacements (config, wrangler, Dashboard), **créer des indexes** sur toutes les colonnes requêtées pour éviter les factures surprises, et **configurer les bindings pour Production ET Preview** dans le Dashboard Cloudflare. La limite de **500 MB par base** et **5 millions rows_read/jour** en free tier est généralement suffisante pour un site de contenu moyen, à condition d'éviter les full table scans. Pour le développement local, utilisez toujours `--persist` et inspectez directement le fichier SQLite dans `.wrangler/state/` pour déboguer efficacement.