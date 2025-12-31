# CI/CD GitHub Actions pour Nuxt 4 SSG sur Cloudflare Pages

**Découverte majeure** : L'action `cloudflare/pages-action@v1` est **dépréciée depuis octobre 2024**. La migration vers `cloudflare/wrangler-action@v3` est impérative. Ce guide présente les meilleures pratiques actualisées pour décembre 2025, couvrant le cache pnpm optimisé, le build cache Nuxt 4, le déploiement Cloudflare, et les preview environments automatisés.

Le stack technique ciblé (Nuxt 4.2.x, pnpm 10.26+, Node.js 22 LTS, Nuxt Content 3.10+) requiert une configuration spécifique tenant compte de la nouvelle structure `app/` directory et des outputs SSG dans `.output/public`.

---

## Configuration pnpm cache optimale pour 2025

La gestion du cache pnpm dans GitHub Actions a évolué significativement. Trois approches principales existent, avec des compromis différents entre simplicité et contrôle.

### Approche recommandée : pnpm/action-setup avec cache intégré

Depuis la version 4, `pnpm/action-setup` intègre une option `cache: true` qui simplifie considérablement la configuration :

```yaml
- name: Install pnpm
  uses: pnpm/action-setup@v4
  with:
    version: 10
    cache: true  # Active le cache automatiquement

- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: 22

- name: Install dependencies
  run: pnpm install --frozen-lockfile
```

Cette approche gère automatiquement le **pnpm store prune** en post-action et génère une clé de cache basée sur `pnpm-lock.yaml`. Elle convient à **90% des projets** sans configuration supplémentaire.

### Configuration manuelle avec actions/cache pour contrôle avancé

Pour les projets nécessitant une rotation de cache temporelle ou des clés personnalisées :

```yaml
- name: Install pnpm
  uses: pnpm/action-setup@v4
  with:
    version: 10
    run_install: false

- name: Get pnpm store directory
  shell: bash
  run: |
    echo "STORE_PATH=$(pnpm store path --silent)" >> $GITHUB_ENV

- name: Get cache rotation key
  id: cache-rotation
  shell: bash
  run: |
    echo "YEAR_MONTH=$(/bin/date -u '+%Y%m')" >> $GITHUB_OUTPUT

- name: Setup pnpm cache
  uses: actions/cache@v4
  with:
    path: ${{ env.STORE_PATH }}
    key: ${{ runner.os }}-pnpm-v10-${{ steps.cache-rotation.outputs.YEAR_MONTH }}-${{ hashFiles('**/pnpm-lock.yaml') }}
    restore-keys: |
      ${{ runner.os }}-pnpm-v10-${{ steps.cache-rotation.outputs.YEAR_MONTH }}-
      ${{ runner.os }}-pnpm-v10-
```

La rotation mensuelle via `YEAR_MONTH` évite la fragmentation du cache sur le long terme. Le cache GitHub est limité à **10 GB par repository**.

### Règles critiques à respecter

- **Ne jamais combiner** `pnpm/action-setup` cache ET `setup-node` cache simultanément
- **Ne pas cacher `node_modules`** directement — GitHub le déconseille formellement
- Toujours utiliser `--frozen-lockfile` en CI pour la reproductibilité
- Installer pnpm **avant** setup-node quand vous utilisez `cache: 'pnpm'`

---

## Build cache Nuxt 4 : répertoires et stratégies

Nuxt 4 introduit une nouvelle structure de répertoires avec le dossier `app/` qui modifie les stratégies de cache par rapport à Nuxt 3.

### Répertoires à cacher

| Répertoire | Usage | Priorité |
|------------|-------|----------|
| `node_modules/.cache/nuxt` | Artefacts build Nuxt (via nuxt-build-cache) | **Élevée** |
| `node_modules/.vite` | Cache Vite | **Moyenne** |
| `.nuxt/` | Artefacts de développement | Faible (régénéré) |
| `.output/` | Sortie production | **Ne pas cacher** |

Le module expérimental `nuxt-build-cache` de @pi0 collecte les artefacts de `.nuxt/` dans un fichier tar stocké dans `node_modules/.cache/nuxt/build/{hash}/`. Le hash est généré via `ohash` à partir du code source et des configurations.

### Configuration cache GitHub Actions pour Nuxt 4

```yaml
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
```

La clé multi-niveaux invalide le cache si :
1. Le lockfile change (nouvelles dépendances)
2. La configuration Nuxt change
3. Le code source ou contenu change

### Invalidation forcée du cache

Pour forcer un rebuild complet, définissez les variables d'environnement :

```yaml
env:
  NUXT_IGNORE_BUILD_CACHE: true  # Ignore le cache existant
  # ou
  NUXT_DISABLE_BUILD_CACHE: true  # Désactive complètement le module
```

---

## Configuration nuxt.config.ts optimale pour SSG Cloudflare

```typescript
// nuxt.config.ts - Nuxt 4.2.x + Nuxt Content 3.10+ SSG
export default defineNuxtConfig({
  future: {
    compatibilityVersion: 4,
  },

  ssr: true,

  modules: [
    '@nuxt/content',
  ],

  nitro: {
    // Pour SSG pur (pas de server-side rendering dynamique)
    preset: 'static',
    
    prerender: {
      crawlLinks: true,
      routes: ['/', '/sitemap.xml', '/robots.txt'],
      ignore: ['/api/**'],
      retry: 3,
      retryDelay: 500,
      concurrency: 10,
    },

    autoSubfolderIndex: false,
  },

  typescript: {
    strict: true,
    typeCheck: true,
  },

  experimental: {
    inlineRouteRules: true,
  },
})
```

**Point important** : Pour Nuxt Content 3.10+ en mode SSG pur, la base D1 n'est **pas requise**. Le contenu est intégré au build et SQLite WASM est utilisé côté client pour les requêtes de navigation.

### Structure Nuxt 4 vs Nuxt 3

| Aspect | Nuxt 3 | Nuxt 4 |
|--------|--------|--------|
| Code applicatif | Racine plate (`pages/`, `components/`) | Répertoire `app/` |
| Alias `~` | Pointe vers racine | Pointe vers `app/` |
| Sortie SSG | `.output/public` ou `dist` | `.output/public` |
| TypeScript | Config unique | Configs séparées (app, server, node) |

---

## Déploiement avec cloudflare/wrangler-action@v3

### Migration obligatoire depuis pages-action

L'action `cloudflare/pages-action` est **archivée et dépréciée**. La v1.5.0 est la dernière version. Utilisez exclusivement `cloudflare/wrangler-action@v3`.

### Configuration complète du déploiement

```yaml
- name: Deploy to Cloudflare Pages
  id: deploy
  uses: cloudflare/wrangler-action@v3
  with:
    apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
    command: >-
      pages deploy .output/public 
      --project-name=mon-blog-nuxt 
      --branch=${{ github.head_ref || github.ref_name }}
    gitHubToken: ${{ secrets.GITHUB_TOKEN }}
```

### Outputs disponibles (Wrangler ≥3.78.0)

| Output | Description |
|--------|-------------|
| `deployment-url` | URL unique du déploiement |
| `pages-deployment-alias-url` | URL alias de branche (preview) |
| `pages-deployment-id` | ID du déploiement (≥3.81.0) |
| `pages-environment` | `production` ou `preview` |

### Création du token API Cloudflare

1. Naviguer vers https://dash.cloudflare.com/profile/api-tokens
2. **Create Token** → **Custom Token**
3. Permissions : Account → **Cloudflare Pages** → **Edit**
4. Restreindre au compte spécifique

L'Account ID se trouve dans l'URL du dashboard : `https://dash.cloudflare.com/<ACCOUNT_ID>/pages`

---

## Preview environments automatisés

Cloudflare Pages détecte automatiquement l'environnement via le nom de branche : `main` → production, autres → preview.

### Commentaires automatiques sur PR

```yaml
- name: Find existing comment
  uses: peter-evans/find-comment@v3
  id: fc
  with:
    issue-number: ${{ github.event.pull_request.number }}
    comment-author: 'github-actions[bot]'
    body-includes: 'Preview Deployment'

- name: Create or update comment
  uses: peter-evans/create-or-update-comment@v5
  if: github.event_name == 'pull_request'
  with:
    comment-id: ${{ steps.fc.outputs.comment-id }}
    issue-number: ${{ github.event.pull_request.number }}
    edit-mode: replace
    body: |
      ## 🚀 Preview Deployment

      | Info | Value |
      |------|-------|
      | **Commit** | `${{ github.event.pull_request.head.sha }}` |
      | **Preview URL** | ${{ steps.deploy.outputs.pages-deployment-alias-url }} |
      
      _Mis à jour automatiquement_
    reactions: rocket
```

### Nettoyage des previews après merge

**Cloudflare ne supprime pas automatiquement** les déploiements preview. Implémentez un workflow de cleanup :

```yaml
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
            "https://api.cloudflare.com/client/v4/accounts/${{ secrets.CLOUDFLARE_ACCOUNT_ID }}/pages/projects/${{ vars.PROJECT_NAME }}/deployments" \
            -H "Authorization: Bearer ${{ secrets.CLOUDFLARE_API_TOKEN }}")
          
          BRANCH="${{ github.head_ref }}"
          DEPLOYMENT_ID=$(echo $DEPLOYMENTS | jq -r ".result[] | select(.deployment_trigger.metadata.branch == \"$BRANCH\") | .id" | head -1)
          
          if [ -n "$DEPLOYMENT_ID" ]; then
            curl -X DELETE \
              "https://api.cloudflare.com/client/v4/accounts/${{ secrets.CLOUDFLARE_ACCOUNT_ID }}/pages/projects/${{ vars.PROJECT_NAME }}/deployments/${DEPLOYMENT_ID}?force=true" \
              -H "Authorization: Bearer ${{ secrets.CLOUDFLARE_API_TOKEN }}"
          fi
```

---

## Optimisations avancées du workflow

### Jobs parallèles avec artifacts

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

jobs:
  lint:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      # ... lint steps

  typecheck:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      # ... typecheck steps

  build:
    needs: [lint, typecheck]
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 10, cache: true }
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - run: pnpm install --frozen-lockfile
      - run: pnpm generate
      
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: .output/public
          retention-days: 1

  deploy:
    needs: [build]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-output
          path: .output/public
      # ... deploy steps
```

### Retry strategy pour déploiements instables

```yaml
- name: Deploy with retry
  uses: nick-fields/retry@v3
  with:
    timeout_minutes: 10
    max_attempts: 3
    retry_wait_seconds: 30
    command: |
      npx wrangler pages deploy .output/public \
        --project-name=mon-blog \
        --branch=${{ github.head_ref || github.ref_name }}
  env:
    CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

### Conditional deployments

```yaml
deploy-preview:
  if: github.event_name == 'pull_request'
  # ...

deploy-production:
  if: github.ref == 'refs/heads/main' && github.event_name == 'push'
  environment:
    name: production
    url: https://mon-blog.com
  # ...
```

---

## Workflow complet de référence

```yaml
name: CI/CD Nuxt 4 Blog

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, synchronize, reopened]
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

env:
  NODE_VERSION: '22'
  PNPM_VERSION: '10'

jobs:
  lint:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: '${{ env.PNPM_VERSION }}', cache: true }
      - uses: actions/setup-node@v4
        with: { node-version: '${{ env.NODE_VERSION }}' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint

  typecheck:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: '${{ env.PNPM_VERSION }}', cache: true }
      - uses: actions/setup-node@v4
        with: { node-version: '${{ env.NODE_VERSION }}' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck

  build:
    needs: [lint, typecheck]
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: '${{ env.PNPM_VERSION }}', cache: true }
      - uses: actions/setup-node@v4
        with: { node-version: '${{ env.NODE_VERSION }}' }
      
      - name: Cache Nuxt build
        uses: actions/cache@v4
        with:
          path: |
            node_modules/.cache/nuxt
            node_modules/.vite
          key: ${{ runner.os }}-nuxt-${{ hashFiles('nuxt.config.ts', 'pnpm-lock.yaml') }}
          restore-keys: ${{ runner.os }}-nuxt-
      
      - run: pnpm install --frozen-lockfile
      - run: pnpm generate
      
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: .output/public
          retention-days: 1

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
          name: dist
          path: .output/public
      
      - name: Deploy to Cloudflare Pages
        id: deploy
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy .output/public --project-name=${{ vars.CF_PROJECT_NAME }} --branch=${{ github.head_ref || github.ref_name }}
          gitHubToken: ${{ secrets.GITHUB_TOKEN }}

  comment-pr:
    needs: [deploy]
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - uses: peter-evans/find-comment@v3
        id: fc
        with:
          issue-number: ${{ github.event.pull_request.number }}
          comment-author: 'github-actions[bot]'
          body-includes: 'Preview Deployment'

      - uses: peter-evans/create-or-update-comment@v5
        with:
          comment-id: ${{ steps.fc.outputs.comment-id }}
          issue-number: ${{ github.event.pull_request.number }}
          edit-mode: replace
          body: |
            ## 🚀 Preview Deployment
            
            | Commit | URL |
            |--------|-----|
            | `${{ github.sha }}` | ${{ needs.deploy.outputs.url }} |
```

---

## Conclusion

La configuration CI/CD optimale pour un blog Nuxt 4 SSG sur Cloudflare Pages en 2025 repose sur plusieurs piliers : l'utilisation de `pnpm/action-setup@v4` avec cache intégré pour la simplicité, `cloudflare/wrangler-action@v3` pour le déploiement (impératif depuis la dépréciation de pages-action), et une stratégie de cache multi-niveaux pour les artefacts Nuxt.

Les gains de performance significatifs proviennent de la parallélisation des jobs (lint, typecheck, build), du cache intelligent basé sur les hash de fichiers, et de l'utilisation d'artifacts pour séparer build et deploy. La gestion des preview environments avec commentaires automatiques sur PR améliore considérablement l'expérience développeur, tandis que le workflow de cleanup évite l'accumulation de déploiements obsolètes.