# Commande d'Initialisation

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
