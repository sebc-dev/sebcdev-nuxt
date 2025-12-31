# Nuxt 4 SSG sur Cloudflare Pages : guide complet de configuration

Le déploiement de Nuxt 4.2.x en mode SSG sur Cloudflare Pages ne nécessite **pas de preset Nitro spécifique** — la commande `nuxt generate` produit des fichiers HTML statiques purs qui fonctionnent sans configuration serveur. Les deux points critiques à maîtriser sont la désactivation des optimisations Cloudflare incompatibles avec Vue (Rocket Loader, Email Obfuscation) et la configuration correcte des headers de cache pour les assets hashés.

## Configuration nuxt.config.ts pour SSG pur

Pour un déploiement statique complet, la configuration Nitro reste minimale. Le paramètre `ssr: true` (par défaut) est **essentiel** car il active le prerendering — contrairement à l'intuition, `ssr: false` génèrerait des coquilles HTML vides sans contenu pré-rendu.

```typescript
// nuxt.config.ts
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  compatibilityDate: '2025-01-01',
  
  // ssr: true est le défaut - OBLIGATOIRE pour le prerendering
  ssr: true,
  
  modules: ['@nuxt/content'],
  
  css: ['~/assets/css/main.css'],
  
  vite: {
    plugins: [tailwindcss()]  // TailwindCSS 4.1.x via Vite plugin
  },
  
  nitro: {
    // Pas de preset nécessaire pour SSG pur
    prerender: {
      crawlLinks: true,    // Découverte automatique des routes
      routes: ['/'],       // Point d'entrée du crawl
    },
  },
  
  routeRules: {
    '/**': { prerender: true },  // Force le prerendering global
  },
  
  runtimeConfig: {
    public: {
      siteUrl: '',      // NUXT_PUBLIC_SITE_URL
      apiBase: '',      // NUXT_PUBLIC_API_BASE
    }
  }
})
```

La nouvelle structure Nuxt 4 place le code applicatif dans `app/` tandis que `server/`, `public/`, et `nuxt.config.ts` restent à la racine. L'alias `~` pointe désormais vers `app/`.

## Le fichier wrangler.toml n'est pas requis

Pour un déploiement SSG pur, **wrangler.toml est optionnel**. Cloudflare Pages sert directement les fichiers statiques de `.output/public/` sans configuration additionnelle. Si vous avez besoin de bindings ou variables d'environnement côté Cloudflare :

```toml
# wrangler.toml (optionnel)
name = "mon-site-nuxt"
compatibility_date = "2025-01-01"
pages_build_output_dir = ".output/public"

# Node.js compatibility (inutile pour SSG pur, mais safe)
compatibility_flags = ["nodejs_compat"]

[vars]
API_URL = "https://api.example.com"
```

## Commandes de build et variables d'environnement

Les commandes `nuxt generate` et `nuxt build --prerender` produisent un résultat identique pour le SSG — les deux génèrent des fichiers statiques dans `.output/public/`. La différence avec `nuxt build` simple est que ce dernier inclut un serveur Nitro pour le SSR.

### Variables d'environnement requises

| Variable | Valeur | Obligatoire |
|----------|--------|-------------|
| `NODE_VERSION` | `22` | ✅ Oui |
| `PNPM_VERSION` | `10` | ✅ Oui (si pnpm utilisé) |
| `NUXT_PUBLIC_*` | Vos valeurs | Build-time uniquement |

Créez un fichier `.nvmrc` à la racine avec `22` comme contenu — cette méthode est plus fiable que les variables d'environnement UI quand wrangler.toml est présent.

### Configuration Cloudflare Pages Dashboard

- **Build command** : `pnpm nuxt generate`
- **Output directory** : `.output/public`
- **Root directory** : `/` (ou chemin du monorepo)

## Paramètres Cloudflare à désactiver obligatoirement

Trois fonctionnalités Cloudflare cassent l'hydratation Vue/Nuxt et doivent être désactivées **avant** le premier déploiement :

**Rocket Loader** (`Speed > Settings > Content Optimization`) injecte un script qui diffère le chargement JavaScript jusqu'après `window.onload`. Cela détruit l'hydratation Vue car les commentaires HTML `<!---->` utilisés comme placeholders ne correspondent plus au DOM attendu, générant des erreurs `HierarchyRequestError`.

**Email Address Obfuscation** (`Security > Settings > Scrape Shield`) injecte `email-decode.min.js` qui interfère avec les transitions SPA et peut corrompre les templates Vue en plaçant des scripts à l'intérieur des balises `<template>`.

**Server-side Excludes** (`Security > Settings > Scrape Shield`) peut injecter du contenu non souhaité dans le HTML.

Auto Minify et Mirage ont été **dépréciés** respectivement en août 2024 et septembre 2025 — aucune action requise. Brotli compression reste safe et améliore les performances.

## Configuration des headers et redirections

Le fichier `_headers` doit être placé dans `public/` pour être copié dans `.output/public/` au build.

```plaintext
# public/_headers

# Security headers globaux
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()

# HTML - toujours revalider
/*.html
  Cache-Control: public, max-age=0, must-revalidate

/
  Cache-Control: public, max-age=0, must-revalidate

# Assets Nuxt hashés - cache immutable 1 an
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

# Fonts - cache long
/fonts/*
  Cache-Control: public, max-age=31536000, immutable

# Bloquer l'indexation des preview deployments
https://:project.pages.dev/*
  X-Robots-Tag: noindex
```

Les fichiers `/_nuxt/*.js` et `/_nuxt/*.css` contiennent des hashes de contenu (`entry.abc123.js`) — le cache `immutable` est safe car le nom change automatiquement quand le contenu change.

### Redirections

```plaintext
# public/_redirects

# Normalisation trailing slash
/about/ /about 301

# Redirections legacy
/old-blog/* /blog/:splat 301

# Liens externes
/twitter https://twitter.com/handle 302
```

**Important** : Le fallback SPA (`/* /index.html 200`) n'est **pas nécessaire** pour SSG pur — Nuxt génère un fichier HTML individuel pour chaque route.

## Workflow GitHub Actions recommandé

L'utilisation de GitHub Actions au lieu des builds natifs Cloudflare offre un contrôle total sur le caching et les étapes de build :

```yaml
# .github/workflows/deploy.yml
name: Deploy to Cloudflare Pages

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '22'
  PNPM_VERSION: '10'

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      deployments: write
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Cache .nuxt
        uses: actions/cache@v4
        with:
          path: |
            .nuxt
            .output
          key: nuxt-${{ hashFiles('**/pnpm-lock.yaml') }}-${{ github.sha }}
          restore-keys: nuxt-${{ hashFiles('**/pnpm-lock.yaml') }}-
      
      - run: pnpm install --frozen-lockfile
      
      - run: pnpm nuxt generate
        env:
          NUXT_PUBLIC_SITE_URL: ${{ vars.SITE_URL }}
      
      - uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy .output/public --project-name=mon-site
```

Les **Direct Uploads** via Wrangler CLI ne comptent pas dans le quota de 500 builds/mois du free tier.

## Limites du free tier Cloudflare Pages

| Limite | Valeur free tier |
|--------|------------------|
| Builds par mois | **500** (compte entier) |
| Builds concurrents | 1 |
| Timeout build | 20 minutes |
| Fichiers par déploiement | **20,000** |
| Taille max fichier | **25 MiB** |
| Projets par compte | 100 |
| Bandwidth | **Illimité** |
| Preview deployments | Illimité |

## Anti-patterns à éviter

- **Ne pas mettre `ssr: false`** en pensant que "SSG = pas de SSR" — cela désactive le prerendering et génère des pages vides
- **Ne pas utiliser le preset `cloudflare_pages`** pour du SSG pur — il ajoute inutilement des server functions
- **Ne pas cacher `node_modules`** directement avec pnpm — cache le store pnpm à la place
- **Ne pas oublier de désactiver Rocket Loader** — c'est la cause #1 des bugs d'hydratation sur Cloudflare
- **Ne pas ajouter de fallback SPA** (`/* /index.html 200`) pour du SSG — chaque route a son propre HTML
- **Ne pas utiliser `NITRO_PRESET`** en variable d'environnement pour SSG — c'est détecté automatiquement

## Checklist actionnable pour Claude Code

```markdown
## Checklist déploiement Nuxt 4 SSG → Cloudflare Pages

### Configuration projet
- [ ] nuxt.config.ts : `ssr: true` (défaut) conservé
- [ ] nuxt.config.ts : `nitro.prerender.crawlLinks: true`
- [ ] nuxt.config.ts : Pas de preset Nitro spécifié
- [ ] .nvmrc créé avec contenu `22`
- [ ] TailwindCSS 4.1.x via `@tailwindcss/vite` plugin

### Fichiers statiques dans public/
- [ ] public/_headers créé avec cache immutable pour /_nuxt/*
- [ ] public/_headers : HTML avec max-age=0, must-revalidate
- [ ] public/_headers : X-Robots-Tag noindex pour *.pages.dev
- [ ] public/_redirects créé (si redirections nécessaires)

### Cloudflare Dashboard
- [ ] Build command : `pnpm nuxt generate`
- [ ] Output directory : `.output/public`
- [ ] NODE_VERSION=22 configuré
- [ ] PNPM_VERSION=10 configuré
- [ ] Rocket Loader : DÉSACTIVÉ
- [ ] Email Obfuscation : DÉSACTIVÉ
- [ ] Server-side Excludes : DÉSACTIVÉ

### CI/CD (si GitHub Actions)
- [ ] CLOUDFLARE_API_TOKEN dans secrets
- [ ] CLOUDFLARE_ACCOUNT_ID dans secrets
- [ ] Cache pnpm store configuré
- [ ] Cache .nuxt configuré
- [ ] wrangler-action@v3 pour deploy

### Vérification post-déploiement
- [ ] Console navigateur sans erreurs hydratation
- [ ] Toutes les routes SSG accessibles
- [ ] Headers Cache-Control corrects (DevTools Network)
- [ ] Preview deployments non indexés (check X-Robots-Tag)
```

## Structure de fichiers finale

```
project-root/
├── app/                      # Code source Nuxt 4
│   ├── assets/css/main.css
│   ├── components/
│   ├── pages/
│   └── app.vue
├── content/                  # Nuxt Content markdown
├── public/
│   ├── _headers             # Headers Cloudflare
│   ├── _redirects           # Redirections Cloudflare
│   └── favicon.ico
├── server/                   # API routes (vide pour SSG pur)
├── .nvmrc                    # Contenu: 22
├── .github/workflows/deploy.yml
├── nuxt.config.ts
├── package.json
└── pnpm-lock.yaml
```

## Conclusion

Le déploiement Nuxt 4 SSG sur Cloudflare Pages est remarquablement simple une fois les pièges connus évités. La clé est de **ne pas sur-configurer** : pas de preset Nitro, pas de wrangler.toml obligatoire, pas de fallback SPA. Les trois actions critiques sont : garder `ssr: true`, désactiver Rocket Loader et Email Obfuscation, et configurer des headers de cache appropriés pour les assets hashés. Le free tier Cloudflare avec sa bande passante illimitée et ses 500 builds mensuels couvre largement les besoins d'un site statique en production.