# Versions des Packages (Décembre 2025)

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
