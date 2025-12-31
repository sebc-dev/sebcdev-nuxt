# Story 1.4: Cloudflare Pages Deployment

Status: ready-for-dev

## Story

As a visiteur,
I want the site to load quickly from anywhere in the world,
So that I have a fast, reliable reading experience.

## Acceptance Criteria

1. **AC1 - Déploiement Cloudflare Pages** : Le site est déployé et accessible via `*.pages.dev`
2. **AC2 - TTFB** : Le TTFB est < 200ms depuis différentes régions
3. **AC3 - HTTPS** : Le site est servi via HTTPS avec certificat valide
4. **AC4 - Headers sécurité** : Les headers CSP, X-Frame-Options, HSTS sont configurés
5. **AC5 - GitHub Actions CI/CD** : Le workflow CI/CD déploie automatiquement sur push main
6. **AC6 - Preview deployments** : Les PRs génèrent des preview URLs automatiquement

## Tasks / Subtasks

- [ ] **Task 1: Configuration wrangler.toml complète** (AC: #1)
  - [ ] 1.1 Créer wrangler.toml avec configuration D1 (déjà fait Story 1.1)
  - [ ] 1.2 Ajouter la configuration Cloudflare Pages
  - [ ] 1.3 Configurer le compatibility_date

- [ ] **Task 2: Fichier public/_headers** (AC: #4)
  - [ ] 2.1 Créer `public/_headers` avec headers de sécurité OWASP
  - [ ] 2.2 Configurer CSP (Content-Security-Policy)
  - [ ] 2.3 Configurer cache headers pour assets hashés
  - [ ] 2.4 Configurer Early Hints pour fonts
  - [ ] 2.5 Configurer headers SEO (llms.txt, sitemap)

- [ ] **Task 3: Configuration Cloudflare Dashboard** (AC: #1, #2, #3)
  - [ ] 3.1 Créer le projet Cloudflare Pages
  - [ ] 3.2 Lier avec le repository GitHub
  - [ ] 3.3 Configurer le build (pnpm, Node 22)
  - [ ] 3.4 Désactiver les optimisations problématiques (Rocket Loader, Mirage)
  - [ ] 3.5 Activer HTTP/3 et Early Hints

- [ ] **Task 4: GitHub Actions workflow deploy.yml** (AC: #5, #6)
  - [ ] 4.1 Créer `.github/workflows/deploy.yml`
  - [ ] 4.2 Configurer jobs parallèles (lint, typecheck, build)
  - [ ] 4.3 Configurer le job deploy avec wrangler-action
  - [ ] 4.4 Configurer les commentaires PR avec preview URL

- [ ] **Task 5: Secrets et variables GitHub** (AC: #5)
  - [ ] 5.1 Créer le token API Cloudflare avec permissions Pages Edit
  - [ ] 5.2 Configurer `CLOUDFLARE_API_TOKEN` secret
  - [ ] 5.3 Configurer `CLOUDFLARE_ACCOUNT_ID` secret
  - [ ] 5.4 Configurer variables: `CF_PROJECT_NAME`, `SITE_URL`

- [ ] **Task 6: Workflow cleanup-previews.yml** (AC: #6)
  - [ ] 6.1 Créer `.github/workflows/cleanup-previews.yml`
  - [ ] 6.2 Configurer la suppression des previews après merge

- [ ] **Task 7: Configuration ESLint + Prettier** (AC: #5)
  - [ ] 7.1 Configurer ESLint flat config
  - [ ] 7.2 Configurer Prettier avec tailwindStylesheet
  - [ ] 7.3 Ajouter scripts lint et typecheck dans package.json

- [ ] **Task 8: Validation déploiement** (AC: #1-6)
  - [ ] 8.1 Push sur main et vérifier le déploiement
  - [ ] 8.2 Tester l'URL de production
  - [ ] 8.3 Vérifier les headers de sécurité avec securityheaders.com
  - [ ] 8.4 Vérifier le TTFB avec WebPageTest

## Dev Notes

### Prérequis des Stories précédentes

Cette story s'appuie sur:
- Projet Nuxt 4 fonctionnel (Story 1.1)
- wrangler.toml avec D1 configuré (Story 1.1)
- TailwindCSS 4 et shadcn-vue configurés (Story 1.2)
- Layouts responsive fonctionnels (Story 1.3)

### Configuration wrangler.toml

```toml
# wrangler.toml
name = "sebc-dev"
compatibility_date = "2024-09-19"

# D1 Database (OBLIGATOIRE pour Nuxt Content 3)
[[d1_databases]]
binding = "DB"
database_name = "content-db"
database_id = "VOTRE_DATABASE_ID"  # Généré par: wrangler d1 create content-db
```

### Fichier public/_headers

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

/*.svg
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

/llms.txt
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

### GitHub Actions deploy.yml

```yaml
# .github/workflows/deploy.yml
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
        with:
          version: ${{ env.PNPM_VERSION }}

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm lint

  typecheck:
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
      - run: pnpm typecheck

  build:
    needs: [lint, typecheck]
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

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

      - uses: actions/upload-artifact@v4
        with:
          name: build-output
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
            ## Preview Deployment

            | Info | Valeur |
            |------|--------|
            | **Commit** | `${{ github.event.pull_request.head.sha }}` |
            | **Preview URL** | ${{ needs.deploy.outputs.url }} |

            _Mis à jour automatiquement à chaque push_
          reactions: rocket
```

### GitHub Actions cleanup-previews.yml

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

### Scripts package.json

```json
{
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build",
    "generate": "nuxt generate",
    "preview": "nuxt preview",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "typecheck": "nuxt typecheck",
    "prepare": "husky"
  }
}
```

### Configuration ESLint (eslint.config.mjs)

```javascript
// eslint.config.mjs
import { createConfigForNuxt } from '@nuxt/eslint-config/flat'

export default createConfigForNuxt({
  features: {
    stylistic: true,
    tooling: true,
  },
})
  .append({
    rules: {
      '@stylistic/quotes': ['error', 'single'],
      '@stylistic/semi': ['error', 'false'],
    },
  })
```

### Configuration Prettier (.prettierrc)

```json
{
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindStylesheet": "./app/assets/css/main.css",
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2
}
```

### Paramètres Cloudflare Dashboard

**À désactiver (Speed → Optimization):**
- ❌ Rocket Loader
- ❌ Mirage
- ❌ Email Address Obfuscation
- ❌ Auto-minification

**À activer:**
- ✅ HTTP/3 (Protocol Optimization)
- ✅ Early Hints (Protocol Optimization)
- ✅ Build caching (beta)

**Configuration Build (Settings → Builds):**
| Paramètre | Valeur |
|-----------|--------|
| Framework preset | Nuxt.js |
| Build command | `pnpm run build` |
| Build output directory | `.output/public` |
| Node version | `22` |

**Environment Variables:**
| Variable | Valeur | Scope |
|----------|--------|-------|
| `HUSKY` | `0` | Production & Preview |

### Création Token API Cloudflare

1. Naviguer vers https://dash.cloudflare.com/profile/api-tokens
2. **Create Token** → **Custom Token**
3. Permissions: Account → **Cloudflare Pages** → **Edit**
4. Restreindre au compte spécifique

### Secrets et Variables GitHub

**Secrets (Settings → Secrets → Actions):**
- `CLOUDFLARE_API_TOKEN` : Token créé ci-dessus
- `CLOUDFLARE_ACCOUNT_ID` : ID du compte Cloudflare

**Variables (Settings → Variables → Actions):**
- `CF_PROJECT_NAME` : `sebc-dev`
- `SITE_URL` : `https://sebc.dev`

### Headers de sécurité expliqués

| Header | Fonction |
|--------|----------|
| `X-Content-Type-Options: nosniff` | Empêche MIME-sniffing |
| `X-Frame-Options: DENY` | Protection clickjacking |
| `Strict-Transport-Security` | Force HTTPS (2 ans) |
| `Content-Security-Policy` | Protection XSS majeure |
| `Cross-Origin-Opener-Policy` | Protection cross-origin |

**Limitation CSP:** shadcn-vue/Reka UI nécessite `style-src 'unsafe-inline'` pour le positionnement des composants.

### Outils de validation

| Outil | URL | Usage |
|-------|-----|-------|
| Security Headers | https://securityheaders.com | Vérifier headers sécurité |
| WebPageTest | https://webpagetest.org | Tester TTFB multi-régions |
| Lighthouse | Chrome DevTools | Score performance global |
| SSL Labs | https://www.ssllabs.com/ssltest/ | Vérifier certificat HTTPS |

### Anti-patterns à éviter

- ❌ Ne PAS activer Rocket Loader ou Mirage (problèmes hydratation)
- ❌ Ne PAS oublier de désactiver Husky en CI (`HUSKY=0`)
- ❌ Ne PAS utiliser `--no-verify` pour bypasser les hooks
- ❌ Ne PAS combiner cache pnpm/action-setup ET setup-node cache
- ❌ Ne PAS oublier `--frozen-lockfile` en CI

### Apprentissages des Stories précédentes

- Structure `app/`, `content/`, `server/` confirmée
- TailwindCSS 4 via @tailwindcss/vite fonctionne
- D1 configuré dans wrangler.toml

### References

- [Source: architecture/core-architectural-decisions/infrastructure-deployment.md] - Stack tests a11y, Cloudflare avantages
- [Source: architecture/starter-template-evaluation/parametres-cloudflare-pages.md] - Headers _headers complet
- [Source: architecture/starter-template-evaluation/cicd-github-actions-recommande.md] - Workflows GitHub Actions
- [Source: epics/epic-1-core-reading-experience.md#Story-1.4] - Requirements NFR4, NFR20-23

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

### File List
