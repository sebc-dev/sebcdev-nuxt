# Starter Template Evaluation

## Primary Technology Domain

Full-stack Content Platform (Nuxt 4 SSG) - génération statique complète pour Cloudflare Pages.

## Approche Sélectionnée : Initialisation Modulaire

**Rationale :** Étant donné le stack spécifique (Nuxt 4 + Content 3 + TailwindCSS 4 + shadcn-vue), une approche modulaire garantit les versions les plus récentes et un contrôle total sur chaque composant. Aucun template existant ne combine exactement ces technologies dans leurs dernières versions.

## Commande d'Initialisation

```bash
# Créer projet Nuxt 4
pnpm dlx nuxi@latest init sebc-dev
cd sebc-dev && pnpm install

# Modules essentiels
pnpm add @nuxt/content @nuxt/image
pnpm add -D @tailwindcss/vite tailwindcss wrangler
pnpm dlx nuxi@latest module add shadcn-nuxt
pnpm dlx nuxi@latest module add i18n

# Suite SEO complète (sitemap, robots, schema.org, og-image, link-checker)
pnpm add @nuxtjs/seo

# Validation de schéma
pnpm add zod

# Recherche full-text client-side
pnpm add minisearch

# Modules performance Lighthouse
pnpm add -D nuxt-vitalizer

# Génération automatique llms.txt
pnpm add nuxt-llms

# Security headers + CSP hash generation SSG
pnpm add nuxt-security

# Tests a11y CI
pnpm add -D @axe-core/playwright

# Initialiser shadcn-vue (utilise Reka UI)
pnpm dlx shadcn-vue@latest init

# Créer la base D1 Cloudflare (OBLIGATOIRE pour Nuxt Content 3 + cloudflare_pages)
wrangler d1 create content-db
# → Copier le database_id affiché dans wrangler.toml

# Créer le fichier .nvmrc pour cohérence Node.js
echo "22" > .nvmrc
```

## Fichier .nvmrc

Le fichier `.nvmrc` garantit la cohérence de la version Node.js entre développement local, CI et Cloudflare Pages. Cette méthode est plus fiable que les variables d'environnement UI quand wrangler.toml est présent.

```
22
```

## Cloudflare D1 - Requis pour Nuxt Content 3

**⚠️ CRITIQUE : D1 est OBLIGATOIRE avec le preset `cloudflare_pages`**

| Contexte | Accès fichiers | Base de données |
|----------|----------------|-----------------|
| Build (Node.js) | ✅ Filesystem disponible | SQLite local généré |
| Runtime (CF Worker) | ❌ Pas d'accès filesystem | D1 obligatoire |

**Pourquoi D1 est requis :**
- Le preset `cloudflare_pages` déploie un Worker (serveur) pour les routes API et le contenu dynamique
- Le runtime Worker n'a **pas accès au système de fichiers**
- Il ne peut donc pas lire le fichier SQLite généré lors du build
- Sans D1 configuré : **Erreur 500 au runtime** dès qu'une requête interroge l'API de contenu

**Configuration wrangler.toml :**

```toml
name = "sebc-dev"
compatibility_date = "2024-09-19"

[[d1_databases]]
binding = "DB"
database_name = "content-db"
database_id = "VOTRE_DATABASE_ID"  # Généré par: wrangler d1 create content-db
```

**Configuration nuxt.config.ts :**

```typescript
content: {
  database: {
    type: 'd1',
    bindingName: 'DB'  // Doit correspondre au binding dans wrangler.toml
  }
}
```

Voir [Limites Cloudflare Free Tier](#limites-cloudflare-free-tier) pour les quotas D1 et Pages.

## Scripts npm D1

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

## Commandes Wrangler D1 Essentielles

**Diagnostic :**
```bash
# Lister toutes les bases D1 du compte
wrangler d1 list

# Infos sur une base spécifique
wrangler d1 info content-db

# Vérifier les tables locales
wrangler d1 execute DB --local --command "SELECT name FROM sqlite_master WHERE type='table'"

# Vérifier les tables en production
wrangler d1 execute DB --remote --command "SELECT * FROM _content_info LIMIT 5"
```

**Backup et Time Travel :**
```bash
# Exporter pour backup
wrangler d1 export DB --remote --output backup.sql

# Time Travel (récupération point-in-time, 7 jours free tier)
wrangler d1 time-travel info DB --timestamp "2025-12-25T10:00:00Z"
```

**Localisation des données locales :**
```
.wrangler/state/v3/d1/miniflare-D1DatabaseObject/*.sqlite
```
Inspectable avec n'importe quel client SQLite ou l'extension VS Code SQLite.

## CI/CD GitHub Actions (Recommandé)

L'intégration Git directe de Cloudflare Pages est simple, mais un workflow GitHub Actions offre plus de contrôle :
- **Direct Uploads ne comptent pas dans le quota 500 builds/mois**
- Jobs parallèles (lint, typecheck, build) pour réduire le temps total
- Cache pnpm et Nuxt optimisé
- Preview URLs automatiques dans les PRs
- Cleanup des previews après merge

### Workflow principal optimisé

```yaml
# .github/workflows/deploy.yml
name: CI/CD Nuxt 4 Blog

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, synchronize, reopened]
  workflow_dispatch:

# Annule les workflows précédents sur la même branche
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

env:
  NODE_VERSION: '22'
  PNPM_VERSION: '10'

jobs:
  # ═══════════════════════════════════════════════════════════════
  # LINT - Parallèle avec typecheck
  # ═══════════════════════════════════════════════════════════════
  lint:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}
          cache: true  # Cache intégré - gère automatiquement le pnpm store

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - run: pnpm install --frozen-lockfile
      - run: pnpm lint

  # ═══════════════════════════════════════════════════════════════
  # TYPECHECK - Parallèle avec lint
  # ═══════════════════════════════════════════════════════════════
  typecheck:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}
          cache: true

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck

  # ═══════════════════════════════════════════════════════════════
  # BUILD - Après lint + typecheck
  # ═══════════════════════════════════════════════════════════════
  build:
    needs: [lint, typecheck]
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}
          cache: true

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      # Cache spécifique Nuxt 4 (artefacts de build)
      - name: Cache Nuxt build
        uses: actions/cache@v4
        with:
          path: |
            node_modules/.cache/nuxt
            node_modules/.vite
          key: ${{ runner.os }}-nuxt-build-${{ hashFiles('nuxt.config.ts', 'pnpm-lock.yaml') }}-${{ hashFiles('app/**/*', 'content/**/*') }}
          restore-keys: |
            ${{ runner.os }}-nuxt-build-${{ hashFiles('nuxt.config.ts', 'pnpm-lock.yaml') }}-
            ${{ runner.os }}-nuxt-build-

      - run: pnpm install --frozen-lockfile

      - run: pnpm run build
        env:
          NUXT_PUBLIC_SITE_URL: ${{ vars.SITE_URL }}

      # Upload artifact pour le job deploy (évite rebuild)
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: .output/public
          retention-days: 1

  # ═══════════════════════════════════════════════════════════════
  # DEPLOY - Télécharge l'artifact et déploie
  # ═══════════════════════════════════════════════════════════════
  deploy:
    needs: [build]
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      contents: read
      deployments: write
      pull-requests: write

    environment:
      name: ${{ github.ref == 'refs/heads/main' && 'production' || 'preview' }}
      url: ${{ steps.deploy.outputs.pages-deployment-alias-url }}

    outputs:
      url: ${{ steps.deploy.outputs.pages-deployment-alias-url }}

    steps:
      - uses: actions/checkout@v4

      - uses: actions/download-artifact@v4
        with:
          name: build-output
          path: .output/public

      - name: Deploy to Cloudflare Pages
        id: deploy
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy .output/public --project-name=${{ vars.CF_PROJECT_NAME }} --branch=${{ github.head_ref || github.ref_name }}
          gitHubToken: ${{ secrets.GITHUB_TOKEN }}

  # ═══════════════════════════════════════════════════════════════
  # COMMENT PR - Affiche l'URL preview dans la PR
  # ═══════════════════════════════════════════════════════════════
  comment-pr:
    needs: [deploy]
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - name: Find existing comment
        uses: peter-evans/find-comment@v3
        id: fc
        with:
          issue-number: ${{ github.event.pull_request.number }}
          comment-author: 'github-actions[bot]'
          body-includes: 'Preview Deployment'

      - name: Create or update comment
        uses: peter-evans/create-or-update-comment@v5
        with:
          comment-id: ${{ steps.fc.outputs.comment-id }}
          issue-number: ${{ github.event.pull_request.number }}
          edit-mode: replace
          body: |
            ## 🚀 Preview Deployment

            | Info | Valeur |
            |------|--------|
            | **Commit** | `${{ github.event.pull_request.head.sha }}` |
            | **Preview URL** | ${{ needs.deploy.outputs.url }} |

            _Mis à jour automatiquement à chaque push_
          reactions: rocket
```

### Workflow cleanup des preview deployments

Cloudflare ne supprime pas automatiquement les déploiements preview après merge. Ce workflow nettoie :

```yaml
# .github/workflows/cleanup-previews.yml
name: Cleanup Preview Deployments

on:
  pull_request:
    types: [closed]

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Delete preview deployment
        run: |
          DEPLOYMENTS=$(curl -s \
            "https://api.cloudflare.com/client/v4/accounts/${{ secrets.CLOUDFLARE_ACCOUNT_ID }}/pages/projects/${{ vars.CF_PROJECT_NAME }}/deployments" \
            -H "Authorization: Bearer ${{ secrets.CLOUDFLARE_API_TOKEN }}")

          BRANCH="${{ github.head_ref }}"
          DEPLOYMENT_ID=$(echo $DEPLOYMENTS | jq -r ".result[] | select(.deployment_trigger.metadata.branch == \"$BRANCH\") | .id" | head -1)

          if [ -n "$DEPLOYMENT_ID" ] && [ "$DEPLOYMENT_ID" != "null" ]; then
            echo "Deleting deployment $DEPLOYMENT_ID for branch $BRANCH"
            curl -X DELETE \
              "https://api.cloudflare.com/client/v4/accounts/${{ secrets.CLOUDFLARE_ACCOUNT_ID }}/pages/projects/${{ vars.CF_PROJECT_NAME }}/deployments/${DEPLOYMENT_ID}?force=true" \
              -H "Authorization: Bearer ${{ secrets.CLOUDFLARE_API_TOKEN }}"
          else
            echo "No deployment found for branch $BRANCH"
          fi
```

### Retry strategy pour déploiements instables

Pour les cas de rate limiting ou erreurs réseau transitoires :

```yaml
# Alternative au step deploy standard
- name: Deploy with retry
  uses: nick-fields/retry@v3
  with:
    timeout_minutes: 10
    max_attempts: 3
    retry_wait_seconds: 30
    command: |
      npx wrangler pages deploy .output/public \
        --project-name=${{ vars.CF_PROJECT_NAME }} \
        --branch=${{ github.head_ref || github.ref_name }}
  env:
    CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

### Configuration requise

**Secrets GitHub (Settings → Secrets and variables → Actions → Secrets) :**
- `CLOUDFLARE_API_TOKEN` : Token avec permissions Cloudflare Pages Edit
- `CLOUDFLARE_ACCOUNT_ID` : ID du compte Cloudflare

**Variables GitHub (Settings → Secrets and variables → Actions → Variables) :**
- `CF_PROJECT_NAME` : `sebc-dev`
- `SITE_URL` : `https://sebc.dev`

**Création du token API Cloudflare :**
1. Naviguer vers https://dash.cloudflare.com/profile/api-tokens
2. **Create Token** → **Custom Token**
3. Permissions : Account → **Cloudflare Pages** → **Edit**
4. Restreindre au compte spécifique

### Cache pnpm : approches disponibles

| Approche | Configuration | Cas d'usage |
|----------|---------------|-------------|
| **`cache: true` (Recommandé)** | `pnpm/action-setup@v4` avec `cache: true` | 90% des projets, simplifié |
| Cache manuel | `actions/cache@v4` avec pnpm store path | Contrôle avancé, rotation temporelle |
| setup-node cache | `actions/setup-node` avec `cache: 'pnpm'` | **⚠️ Ne pas combiner avec pnpm/action-setup cache** |

**Règles critiques :**
- Ne jamais combiner `pnpm/action-setup` cache ET `setup-node` cache simultanément
- Ne pas cacher `node_modules` directement (déconseillé par GitHub)
- Toujours utiliser `--frozen-lockfile` en CI

### Outputs wrangler-action disponibles (≥3.78.0)

| Output | Description |
|--------|-------------|
| `deployment-url` | URL unique du déploiement |
| `pages-deployment-alias-url` | URL alias de branche (preview) |
| `pages-deployment-id` | ID du déploiement (≥3.81.0) |
| `pages-environment` | `production` ou `preview` |

### Renovate Bot (Gestion automatique des dépendances)

Renovate surpasse Dependabot pour les projets pnpm : support pnpm 10 complet, workspace catalogs, groupement intelligent.

**Installation :** https://github.com/apps/renovate → Install

**renovate.json :**

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended",
    ":dependencyDashboard",
    ":semanticCommitTypeAll(chore)"
  ],
  "timezone": "Europe/Paris",
  "schedule": ["after 9pm on sunday"],
  "postUpdateOptions": ["pnpmDedupe"],
  "lockFileMaintenance": {
    "enabled": true,
    "automerge": true,
    "schedule": ["before 6am on monday"]
  },
  "vulnerabilityAlerts": {
    "enabled": true,
    "labels": ["security", "priority"],
    "automerge": true,
    "schedule": [],
    "minimumReleaseAge": null,
    "prCreation": "immediate"
  },
  "osvVulnerabilityAlerts": true,
  "packageRules": [
    {
      "description": "Automerge non-major dev dependencies",
      "matchDepTypes": ["devDependencies"],
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true,
      "automergeType": "branch",
      "minimumReleaseAge": "3 days"
    },
    {
      "description": "Group Nuxt ecosystem",
      "matchPackageNames": ["nuxt", "@nuxt/**", "nuxi", "@nuxtjs/**"],
      "groupName": "Nuxt packages"
    },
    {
      "description": "Group Vue ecosystem",
      "matchPackageNames": ["vue", "@vue/**", "vue-router", "pinia"],
      "groupName": "Vue packages"
    },
    {
      "description": "Major updates require dashboard approval",
      "matchUpdateTypes": ["major"],
      "dependencyDashboardApproval": true,
      "labels": ["dependencies", "breaking"]
    }
  ]
}
```

**Options clés :**

| Option | Valeur | Effet |
|--------|--------|-------|
| `postUpdateOptions: ["pnpmDedupe"]` | pnpm | Déduplique après chaque update |
| `minimumReleaseAge: "3 days"` | Prod deps | Protection supply chain |
| `vulnerabilityAlerts.minimumReleaseAge: null` | Security | Patches immédiats |
| `automergeType: "branch"` | Dev deps | Merge sans PR (moins de bruit) |

### Workflow Security Audit (GitHub Actions)

Workflow dédié à l'audit de sécurité des dépendances, avec upload SARIF vers GitHub Security.

```yaml
# .github/workflows/security.yml
name: Security Audit

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 6 * * 1'  # Lundi 6h UTC

permissions:
  contents: read
  security-events: write

env:
  PNPM_VERSION: '10'
  NODE_VERSION: '22'

jobs:
  dependency-audit:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      # Audit avec audit-ci (gating CI plus flexible que pnpm audit)
      - name: Security audit
        run: pnpm dlx audit-ci@^7 --config ./audit-ci.jsonc

      # Générer rapport SARIF pour GitHub Security tab
      - name: Generate SARIF report
        if: always()
        run: |
          pnpm audit --json > audit.json || true
          npx npm-audit-sarif -o audit.sarif audit.json

      - name: Upload to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: audit.sarif
          category: 'dependency-audit'

  license-check:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: Check licenses
        run: |
          npx license-checker --production \
            --onlyAllow "MIT;ISC;Apache-2.0;BSD-3-Clause;BSD-2-Clause;0BSD;CC0-1.0;Unlicense"
```

**Configuration audit-ci.jsonc :**

```jsonc
{
  "$schema": "https://github.com/IBM/audit-ci/raw/main/docs/schema.json",
  // Fail sur moderate et plus sévère
  "moderate": true,
  // Allowlist avec expiration (à documenter)
  "allowlist": [
    // "GHSA-xxxx-yyyy-zzzz"  // Exemple: dev-only, expires 2025-03-01
  ],
  "report-type": "important",
  "retry-count": 3
}
```

**Licences autorisées (permissives) :**

| Licence | Type | Safe pour projet propriétaire |
|---------|------|------------------------------|
| MIT, ISC, BSD-* | Permissive | ✅ Oui |
| Apache-2.0 | Permissive | ✅ Oui (attribution requise) |
| 0BSD, CC0-1.0, Unlicense | Public domain | ✅ Oui |
| GPL, AGPL, LGPL | Copyleft | ⚠️ Évaluation requise |

### Prettier avec TailwindCSS 4

**⚠️ Changement TailwindCSS v4 :** Utiliser `tailwindStylesheet` au lieu de `tailwindConfig` :

```json
{
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindStylesheet": "./app/assets/css/main.css",
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2
}
```

**Note :** Le plugin trie automatiquement les classes Tailwind selon l'ordre recommandé. Compatible avec @nuxt/eslint stylistic (pas de conflit).

### Release Please (CHANGELOG automatisé)

Alternative à semantic-release, plus légère et maintenue par Google. Parse les commits Conventional Commits et génère automatiquement le CHANGELOG + tags de version.

**Workflow GitHub Actions :**

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        with:
          release-type: node
```

**Configuration release-please-config.json :**

```json
{
  "packages": {
    ".": {
      "changelog-sections": [
        { "type": "feat", "section": "Features" },
        { "type": "fix", "section": "Bug Fixes" },
        { "type": "perf", "section": "Performance" },
        { "type": "docs", "section": "Documentation" },
        { "type": "chore", "section": "Maintenance", "hidden": true }
      ]
    }
  }
}
```

**Fonctionnement :**
1. À chaque push sur main, crée/met à jour une PR "Release"
2. La PR accumule les commits et prépare le CHANGELOG
3. Merge de la PR → crée un tag Git + GitHub Release automatiquement

**Prérequis :** Commits suivant Conventional Commits (`feat:`, `fix:`, `perf:`, etc.)

### Conventional Commits & Git Hooks

Configuration complète pour valider les messages de commit et linter les fichiers stagés avant chaque commit.

**Installation :**

```bash
# Dépendances
pnpm add -D husky lint-staged @commitlint/cli @commitlint/config-conventional @commitlint/types

# Initialiser Husky v9
pnpm exec husky init

# Créer les hooks (format v9 simplifié - pas de shebang)
echo "pnpm lint-staged" > .husky/pre-commit
echo 'pnpm exec commitlint --edit "$1"' > .husky/commit-msg

# Rendre exécutables
chmod +x .husky/pre-commit .husky/commit-msg
```

**Configuration Commitlint (commitlint.config.ts) :**

```typescript
import type { UserConfig } from '@commitlint/types'
import { RuleConfigSeverity } from '@commitlint/types'

const Configuration: UserConfig = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      RuleConfigSeverity.Error,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'build', 'ci', 'chore', 'revert'],
    ],
    // Scopes alignés sur structure Nuxt 4
    'scope-enum': [
      RuleConfigSeverity.Error,
      'always',
      [
        // app/ directory
        'app', 'components', 'composables', 'layouts', 'pages',
        'plugins', 'middleware', 'utils', 'assets',
        // server/ directory
        'server', 'api', 'server-routes',
        // Other directories
        'shared', 'content', 'public', 'config', 'types', 'deps',
      ],
    ],
    'scope-case': [RuleConfigSeverity.Error, 'always', 'kebab-case'],
    'header-max-length': [RuleConfigSeverity.Error, 'always', 100],
    'body-max-line-length': [RuleConfigSeverity.Error, 'always', 100],
  },
  ignores: [
    (commit) => commit.startsWith('Merge'),
    (commit) => /^v?\d+\.\d+\.\d+/.test(commit),
  ],
}

export default Configuration
```

**Configuration lint-staged (lint-staged.config.mjs) :**

```javascript
/** @type {import('lint-staged').Configuration} */
export default {
  'app/**/*.{ts,vue}': ['eslint --fix', 'prettier --write'],
  'server/**/*.ts': ['eslint --fix', 'prettier --write'],
  'shared/**/*.ts': ['eslint --fix', 'prettier --write'],
  '*.{json,md,yaml,css}': 'prettier --write',

  // ⚠️ Typecheck en pre-push, pas pre-commit (performance)
  // '**/*.{ts,vue}': () => 'nuxt typecheck',
}
```

**Script prepare conditionnel (package.json) :**

```json
{
  "scripts": {
    "prepare": "husky"
  }
}
```

**Alternative robuste pour CI (`.husky/install.mjs`) :**

```javascript
// Skip Husky en CI ou production
if (process.env.NODE_ENV === 'production' || process.env.CI === 'true') {
  process.exit(0)
}
const husky = (await import('husky')).default
console.log(husky())
```

Puis modifier package.json : `"prepare": "node .husky/install.mjs"`

**Variable d'environnement Cloudflare Pages :**

Dans le dashboard Cloudflare Pages (Settings → Environment variables) :

| Variable | Valeur | Environment |
|----------|--------|-------------|
| `HUSKY` | `0` | Production & Preview |

Cela désactive Husky pendant les builds CF Pages (évite erreur sur `husky install` dans le container de build).

**Anti-patterns Git Hooks :**

| Anti-pattern | Conséquence | Solution |
|--------------|-------------|----------|
| Typecheck complet en `pre-commit` | Hook >30s → bypass `--no-verify` | Déplacer en `pre-push` ou CI |
| `npx` au lieu de `pnpm exec` | Résolution incorrecte en monorepo pnpm | Toujours `pnpm exec` |
| Scope list >15 items | Friction cognitive, devs hésitent | 5-10 top-level max |
| Fichiers `.js` ambigus | `ERR_REQUIRE_ESM` avec `"type": "module"` | Utiliser `.mjs` ou `.cjs` explicitement |
| Hooks non exécutables | `permission denied` au commit | `chmod +x .husky/*` |

**Test de la configuration :**

```bash
# Tester commitlint
echo "feat: test conventional commits" | pnpm exec commitlint

# Tester un commit réel
git add . && git commit -m "chore(deps): configure conventional commits tooling"
```

**Commitizen — Commits interactifs (optionnel) :**

Pour guider les développeurs avec des prompts interactifs au lieu de taper manuellement le format :

```bash
pnpm add -D commitizen cz-conventional-changelog
```

Ajouter dans `package.json` :

```json
{
  "scripts": {
    "commit": "cz"
  },
  "config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  }
}
```

Usage : `pnpm commit` au lieu de `git commit` → assistant interactif qui guide type, scope, description.

**Alternative : simple-git-hooks (zero dépendances) :**

Si Husky semble trop complexe, **simple-git-hooks** offre une configuration entièrement dans `package.json` :

```bash
pnpm add -D simple-git-hooks lint-staged
pnpm exec simple-git-hooks
```

```json
{
  "scripts": {
    "prepare": "simple-git-hooks"
  },
  "simple-git-hooks": {
    "pre-commit": "pnpm exec lint-staged",
    "commit-msg": "pnpm exec commitlint --edit $1"
  }
}
```

⚠️ Après modification de la config, exécuter `pnpm exec simple-git-hooks` pour appliquer.

| Critère | Husky v9 | simple-git-hooks |
|---------|----------|------------------|
| Dépendances | 1 | 0 |
| Configuration | Fichiers `.husky/` | Dans `package.json` |
| Communauté | ⭐⭐⭐⭐⭐ (16M/sem) | ⭐⭐⭐ (260K/sem) |
| Recommandation | **Défaut** | Projets minimalistes |

### Stratégie Git Branching

**GitHub Flow** est la stratégie recommandée pour un développeur solo sur Nuxt 4 + Cloudflare Pages. GitFlow est considéré legacy pour les projets web en livraison continue.

**Workflow :**
1. Branche `main` toujours déployable (production)
2. Feature branches courtes (`feat/`, `fix/`, `refactor/`)
3. Pull Requests systématiques — même en solo (review code IA)
4. Squash & Merge pour historique linéaire

**Configuration Cloudflare Pages (Branch Deployments) :**

| Paramètre | Valeur |
|-----------|--------|
| Production branch | `main` |
| Preview branches | Custom |
| Include branches | `feat/*`, `fix/*`, `refactor/*` |
| Exclude branches | `dependabot/*`, `renovate/*` |

Chaque feature branch génère un preview deployment : `feat-nom-feature.projet.pages.dev`

**Squash & Merge recommandé :**
- Historique linéaire et propre
- Chaque commit = une feature complète
- Commits granulaires préservés dans la PR pour review
- Facilite la review du code généré par IA

### Optimisation Quota Builds (500/mois)

**Directive `[Skip CI]` :**

Ajouter dans le message de commit pour désactiver le build :

```bash
git commit -m "docs: update README [Skip CI]"
```

Variantes supportées (insensible à la casse) : `[CI Skip]`, `[CI-Skip]`, `[Skip-CI]`, `[CF-Pages-Skip]`

**Build Watch Paths (Dashboard CF → Settings → Builds) :**

| Action | Paths |
|--------|-------|
| **Include** | `app/*`, `server/*`, `content/*`, `shared/*`, `nuxt.config.ts`, `package.json` |
| **Exclude** | `docs/*`, `README.md`, `.github/*`, `*.md` |

⚠️ Si un push contient >3000 fichiers ou >20 commits, le path matching est bypassé.

**Build Caching (beta) :**
- Réduction temps de build ~50%
- Support pnpm natif
- Cache dépendances pendant 7 jours
- Activer dans Dashboard CF → Settings → Builds → Build cache

**Direct Upload (backup quota épuisé) :**

```bash
# Build local puis upload (ne compte PAS dans le quota)
pnpm run build
wrangler pages deploy .output/public --project-name=sebc-dev
```

### Versioning avec bumpp

Alternative légère à release-it, par Anthony Fu (créateur de Vitest). Aucun CHANGELOG — uniquement git tags.

**Installation :**

```bash
pnpm add -D bumpp
```

**Scripts package.json :**

```json
{
  "scripts": {
    "release": "bumpp",
    "release:patch": "bumpp patch",
    "release:minor": "bumpp minor",
    "release:major": "bumpp major",
    "release:alpha": "bumpp prerelease --preid alpha",
    "release:beta": "bumpp prerelease --preid beta",
    "release:rc": "bumpp prerelease --preid rc"
  }
}
```

**Pre-releases (ordre de précédence) :**

```
1.0.0-alpha.0 < 1.0.0-alpha.1 < 1.0.0-beta.0 < 1.0.0-rc.1 < 1.0.0
```

| Pre-release | Usage |
|-------------|-------|
| **alpha** | Tests internes précoces |
| **beta** | Fonctionnalités complètes, tests externes |
| **rc** | Release candidate, derniers ajustements |

**Usage interactif :**

```bash
pnpm release
# → Prompt interactif pour choisir le type de bump
# → Commit, tag et push automatiques
```

**Interprétation SemVer pour sites web :**

| Bump | Déclencheur |
|------|-------------|
| **MAJOR** | Refonte majeure, changement structure URLs |
| **MINOR** | Nouvelles fonctionnalités, nouvelles sections |
| **PATCH** | Corrections bugs, typos, ajustements CSS |

**Note :** Release Please (déjà documenté) génère un CHANGELOG automatique. bumpp est plus léger si le CHANGELOG n'est pas nécessaire.

**SemVer 0.x.x vs 1.x.x :**

La version **0.x.x** signale que l'API/interface n'est pas encore stable. Restez en 0.x.x pendant le développement actif et passez à **1.0.0** lorsque les fonctionnalités principales sont considérées stables pour la production.

### Format CHANGELOG (Keep a Changelog)

Si vous maintenez un CHANGELOG manuel ou utilisez commit-and-tag-version :

```markdown
# Changelog

Tous les changements notables de ce projet sont documentés dans ce fichier.

Le format est basé sur [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/),
et ce projet adhère au [Semantic Versioning](https://semver.org/lang/fr/).

## [Unreleased]

### Added
- Support du mode sombre

## [0.2.0] - 2025-12-30

### Added
- Composant de navigation responsive ([#15](https://github.com/user/repo/issues/15))
- Intégration recherche MiniSearch

### Fixed
- Erreur d'hydratation sur les pages SSG ([#23](https://github.com/user/repo/issues/23))

### Changed
- Migration vers Vue 3.5 pour les performances améliorées

## [0.1.0] - 2025-12-01

### Added
- Configuration initiale Nuxt 4
- Déploiement Cloudflare Pages

[Unreleased]: https://github.com/user/repo/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/user/repo/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/user/repo/releases/tag/v0.1.0
```

**Catégories standard :**

| Catégorie | Usage |
|-----------|-------|
| **Added** | Nouvelles fonctionnalités |
| **Changed** | Modifications de fonctionnalités existantes |
| **Deprecated** | Fonctionnalités qui seront supprimées |
| **Removed** | Fonctionnalités supprimées |
| **Fixed** | Corrections de bugs |
| **Security** | Corrections de vulnérabilités |

### Workflow GitHub Actions — Release sur Tag

Complément au workflow deploy.yml pour créer une GitHub Release automatique lors d'un tag :

```yaml
# .github/workflows/release-on-tag.yml
name: Release on Tag

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Extract changelog for this version
        id: changelog
        run: |
          VERSION=${GITHUB_REF#refs/tags/v}
          CHANGELOG=$(awk "/^## \[${VERSION}\]/{flag=1; next} /^## \[/{flag=0} flag" CHANGELOG.md)
          echo "changelog<<EOF" >> $GITHUB_OUTPUT
          echo "$CHANGELOG" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          body: ${{ steps.changelog.outputs.changelog }}
          prerelease: ${{ contains(github.ref, 'alpha') || contains(github.ref, 'beta') || contains(github.ref, 'rc') }}
```

Ce workflow extrait automatiquement la section du CHANGELOG correspondant au tag et crée une GitHub Release. Les tags contenant `alpha`, `beta` ou `rc` sont marqués comme pre-release.

### Review Code IA — Vigilance Sécurité

Le code généré par LLMs présente des risques spécifiques que les PRs et commits atomiques permettent de mitiger.

**Statistiques 2024-2025 :**
- **~20% des packages recommandés par LLMs sont des hallucinations** (n'existent pas sur npm)
- **42% des snippets IA contiennent des erreurs** de nature diverse
- Risque "slopsquatting" : attaquant crée un package malveillant avec le nom halluciné

**Red flags prioritaires :**

| Red flag | Action |
|----------|--------|
| Package inconnu ajouté | Vérifier existence sur npmjs.com AVANT commit |
| API keys hardcodées | Refuser, utiliser `.env` + `runtimeConfig` |
| Méthode/fonction inexistante | Vérifier la doc de la librairie |
| Absence validation inputs | Ajouter validation Zod |
| Gestion erreurs manquante | Ajouter try/catch appropriés |

**Vérification systématique :**

```bash
# Après chaque ajout de dépendance
pnpm audit

# Vérifier qu'un package existe (avant d'ajouter)
npm view <package-name> version
```

**Pattern de traçabilité (optionnel) :**

```bash
# Indiquer la provenance IA dans le commit
git commit -m "feat(components): add UserCard (AI-generated base)"

# Après review manuelle
git commit -m "refactor(components): optimize UserCard (manual review)"
```

Cette pratique facilite les audits de sécurité ultérieurs et la compréhension de l'historique.

**Règle d'or :** Traiter l'IA comme un développeur junior talentueux mais inexpérimenté. Générer le code par petits incréments, demander des explications, et vérifier systématiquement chaque dépendance.

## Limites Cloudflare Free Tier

### Cloudflare Pages

| Limite | Valeur Free Tier |
|--------|------------------|
| Builds par mois | **500** (compte entier) |
| Builds concurrents | 1 |
| Timeout build | 20 minutes |
| Fichiers par déploiement | **20,000** |
| Taille max fichier | **25 MiB** |
| Projets par compte | 100 |
| Bandwidth | **Illimité** |
| Preview deployments | Illimité |

### Cloudflare D1

| Limite | Valeur Free Tier |
|--------|------------------|
| Stockage | 5 GB |
| Taille max par DB | **500 MB** |
| Lectures/jour | 5 millions rows_read |
| Écritures/jour | 100 000 rows_written |

**Note :** Les Direct Uploads via GitHub Actions (wrangler-action) ne comptent pas dans le quota 500 builds/mois.

## Sécurité Cloudflare (SSG)

### Rate Limiting WAF (gratuit)

Le rate limiting applicatif est **impossible en SSG pur** sans serveur. Utiliser le WAF Cloudflare (1 règle gratuite) :

**Configuration via Dashboard** (Security → WAF → Rate limiting rules) :

| Paramètre | Valeur |
|-----------|--------|
| Expression | `(http.request.uri.path contains "/api/")` |
| Caractéristique | IP Address |
| Seuil | 100 requêtes / minute |
| Action | Managed Challenge |

**Alternatives selon le cas :**

| Protection | Solution | Coût |
|------------|----------|------|
| Protection globale bots | Bot Fight Mode | Gratuit |
| Rate limiting API | WAF Rate Limiting | Gratuit (1 règle) |
| Formulaires | Cloudflare Turnstile | Gratuit |
| API endpoints | Pages Functions + Workers | Inclus |

### Cloudflare Turnstile (Formulaires)

Alternative CAPTCHA gratuite et respectueuse de la vie privée. Utiliser pour les formulaires de contact, commentaires, newsletter.

**1. Configuration Dashboard Cloudflare :**
- Naviguer vers Turnstile → Add Site
- Récupérer `SITE_KEY` (public) et `SECRET_KEY` (privé)

**2. Composant Vue côté client :**

```vue
<!-- app/components/TurnstileWidget.vue -->
<script setup lang="ts">
const siteKey = useRuntimeConfig().public.turnstileSiteKey

const emit = defineEmits<{
  verified: [token: string]
  error: []
}>()

onMounted(() => {
  if (window.turnstile) {
    window.turnstile.render('#turnstile-container', {
      sitekey: siteKey,
      callback: (token: string) => emit('verified', token),
      'error-callback': () => emit('error')
    })
  }
})
</script>

<template>
  <div id="turnstile-container" />
</template>
```

**3. Validation côté Pages Function :**

```typescript
// functions/api/contact.ts
interface TurnstileResponse {
  success: boolean
  'error-codes'?: string[]
}

export async function onRequestPost(context: EventContext<Env, string, unknown>) {
  const formData = await context.request.formData()
  const token = formData.get('cf-turnstile-response') as string

  // Validation Turnstile
  const verification = await fetch(
    'https://challenges.cloudflare.com/turnstile/v0/siteverify',
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        secret: context.env.TURNSTILE_SECRET_KEY,
        response: token,
        remoteip: context.request.headers.get('cf-connecting-ip')
      })
    }
  ).then(r => r.json() as Promise<TurnstileResponse>)

  if (!verification.success) {
    return new Response(JSON.stringify({ error: 'Vérification échouée' }), {
      status: 403,
      headers: { 'Content-Type': 'application/json' }
    })
  }

  // Traitement du formulaire...
  return new Response(JSON.stringify({ success: true }), {
    headers: { 'Content-Type': 'application/json' }
  })
}
```

**4. Configuration nuxt.config.ts :**

```typescript
runtimeConfig: {
  turnstileSecretKey: '',  // TURNSTILE_SECRET_KEY env var
  public: {
    turnstileSiteKey: ''   // NUXT_PUBLIC_TURNSTILE_SITE_KEY env var
  }
}
```

**5. Script Turnstile dans app.vue :**

```typescript
useHead({
  script: [{
    src: 'https://challenges.cloudflare.com/turnstile/v0/api.js',
    async: true,
    defer: true
  }]
})
```

**Note :** Turnstile est prévu pour la phase post-MVP (formulaire de contact, commentaires).

## Décisions Architecturales du Starter

**Language & Runtime:**
- TypeScript strict par défaut
- Node.js 22 LTS "Jod" (version stable recommandée par Cloudflare Pages)
- pnpm 10+ comme package manager (⚠️ lifecycle scripts désactivés par défaut - config via `package.json` champ `pnpm` ou `pnpm-workspace.yaml`)

**Styling Solution:**
- TailwindCSS 4.1.17+ via @tailwindcss/vite (⚠️ @nuxtjs/tailwindcss@6.14.0 supporte TW4, mais @tailwindcss/vite recommandé pour nouveaux projets CSS-first)
- Configuration CSS-native avec @theme (remplace tailwind.config.js)
- Tokens sémantiques oklch pour dark-first design

**Build Tooling:**
- Vite comme bundler (intégré Nuxt)
- SSG mode pour génération statique
- MiniSearch index généré post-build (script `scripts/generate-search-index.mjs`)
- Cloudflare Pages compatible (preset cloudflare_pages)

**UI Components:**
- shadcn-vue 2.4.3+ avec Reka UI primitives (rebrand de Radix Vue depuis février 2025)
- Composants dans app/components/ui/
- Tests a11y @axe-core/playwright 4.11.0 en CI (couverture WCAG incomplète native)
- Tests a11y unitaires vitest-axe pour validation composants isolés

**Content Management:**
- Nuxt Content 3.10.0+ avec collections
- Fichiers MDC dans content/
- Validation Zod 4 (~5KB gzip / ~10KB min) ou `zod/mini` (~2KB gzip / ~5KB min) + Standard Schema natif
- ⚠️ Import Zod 4 : `import { z } from 'zod/v4'` (API moderne avec top-level validators)
- ⚠️ `@zod/mini` npm package **déprécié** → utiliser `import * as z from 'zod/mini'` (import path)
- Migration automatique : `npx zod-v3-to-v4 path/to/tsconfig.json`

**Search:**
- MiniSearch 7.x (~7KB minified, zéro dépendance)
- Index JSON pré-généré au build time
- Boosting par champ, fuzzy search, prefix search natifs
- Stemming FR via option `stemmer` (snowball-stemmers)

**Internationalization:**
- @nuxtjs/i18n v10.2.1+
- Strategy: prefix_except_default (/about en, /fr/about fr)
- Propriété `language` obligatoire pour hreflang auto
- Lazy-loading des traductions

**Performance Optimization:**
- Hydratation lazy native Nuxt 4 (hydrate-on-visible, hydrate-on-idle)
- nuxt-vitalizer 2.0.0+ pour optimisation LCP
- features.inlineStyles pour réduction CLS

## Configuration nuxt.config.ts de Base

```typescript
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  // Nuxt 4 par défaut, plus besoin de future.compatibilityVersion: 4
  compatibilityDate: '2025-07-15',

  // ORDRE CRITIQUE: @nuxtjs/seo AVANT @nuxt/content
  modules: [
    '@nuxt/image',
    '@nuxtjs/i18n',
    '@nuxtjs/seo',         // Suite SEO complète - AVANT @nuxt/content
    '@nuxt/content',       // APRÈS @nuxtjs/seo
    'nuxt-llms',           // Génération automatique /llms.txt
    'nuxt-vitalizer',
    'shadcn-nuxt',
  ],

  css: ['~/assets/css/main.css'],

  vite: {
    plugins: [tailwindcss()],
  },

  // Configuration site (requise par @nuxtjs/seo)
  site: {
    url: 'https://sebc.dev',
    name: 'sebc.dev',
    description: 'Blog technique sur le développement web',
    defaultLocale: 'en',
  },

  // Configuration OG Image (inclus dans @nuxtjs/seo)
  ogImage: {
    zeroRuntime: true,  // ESSENTIEL pour SSG pur
  },

  // Optimisations performances Lighthouse
  features: {
    inlineStyles: true, // Remplace experimental.inlineSSRStyles (réduit CLS 0.77 → 0.00)
  },

  i18n: {
    locales: [
      { code: 'en', language: 'en-US', name: 'English' },
      { code: 'fr', language: 'fr-FR', name: 'Français' },
    ],
    defaultLocale: 'en',
    strategy: 'prefix_except_default', // /about (en), /fr/about (fr)
    baseUrl: 'https://sebc.dev',
  },

  shadcn: {
    prefix: '',
    componentDir: './app/components/ui',
  },

  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      autoSubfolderIndex: false, // Évite les doubles redirects Cloudflare
      crawlLinks: true,
      routes: ['/rss.xml'], // Pre-render RSS feed
    },
  },

  routeRules: {
    // Cache statique agressif pour assets buildés par Nuxt (1 an, immutable)
    '/_nuxt/**': { headers: { 'cache-control': 'public, max-age=31536000, immutable' } },
    // Cache court pour HTML SSG (permet rollbacks rapides)
    '/**': { headers: { 'cache-control': 'public, max-age=0, must-revalidate' } },
  },

  // MiniSearch: index généré via script postgenerate
  // "scripts": { "generate": "nuxt generate", "postgenerate": "node scripts/generate-search-index.mjs" }
})
```

## Configuration TailwindCSS 4

```css
/* app/assets/css/main.css */
@import "tailwindcss";

/* Scanner les fichiers Markdown pour les classes Tailwind dans les attributs MDC */
@source "../../../content/**/*";

@theme {
  /* Typographie */
  --font-display: "Satoshi", "sans-serif";
  --font-mono: "JetBrains Mono", "monospace";
  
  /* Palette oklch dark-first */
  --color-primary: oklch(0.92 0.19 114.08);
  --color-secondary: oklch(0.85 0.15 242.32);
  --color-background: oklch(0.13 0.01 264.05);
  --color-foreground: oklch(0.98 0.01 264.05);
  
  /* Breakpoints personnalisés */
  --breakpoint-3xl: 1920px;
}
```

## Configuration pnpm 10

**Option 1 - package.json (Recommandée) :**
```json
{
  "pnpm": {
    "onlyBuiltDependencies": ["esbuild", "sharp"]
  }
}
```

**Option 2 - pnpm-workspace.yaml (Recommandée pour sécurité) :**
```yaml
# pnpm-workspace.yaml - Configuration pnpm 10 avec sécurité
packages:
  - '.'

onlyBuiltDependencies:
  - esbuild
  - sharp

# ══════════════════════════════════════════════════════════════
# SÉCURITÉ SUPPLY CHAIN (pnpm 10.16+)
# ══════════════════════════════════════════════════════════════

# Délai avant installation de nouveaux packages (protection supply chain)
# 1440 minutes = 24 heures - bloque packages malveillants récents
minimumReleaseAge: 1440

# Bloque les dépendances transitives depuis git URLs ou local paths
# Empêche injection de code malveillant via subdeps exotiques
blockExoticSubdeps: true  # pnpm 10.26.0+

# ══════════════════════════════════════════════════════════════
# AUDIT - Gestion des faux positifs
# ══════════════════════════════════════════════════════════════

auditConfig:
  # CVEs à ignorer (après évaluation manuelle du risque)
  ignoreCves: []
    # - CVE-2022-36313  # Exemple : vulnerability non exploitable dans notre contexte

  # GitHub Security Advisories à ignorer
  ignoreGhsas: []
    # - GHSA-42xw-2xvc-qx8m  # Exemple : dev dependency uniquement

# Force versions non-vulnérables des dépendances transitives
overrides:
  # "lodash@<4.17.21": "^4.17.21"  # Exemple : forcer version patchée
```

**Note importante :** La syntaxe `.npmrc` avec `pnpm.onlyBuiltDependencies[]` est **incorrecte pour pnpm 10**. Utiliser `package.json` (champ `pnpm`) ou `pnpm-workspace.yaml` à la place.

**Commandes pnpm audit essentielles :**

```bash
# Audit basique (fail si high/critical)
pnpm audit --audit-level=high

# Ignorer les CVEs sans fix disponible
pnpm audit --audit-level=high --ignore-unfixable

# Générer JSON pour CI
pnpm audit --json > audit.json

# ⚠️ IMPORTANT : --fix modifie pnpm.overrides, pas le lockfile
pnpm audit --fix && pnpm install  # Les deux commandes sont requises
```

## Arborescence Projet Nuxt 4

```
sebc-dev/
├── app/                          # srcDir (nouveau défaut Nuxt 4)
│   ├── assets/
│   │   └── css/
│   │       └── main.css          # TailwindCSS entry point
│   ├── components/
│   │   ├── ui/                   # shadcn-vue (Reka UI)
│   │   ├── content/              # ArticleCard, TableOfContents
│   │   ├── layout/               # TheHeader, TheFooter
│   │   └── search/               # SearchCommand, SearchFilters
│   ├── composables/
│   │   └── index.ts              # Re-export si sous-dossiers
│   ├── layouts/
│   ├── pages/
│   ├── plugins/
│   │   └── ssr-width.ts
│   └── utils/
│       └── cn.ts                 # Utilitaire shadcn class merge
│
├── shared/                       # Code isomorphe partagé app/server
│   ├── types/                    # ✅ Auto-importé
│   │   └── article.ts
│   └── utils/                    # ✅ Auto-importé
│       └── validation.ts
│
├── content/                      # Nuxt Content 3 (RACINE obligatoire)
│   ├── fr/
│   └── en/
│
├── server/                       # Nitro (RACINE obligatoire)
│   ├── api/                      # Build-time uniquement en SSG
│   ├── routes/
│   │   └── rss.xml.ts
│   └── plugins/
│       └── llms-extend.ts
│
├── public/                       # Assets statiques (RACINE obligatoire)
│   ├── fonts/
│   ├── images/
│   ├── _headers                  # Headers Cloudflare
│   └── favicon.ico
│
├── nuxt.config.ts
├── content.config.ts             # Configuration Nuxt Content
├── wrangler.toml                 # Configuration D1 Cloudflare
├── pnpm-workspace.yaml           # Configuration pnpm 10
└── package.json
```

### Dossier `shared/` (Nuxt 3.14+)

Le dossier `shared/` permet de partager du code entre `app/` (Vue) et `server/` (Nitro). **Seuls `shared/utils/` et `shared/types/` sont auto-importés** — les autres fichiers nécessitent un import explicite via `#shared/path`.

```typescript
// shared/types/article.ts - Auto-importé, accessible partout
export interface Article {
  title: string
  slug: string
  pillar: 'ai' | 'engineering' | 'ux'
}

// shared/utils/formatReadingTime.ts - Auto-importé
export const formatReadingTime = (minutes: number, locale: string) =>
  locale === 'fr' ? `${minutes} min de lecture` : `${minutes} min read`
```

**⚠️ Restrictions importantes :**
- Le code dans `shared/` ne peut **PAS** importer de dépendances Vue ou Nitro spécifiques
- Uniquement du TypeScript isomorphe pur (types, fonctions utilitaires)
- Accès via alias `#shared` : `import { Article } from '#shared/types/article'`

| Dossier | Auto-importé | Alias |
|---------|--------------|-------|
| `shared/types/` | ✅ Oui | `#shared/types/*` |
| `shared/utils/` | ✅ Oui | `#shared/utils/*` |
| `shared/other/` | ❌ Non | `#shared/other/*` (import explicite) |

### Composables dans sous-dossiers (Piège courant)

**⚠️ Les composables dans des sous-dossiers de `composables/` ne sont PAS auto-scannés.**

```
app/composables/
├── useAuth.ts          # ✅ Auto-importé
├── useBlog.ts          # ✅ Auto-importé
└── search/
    └── useSearch.ts    # ❌ PAS auto-importé !
```

**Solutions :**

**Option 1 : Re-export dans `composables/index.ts`** (Recommandé)
```typescript
// app/composables/index.ts
export { useSearch } from './search/useSearch'
```

**Option 2 : Configuration `imports.dirs`**
```typescript
// nuxt.config.ts
imports: {
  dirs: ['composables/**']
}
```

## Hydratation Lazy Native (remplace nuxt-delay-hydration)

```vue
<template>
  <!-- Hydrate quand visible dans le viewport -->
  <LazyExpensiveComponent hydrate-on-visible />
  
  <!-- Hydrate pendant l'idle time du navigateur -->
  <LazyHeavyComponent hydrate-on-idle />
  
  <!-- Hydrate après un délai fixe (2s) -->
  <LazyDelayedComponent :hydrate-after="2000" />
  
  <!-- Hydrate sur interaction (hover, click, focus) -->
  <LazyInteractiveComponent hydrate-on-interaction />
</template>
```

## Migration Reka UI

```typescript
// Ancien import (legacy)
import { TooltipRoot, TooltipTrigger } from 'radix-vue'

// Nouveau import (recommandé depuis février 2025)
import { TooltipRoot, TooltipTrigger } from 'reka-ui'
```

```css
/* Variables CSS également renommées */
/* Ancien */
.element { color: var(--radix-tooltip-trigger-color); }

/* Nouveau */
.element { color: var(--reka-tooltip-trigger-color); }
```

## Paramètres Cloudflare Pages

**À désactiver dans le dashboard Cloudflare** (causent problèmes d'hydratation) :
- ❌ Rocket Loader™
- ❌ Mirage (Image Optimization)
- ❌ Email Address Obfuscation
- ❌ Auto-minification (déprécié août 2024 → minifier au build)

**À activer manuellement pour domaines custom** :
- ✅ HTTP/3 (activé par défaut sur `*.pages.dev` uniquement)

**Configuration Build** :
- Framework preset: `Nuxt.js`
- Build command: `pnpm run build`
- Build output directory: `.output/public`
- Node version: `22 LTS` (version stable recommandée)

**Avantages vérifiés (tier gratuit)** :
- Bande passante illimitée (pas de frais d'egress)
- Early Hints activé par défaut → **+30% LCP** pour nouveaux visiteurs

### Fichier public/_headers

Configuration cache, sécurité et Early Hints optimisée pour Cloudflare Pages :

```
# public/_headers

# ═══════════════════════════════════════════════════════════════════
# SÉCURITÉ GLOBALE
# ═══════════════════════════════════════════════════════════════════
/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  X-XSS-Protection: 0
  Referrer-Policy: strict-origin-when-cross-origin
  Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
  Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=(), bluetooth=(), accelerometer=(), gyroscope=()
  Cross-Origin-Opener-Policy: same-origin-allow-popups
  Cross-Origin-Resource-Policy: same-origin
  Content-Security-Policy: default-src 'self'; script-src 'self' 'strict-dynamic'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob: https:; font-src 'self' data:; connect-src 'self'; frame-ancestors 'none'; base-uri 'none'; form-action 'self'; object-src 'none'; upgrade-insecure-requests

# ═══════════════════════════════════════════════════════════════════
# HTML STATIQUES - Cache court avec revalidation
# ═══════════════════════════════════════════════════════════════════
/*.html
  Cache-Control: public, max-age=0, must-revalidate

/
  Cache-Control: public, max-age=0, must-revalidate

# ═══════════════════════════════════════════════════════════════════
# ASSETS JS/CSS HASHÉS - Cache agressif immutable (1 an)
# ═══════════════════════════════════════════════════════════════════
/_nuxt/*.js
  Cache-Control: public, max-age=31536000, immutable

/_nuxt/*.css
  Cache-Control: public, max-age=31536000, immutable

# ═══════════════════════════════════════════════════════════════════
# IMAGES OPTIMISÉES - Immutable car hashées
# ═══════════════════════════════════════════════════════════════════
/_nuxt/*.webp
  Cache-Control: public, max-age=31536000, immutable

/_nuxt/*.avif
  Cache-Control: public, max-age=31536000, immutable

# ═══════════════════════════════════════════════════════════════════
# IMAGES NON-FINGERPRINTED - 30 jours
# ═══════════════════════════════════════════════════════════════════
/images/*
  Cache-Control: public, max-age=2592000

/*.webp
  Cache-Control: public, max-age=2592000

/*.avif
  Cache-Control: public, max-age=2592000

/*.svg
  Cache-Control: public, max-age=2592000

/*.png
  Cache-Control: public, max-age=2592000

/*.jpg
  Cache-Control: public, max-age=2592000

/favicon.ico
  Cache-Control: public, max-age=86400

# ═══════════════════════════════════════════════════════════════════
# FONTS - CORS obligatoire même en self-hosting
# ═══════════════════════════════════════════════════════════════════
/fonts/*
  Cache-Control: public, max-age=31536000, immutable
  Access-Control-Allow-Origin: *

/*.woff2
  Cache-Control: public, max-age=31536000, immutable
  Access-Control-Allow-Origin: *

# ═══════════════════════════════════════════════════════════════════
# EARLY HINTS (103) - Preload ressources critiques pour LCP
# Active uniquement si Early Hints activé dans CF Dashboard
# ═══════════════════════════════════════════════════════════════════
/
  Link: </fonts/Satoshi-Variable.woff2>; rel=preload; as=font; type=font/woff2; crossorigin

# ═══════════════════════════════════════════════════════════════════
# SEO & CRAWLERS
# ═══════════════════════════════════════════════════════════════════
/sitemap.xml
  Cache-Control: public, max-age=3600

/robots.txt
  Cache-Control: public, max-age=3600

# llms.txt pour assistants IA (nuxt-llms)
/llms.txt
  Content-Type: text/plain; charset=utf-8
  Cache-Control: public, max-age=3600
  X-Robots-Tag: noindex

/llms-full.txt
  Content-Type: text/plain; charset=utf-8
  Cache-Control: public, max-age=3600
  X-Robots-Tag: noindex

# ═══════════════════════════════════════════════════════════════════
# PREVIEW DEPLOYMENTS - Bloquer indexation
# ═══════════════════════════════════════════════════════════════════
https://:project.pages.dev/*
  X-Robots-Tag: noindex

https://*.:project.pages.dev/*
  X-Robots-Tag: noindex
```

**Security Headers expliqués :**

| Header | Fonction | Valeur |
|--------|----------|--------|
| `X-Content-Type-Options` | Empêche MIME-sniffing | `nosniff` |
| `X-Frame-Options` | Protection clickjacking | `DENY` (bloque tout iframe) |
| `X-XSS-Protection` | Désactive filtre XSS navigateur (déprécié, peut introduire failles) | `0` |
| `Referrer-Policy` | Contrôle info envoyée en referer | `strict-origin-when-cross-origin` |
| `Strict-Transport-Security` | Force HTTPS (2 ans) | `max-age=63072000; includeSubDomains; preload` |
| `Permissions-Policy` | Désactive APIs sensibles non utilisées | Liste des APIs désactivées |
| `Cross-Origin-Opener-Policy` | Protection cross-origin | `same-origin-allow-popups` (OAuth compatible) |
| `Cross-Origin-Resource-Policy` | Protège ressources d'embedding externe | `same-origin` |
| `Cross-Origin-Embedder-Policy` | Isolation cross-origin | `unsafe-none` (défaut recommandé SSG) |
| `Content-Security-Policy` | Protection XSS majeure | Voir détail ci-dessous |

**Notes importantes :**
- **`interest-cohort=()` obsolète** : Google a abandonné FLoC en 2022. Chrome v109+ ignore cette directive Permissions-Policy — ne pas l'inclure.
- **COEP `unsafe-none`** : Utiliser `require-corp` uniquement si SharedArrayBuffer nécessaire (jeux, audio processing). Pour un blog SSG, `unsafe-none` évite de casser les embeds externes.

**CSP (Content-Security-Policy) détaillé :**

| Directive | Valeur | Explication |
|-----------|--------|-------------|
| `default-src` | `'self'` | Défaut restrictif |
| `script-src` | `'self' 'strict-dynamic'` | Scripts locaux + chargement en chaîne |
| `style-src` | `'self' 'unsafe-inline'` | ⚠️ `unsafe-inline` requis pour shadcn-vue/Reka UI |
| `img-src` | `'self' data: blob: https:` | Images locales, data URIs, blobs, HTTPS externes |
| `font-src` | `'self' data:` | Fonts self-hosted uniquement |
| `connect-src` | `'self'` | XHR/Fetch locaux uniquement |
| `frame-ancestors` | `'none'` | Équivalent moderne de X-Frame-Options DENY |
| `base-uri` | `'none'` | Protection base tag injection |
| `form-action` | `'self'` | Formulaires soumis localement uniquement |
| `object-src` | `'none'` | Bloque plugins (Flash, etc.) |
| `upgrade-insecure-requests` | activé | Force HTTPS pour ressources HTTP |

**⚠️ Limitation CSP avec shadcn-vue :**

Reka UI (primitives de shadcn-vue) utilise des styles inline pour le positionnement, nécessitant `style-src 'unsafe-inline'`. Composants affectés :

| Composant | CSP Compatible | Raison |
|-----------|---------------|--------|
| Button, Input, Card | ✅ Oui | Classes CSS uniquement |
| Toast/Sonner | ❌ Non | Styles inline positionnement |
| Dialog, Sheet | ❌ Non | Overlay positioning |
| ScrollArea | ❌ Non | Comportement scroll Reka UI |
| Tabs | ⚠️ Partiel | Positionnement indicateur |

**Autres headers :**

| Header | Fonction | Impact |
|--------|----------|--------|
| `Link: rel=preload` | Early Hints 103 pour fonts | LCP -30% |
| `Access-Control-Allow-Origin: *` | CORS pour fonts cross-origin | Obligatoire |
| `immutable` | Jamais revalidé par le CDN | Performance |

**⚠️ Notes importantes :**
- **HSTS preload** : Ne pas activer si pas prêt à s'engager HTTPS sur tous les sous-domaines (retrait prend des mois). Soumettre à hstspreload.org après déploiement.
- **HSTS + Cloudflare** : Si HSTS activé dans CF Dashboard (SSL/TLS → Edge Certificates), il écrase le header origin. Configurer à un seul endroit.
- **Early Hints** : Doit être activé dans Cloudflare Dashboard (Speed → Optimization → Protocol Optimization)
- **CORS fonts** : `crossorigin` obligatoire même en self-hosting pour preload
- **Preview indexation** : Remplacer `:project` par le nom réel du projet CF Pages
- **CSP testing** : Utiliser `Content-Security-Policy-Report-Only` en dev pour identifier les violations avant d'enforcer

## Stratégies de Cache Avancées Cloudflare

### Headers Cache-Control multi-niveaux

Cloudflare propose trois headers distincts pour contrôler le cache à différents niveaux :

| Header | Contrôle | Passé downstream ? |
|--------|----------|-------------------|
| `Cloudflare-CDN-Cache-Control` | Edge Cloudflare uniquement | Non |
| `CDN-Cache-Control` | Tous les CDN | Oui |
| `Cache-Control` | Navigateurs + shared caches | Oui |

**Cas d'usage pratique** : Quand le navigateur doit revalider fréquemment mais l'edge peut cacher plus longtemps :

```
# API data : browser 60s, edge 1h
/api/data.json
  Cache-Control: max-age=60
  Cloudflare-CDN-Cache-Control: max-age=3600
```

Le navigateur voit un TTL de 60 secondes ; l'edge Cloudflare cache pendant 1 heure, réduisant la charge sur l'origin.

### Pattern stale-while-revalidate pour HTML (Alternative)

Pour les sites statiques mis à jour rarement, `stale-while-revalidate` offre un excellent compromis — réponses instantanées avec vérification de fraîcheur en arrière-plan :

```
/*.html
  Cache-Control: max-age=600, stale-while-revalidate=86400
```

**Comportement :**
- **0-10 minutes** : Servi depuis le cache immédiatement (frais)
- **10 min à 24h** : Servi immédiatement (stale), revalidation en background
- **Après 24h** : Requête réseau complète obligatoire

**Trade-off** : Les utilisateurs peuvent voir du contenu jusqu'à 10 minutes obsolète au premier chargement. Acceptable pour blogs et documentation, à éviter pour e-commerce ou contenu temps-réel.

**Alternative stricte (configuration actuelle)** : `max-age=0, must-revalidate` force la revalidation à chaque requête mais exploite les ETags pour des réponses 304 (~200 bytes vs HTML complet).

### Directive `immutable` expliquée

La directive `immutable` dans `Cache-Control` empêche le navigateur de revalider même lors d'un Shift+Refresh :

```
Cache-Control: public, max-age=31536000, immutable
```

**Support navigateurs :**
- ✅ Firefox 49+
- ✅ Safari 11+
- ✅ Chrome/Edge modernes (ignoré mais inoffensif)

**Quand utiliser** : Uniquement pour les assets avec hash dans le nom de fichier (`/_nuxt/*.js`, fonts WOFF2). Le changement d'URL garantit le cache-busting automatique.

### CF-Cache-Status (Debugging)

Valeurs du header `CF-Cache-Status` retourné par Cloudflare :

| Status | Signification |
|--------|---------------|
| `HIT` | Servi depuis l'edge cache Cloudflare |
| `MISS` | Récupéré de l'origin, maintenant en cache |
| `BYPASS` | Réponse avec `no-cache` ou `private` |
| `DYNAMIC` | HTML (non caché par défaut) |
| `REVALIDATED` | Contenu stale revalidé via ETag |
| `EXPIRED` | Contenu expiré, nouvelle requête origin |
| `UPDATING` | stale-while-revalidate en cours |

**Note importante** : HTML n'est pas caché à l'edge par défaut (statut `DYNAMIC`). Pour un site SSG pur, c'est acceptable car l'origin est Cloudflare Pages lui-même (rapide).

### Validation du cache en production

Commandes curl pour vérifier le comportement du cache après déploiement :

```bash
# Vérifier les headers d'un asset fingerprinted
curl -I https://sebc.dev/_nuxt/DC5HVSK5.js

# Réponse attendue :
# cache-control: public, max-age=31536000, immutable
# cf-cache-status: HIT (après première requête)

# Vérifier les headers HTML
curl -I https://sebc.dev/

# Réponse attendue :
# cache-control: public, max-age=0, must-revalidate
# cf-cache-status: DYNAMIC

# Vérifier les fonts avec CORS
curl -I https://sebc.dev/fonts/Satoshi-Variable.woff2

# Réponse attendue :
# cache-control: public, max-age=31536000, immutable
# access-control-allow-origin: *
# cf-cache-status: HIT
```

**Impact Core Web Vitals** : Un caching correct améliore significativement le LCP pour les visiteurs récurrents — les assets fingerprinted se chargent depuis le disk cache (~1ms) au lieu du réseau (~50-200ms par ressource).

### Cache images non-fingerprinted

Pour les images dans `/public/images/` (non transformées par `@nuxt/image`) :

```
# Images statiques non-hashées - 30 jours
/images/*
  Cache-Control: public, max-age=2592000

/*.webp
  Cache-Control: public, max-age=2592000

/*.avif
  Cache-Control: public, max-age=2592000

/*.svg
  Cache-Control: public, max-age=2592000

/*.png
  Cache-Control: public, max-age=2592000

/*.jpg
  Cache-Control: public, max-age=2592000
```

**Note** : Les images transformées par `@nuxt/image` dans `/_ipx/` ou `/_nuxt/` sont déjà hashées et couvertes par le cache immutable 1 an. Cette règle concerne uniquement les images statiques non-optimisées.

| Type d'image | Chemin | Cache recommandé |
|--------------|--------|------------------|
| Optimisées Nuxt | `/_ipx/*`, `/_nuxt/*.webp` | 1 an, immutable |
| Statiques | `/images/*` | 30 jours |
| Favicon | `/favicon.ico` | 1 jour (peut changer lors rebrand) |

## Versions des Packages (Décembre 2025)

| Package | Version recommandée | Notes |
|---------|-------------------|-------|
| nuxt | 4.2.2 | Version stable actuelle (9 déc. 2025) |
| @nuxt/content | 3.10.0+ | Pleinement compatible Nuxt 4; `asSeoCollection()` requis |
| @nuxtjs/seo | **2.0.0+** | Suite SEO complète (sitemap, robots, schema.org, og-image, link-checker) |
| tailwindcss | 4.1.17 | Version stable actuelle (6 nov. 2025) |
| @tailwindcss/vite | **4.1.18** | À synchroniser avec TailwindCSS |
| shadcn-vue | 2.4.3+ | Utilise Reka UI; ⚠️ path aliases `@/` → `app/` à ajuster |
| reka-ui | **2.7.0** | Rebrand de Radix Vue (février 2025); variables CSS `--reka-*` |
| @nuxtjs/i18n | 10.2.1+ | Compatible Nuxt 4; experimental `strictSeo` mode |
| zod | 4.x | ~5KB gzip / ~10KB min; `zod/mini` (~2KB gzip / ~5KB min, réduction 64%) disponible |
| minisearch | 7.1.1+ | ~7KB minified, zéro dépendance, index JSON pré-généré |
| nuxt-vitalizer | 2.0.0+ | DelayHydration component supprimé → macros natives |
| nuxt-security | 2.x | CSP hash generation SSG, headers OWASP |
| @axe-core/playwright | 4.11.0+ | Tests a11y standard |
| nuxt-llms | latest | Génération automatique /llms.txt avec @nuxt/content ^3.2.0 |
| pnpm | 10.26.2+ | Lifecycle scripts désactivés par défaut (config via package.json ou pnpm-workspace.yaml) |
| node | 22 LTS | Version stable Cloudflare Pages (Node 24 non recommandé pour CF) |
| wrangler | 4.0.0 | ⚠️ Wrangler 4.2.0 cassé avec D1 — épingler à 4.0.0 si problèmes |

**Sous-modules inclus dans @nuxtjs/seo :**
- `@nuxtjs/sitemap` 7.5+ - Sitemap XML avec hreflang i18n
- `@nuxtjs/robots` - robots.txt dynamique
- `nuxt-schema-org` v5.0+ - JSON-LD Schema.org
- `nuxt-og-image` 4.0+ - Génération images OG au build (zeroRuntime)
- `nuxt-link-checker` - Validation liens (dev uniquement)
- `nuxt-seo-utils` - useSiteConfig, useLocaleHead...

## Alternatives à Considérer

**Recherche** :
- Orama (8.2k+ stars) : Plus de fonctionnalités, API TypeScript-first, mais plus lourd (~15KB)
- Pagefind : Index post-build automatique, mais nécessite chunks externes

**Validation (Standard Schema compatible)** :

| Library | Taille (gzip / min) | Tree-shaking | Meilleur pour |
|---------|---------------------|--------------|---------------|
| **Valibot v1.0** | ~1KB / ~2KB | Excellent | Client-side, builds minimaux |
| Zod 4 (`zod/mini`) | ~2KB / ~5KB | Bon | Équilibre taille/écosystème |
| Zod 4 (complet) | ~5KB / ~10KB | Modéré | Schémas complexes, inférence TS |

**Bundle Analysis** :
- `npx nuxi analyze` : Intégré (vite-bundle-visualizer)
- nuxt-bundle-analysis : GitHub Action pour CI/CD

## Notes Importantes

1. **TailwindCSS 4 intégration** : `@nuxtjs/tailwindcss@6.14.0` supporte TW4, mais `@tailwindcss/vite` est recommandé pour les nouveaux projets TW4 (Config Viewer moins utile avec CSS-first).

2. **Zod 4 migration** :
   - Import API moderne : `import { z } from 'zod/v4'` (top-level validators, `z.iso.date()`, etc.)
   - Import mini : `import * as z from 'zod/mini'` (pas `@zod/mini` package)
   - Codemod automatique : `npx zod-v3-to-v4 path/to/tsconfig.json`
   - JSON Schema export natif : `z.toJSONSchema(schema)` remplace `zod-to-json-schema` (déprécié nov. 2025)
   - ⚠️ `z.date()` → `z.iso.date()` pour compatibilité JSON Schema
   - ⚠️ `.default()` s'applique maintenant DANS `.optional()` (breaking change comportemental)
   - **zod/mini syntaxe fonctionnelle** (tree-shakable) :
     ```typescript
     // Regular zod (chaining)
     z.string().min(5).max(100).trim()

     // zod/mini (functional, ~64% plus léger)
     z.string().check(z.minLength(5), z.maxLength(100), z.trim())
     ```
   - zod/mini nécessite `z.config(z.locales.en())` pour messages localisés (défaut: "Invalid input")

3. **nuxt-delay-hydration obsolète** : Hydratation lazy native depuis Nuxt 3.16+ (`hydrate-on-visible`, `hydrate-on-idle`, `hydrate-on-interaction`, `hydrate-on-media-query`, `hydrate-when`, `hydrate-never`).

4. **Reka UI est le nouveau standard** : Rebrand de Radix Vue (février 2025). shadcn-vue utilise Reka UI par défaut.

5. **Node.js 22 LTS recommandé** : Version stable pour Cloudflare Pages. Node 24 existe mais non recommandé pour CF (support souvent en retard).

6. **pnpm 10 sécurité** : Lifecycle scripts désactivés par défaut. Configuration via `package.json` (champ `pnpm`) ou `pnpm-workspace.yaml` (pas `.npmrc`).

7. **llms.txt** : Module `nuxt-llms` génère automatiquement `/llms.txt` avec @nuxt/content ^3.2.0. Remplace le server route personnalisé.

8. **@nuxt/content v3 API** : Changements majeurs par rapport à v2 :
   - ❌ Composants supprimés : `<ContentDoc>`, `<ContentList>`, `<ContentQuery>`
   - ✅ Utiliser `<ContentRenderer>` pour tout le rendu
   - ❌ `queryContent()` (v2) → ✅ `queryCollection()` (v3)
   - Mode document-driven supprimé - créer les pages manuellement
   - Composants prose personnalisés dans `components/prose/`

9. **Reading time** : 200 wpm standard, considérer 180 wpm pour contenu technique avec code.

10. **Wrangler 3.33.0+** : les commandes D1 utilisent **local par défaut**. Toujours spécifier `--local` ou `--remote` explicitement pour éviter les confusions.

11. **Wrangler 4.2.0 cassé** : Cette version a des bugs avec D1. Épingler à `"wrangler": "4.0.0"` dans package.json si problèmes.

12. **Tests Nuxt 4 avec @nuxt/test-utils** : Configuration complète dans `testing-patterns.md`. Installation :
    ```bash
    pnpm add -D @nuxt/test-utils vitest vitest-axe @vue/test-utils happy-dom @vitest/coverage-v8
    ```
    Helpers essentiels : `mountSuspended()` (composants async), `mockNuxtImport()` (auto-imports), `registerEndpoint()` (API mock).

13. **Tests a11y vitest-axe** : Pour les tests unitaires d'accessibilité des composants :
    ```typescript
    import { axe, toHaveNoViolations } from 'vitest-axe'
    expect.extend(toHaveNoViolations)

    it('composant accessible', async () => {
      const wrapper = await mountSuspended(MonComposant)
      expect(await axe(wrapper.element)).toHaveNoViolations()
    })
    ```
    Complète @axe-core/playwright (tests E2E) avec validation au niveau composant.

## Disponibilité des fonctionnalités en SSG

En mode SSG (`nuxt generate` ou `nuxt build --preset=cloudflare_pages`), certaines fonctionnalités ne sont disponibles qu'au build-time :

| Fonctionnalité | Build-time | Runtime | Notes |
|----------------|:----------:|:-------:|-------|
| Routes `server/api/` | ✅ | ❌ | Exécutées au build, résultats sauvegardés en payload |
| Pré-rendu via `useFetch` | ✅ | ❌ | Données intégrées dans le HTML généré |
| Server middleware | ✅ | ❌ | Exécuté au build uniquement |
| Génération sitemap/robots | ✅ | ✅ | Fichiers statiques générés |
| Requêtes Nuxt Content | ✅ | ✅ | Via SQLite WASM côté client (D1 non requis pour navigation) |
| Routes API dynamiques | ❌ | ❌ | **Non supporté en SSG** |
| Server-sent events | ❌ | ❌ | Requiert serveur runtime |

**Point critique** : Les appels `useFetch('/api/...')` dans les pages fonctionnent car ils s'exécutent au build-time. Les résultats sont sérialisés dans les payloads statiques. Mais ces routes API ne répondront **pas** à des requêtes runtime.

## Anti-patterns Déploiement Cloudflare

| Anti-pattern | Conséquence | Solution |
|--------------|-------------|----------|
| `ssr: false` | Pages vides, pas de prerendering | Garder `ssr: true` (défaut) |
| Cacher `node_modules` avec pnpm | Cache invalide, builds lents | Cacher le pnpm store à la place |
| Fallback SPA `/* /index.html 200` | Inutile pour SSG (chaque route a son HTML) | Ne pas ajouter |
| `NITRO_PRESET` en variable d'environnement | Détecté automatiquement | Ne pas définir |
| Oublier de désactiver Rocket Loader | Erreurs d'hydratation Vue | Désactiver dans CF Dashboard |
| Pas de dimensions sur images | CLS élevé | Toujours `width` + `height` |
| `wrangler.toml` sans binding D1 | Erreur 500 au runtime | Configurer `[[d1_databases]]` |

## Anti-patterns Structure Nuxt 4

| Anti-pattern | Conséquence | Solution |
|--------------|-------------|----------|
| Placer `content/` dans `app/` | Collections non détectées | Garder `content/` à la racine |
| Placer `server/` dans `app/` | Routes API non détectées | Garder `server/` à la racine |
| Placer `public/` dans `app/` | Assets statiques non servis | Garder `public/` à la racine |
| Modifier `tsconfig.json` directement | Écrasé par Nuxt au build | Utiliser `alias` dans `nuxt.config.ts` |
| Composables sans préfixe `use` | Auto-import échoue | Toujours préfixer avec `use` |
| Code Vue/Nitro dans `shared/` | Erreurs d'import | Uniquement TypeScript isomorphe pur |
| Sous-dossiers dans `composables/` | Pas auto-scannés | Re-export dans `index.ts` ou `imports.dirs` |
| `tailwind.config.ts` avec TW4 | Ignoré | Utiliser `@theme` dans le CSS |
| Ignorer validation frontmatter | Erreurs runtime silencieuses | Toujours définir schémas Zod |

**Note:** L'initialisation du projet avec ces commandes sera la première story d'implémentation.