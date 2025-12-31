# CI/CD GitHub Actions (Recommandé)

L'intégration Git directe de Cloudflare Pages est simple, mais un workflow GitHub Actions offre plus de contrôle :
- **Direct Uploads ne comptent pas dans le quota 500 builds/mois**
- Jobs parallèles (lint, typecheck, build) pour réduire le temps total
- Cache pnpm et Nuxt optimisé
- Preview URLs automatiques dans les PRs
- Cleanup des previews après merge

## Workflow principal optimisé

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

## Workflow cleanup des preview deployments

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

## Retry strategy pour déploiements instables

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

## Configuration requise

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

## Cache pnpm : approches disponibles

| Approche | Configuration | Cas d'usage |
|----------|---------------|-------------|
| **`cache: true` (Recommandé)** | `pnpm/action-setup@v4` avec `cache: true` | 90% des projets, simplifié |
| Cache manuel | `actions/cache@v4` avec pnpm store path | Contrôle avancé, rotation temporelle |
| setup-node cache | `actions/setup-node` avec `cache: 'pnpm'` | **⚠️ Ne pas combiner avec pnpm/action-setup cache** |

**Règles critiques :**
- Ne jamais combiner `pnpm/action-setup` cache ET `setup-node` cache simultanément
- Ne pas cacher `node_modules` directement (déconseillé par GitHub)
- Toujours utiliser `--frozen-lockfile` en CI

## Outputs wrangler-action disponibles (≥3.78.0)

| Output | Description |
|--------|-------------|
| `deployment-url` | URL unique du déploiement |
| `pages-deployment-alias-url` | URL alias de branche (preview) |
| `pages-deployment-id` | ID du déploiement (≥3.81.0) |
| `pages-environment` | `production` ou `preview` |

## Renovate Bot (Gestion automatique des dépendances)

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

## Workflow Security Audit (GitHub Actions)

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

## Prettier avec TailwindCSS 4

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

## Release Please (CHANGELOG automatisé)

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

## Conventional Commits & Git Hooks

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

## Stratégie Git Branching

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

## Optimisation Quota Builds (500/mois)

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

## Versioning avec bumpp

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

## Format CHANGELOG (Keep a Changelog)

Si vous maintenez un CHANGELOG manuel ou utilisez commit-and-tag-version :

```markdown
# Changelog

Tous les changements notables de ce projet sont documentés dans ce fichier.

Le format est basé sur [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/),
et ce projet adhère au [Semantic Versioning](https://semver.org/lang/fr/).

# [Unreleased]

## Added
- Support du mode sombre

# [0.2.0] - 2025-12-30

## Added
- Composant de navigation responsive ([#15](https://github.com/user/repo/issues/15))
- Intégration recherche MiniSearch

## Fixed
- Erreur d'hydratation sur les pages SSG ([#23](https://github.com/user/repo/issues/23))

## Changed
- Migration vers Vue 3.5 pour les performances améliorées

# [0.1.0] - 2025-12-01

## Added
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

## Workflow GitHub Actions — Release sur Tag

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

## Review Code IA — Vigilance Sécurité

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
