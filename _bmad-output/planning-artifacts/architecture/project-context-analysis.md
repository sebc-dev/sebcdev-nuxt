# Project Context Analysis

## Requirements Overview

**Functional Requirements:**
50 FRs couvrant un blog technique bilingue avec système de navigation avancé, filtrage multi-critères, expérience de lecture enrichie (ToC, progression, code blocks), et optimisation GEO pour visibilité dans les moteurs IA.

**Non-Functional Requirements:**
- Performance: Lighthouse **85-98 réaliste** sur mobile (100/100/100/100 atteignable mais scores varient selon environnement), LCP < 2.5s, CLS < 0.1, INP < 200ms
- Accessibilité: WCAG 2.1 AA, navigation clavier 100%
- SEO: Score 100, Schema.org validé, crawl < 500ms
- Sécurité: HTTPS obligatoire, headers CSP, Mozilla Observatory B+
- Disponibilité: 99.5% uptime mensuel
- **Priorité**: Core Web Vitals réels (PageSpeed Insights) > scores Lighthouse synthétiques

**Seuils Core Web Vitals 2025:**

| Métrique | Bon | Acceptable | Mauvais |
|----------|-----|------------|---------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5-4.0s | > 4.0s |
| **INP** (Interaction to Next Paint) | ≤ 200ms | 200-500ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1-0.25 | > 0.25 |

**Scale & Complexity:**

- Primary domain: Full-stack Content Platform (Nuxt 4 Static)
- Complexity level: Medium-High
- Estimated architectural components: ~8-12 modules principaux

## Technical Stack Decisions

| Technologie | Version | Rôle |
|-------------|---------|------|
| **Nuxt** | 4.2.2 | Framework full-stack, SSG mode |
| **Nuxt Content** | 3.10.0+ | CMS fichiers Markdown/MDC |
| **Search Engine** | MiniSearch 7.x | Recherche full-text client-side (~7KB minified, index JSON pré-généré) |
| **TailwindCSS** | 4.1.17 | Styling utility-first via @tailwindcss/vite |
| **@tailwindcss/vite** | 4.1.18 | Intégration Vite pour TailwindCSS 4 |
| **shadcn-vue** | 2.4.3+ | Composants UI accessibles (Reka UI 2.7.0) |
| **Reka UI** | 2.7.0 | Primitives accessibles (rebrand radix-vue février 2025) |
| **Validation** | Zod 4 / zod/mini | Validation schéma (~5KB gzip / ~10KB min, `zod/mini` ~2KB gzip réduction 64%) ou Valibot (~1KB gzip / ~2KB min) + Standard Schema |
| **nuxt-schema-org** | v5.0.10 | Schema.org standalone (ou via @nuxtjs/seo) |
| **@nuxtjs/sitemap** | v7.5.0+ | Génération sitemap, `asSitemapCollection()` pour Content v3 |
| **@nuxtjs/i18n** | v10.2.1+ | Internationalisation, hreflang auto |
| **Hébergement** | Cloudflare Pages | Edge deployment, bande passante illimitée, Early Hints +30% LCP |
| **Runtime** | Node.js 22 LTS | Active LTS "Jod", version stable Cloudflare Pages |
| **Package Manager** | pnpm 10.26+ | Lifecycle scripts désactivés par défaut (config via package.json ou pnpm-workspace.yaml) |

## Implications Architecturales du Stack

**Nuxt Content 3 + MiniSearch :**
- Génération statique complète (SSG) avec Worker Cloudflare pour l'API Content
- **⚠️ Cloudflare D1 OBLIGATOIRE** : le runtime Worker n'a pas accès au filesystem, il faut une base externe
- MiniSearch : index JSON généré post-build via script (`scripts/generate-search-index.mjs`)
- Bundle ~7KB minified, zéro dépendance externe
- Boosting configurable par champ, fuzzy search, prefix search natifs
- Stemming FR via option `stemmer` (snowball-stemmers)
- Filtrage multi-critères entièrement côté client
- Free tier D1 suffisant (5GB, 5M lectures/jour) - coût 0€
- Mise à jour de l'index = rebuild + redéploiement

**Nuxt 4+ Structure :**
- Nouvelle arborescence `app/` (srcDir) pour composants/pages/layouts
- `server/` et `content/` résolus depuis rootDir
- Meilleure séparation des préoccupations
- TypeScript-first avec inférence améliorée
- Hydratation lazy native (hydrate-on-visible, hydrate-on-idle, hydrate-on-interaction)
- Plus besoin de `future.compatibilityVersion: 4` (défaut)

**TailwindCSS 4 + shadcn-vue (Reka UI) :**
- Intégration via @tailwindcss/vite (module @nuxtjs/tailwindcss@6.14 supporte TW4 mais Vite recommandé)
- Configuration CSS-native avec @theme (remplace tailwind.config.js)
- Mode sombre unique = tokens oklch définis directement sans variantes dark:
- Composants shadcn copiés dans `components/ui/` - ownership total
- shadcn-vue path aliases: `@/` résout vers `app/` en Nuxt 4
- Accessibilité via Reka UI (rebrand Radix Vue février 2025) - validation manuelle WCAG requise
- Command component idéal pour la recherche (⌘K palette)
- Tests a11y en CI avec @axe-core/playwright (tooltips WCAG SC 1.4.13 non couverts)

## Technical Constraints & Dependencies

| Contrainte | Impact Architectural |
|------------|---------------------|
| Budget 0€ | Services gratuits (CF Pages + D1 free tier) |
| Solo dev | shadcn-vue = composants prêts, moins de code custom |
| Nuxt 4.2 + Content 3 | Stack moderne, SSG + Worker, MiniSearch index JSON, `asSitemapCollection()` |
| **Cloudflare D1** | OBLIGATOIRE avec preset cloudflare_pages - Worker sans accès filesystem |
| TailwindCSS 4 | @tailwindcss/vite recommandé, config CSS-native @theme |
| shadcn-vue + Reka UI | Composants accessibles, path aliases `@/` → `app/` |
| Zod 4 / Valibot | Validation schéma Content 3, Standard Schema natif |
| Cloudflare Pages | Edge global (330+ DC), bande passante illimitée, HTTP/3 manuel |
| Mode sombre unique | Tokens oklch définis directement, pas de toggle |
| Bilingue natif | @nuxtjs/i18n v10.2.1+, strategy prefix_except_default, strictSeo experimental |

## Cross-Cutting Concerns Identified

1. **Internationalization (i18n)** - @nuxtjs/i18n v10.2.1+, prefix_except_default, hreflang auto via useLocaleHead(), strictSeo experimental
2. **Performance Optimization** - SSG, hydratation lazy native Nuxt 4 (hydrate-on-*), nuxt-vitalizer 2.0+, features.inlineStyles, @nuxt/image v2
3. **Accessibility (a11y)** - Reka UI primitives, @axe-core/playwright 4.11+ CI, validation manuelle NVDA/VoiceOver
4. **SEO/GEO Dual Optimization** - llms.txt (adoption précoce, zéro visite AI crawler vérifié), FAQPage Schema, TechArticle, citations +40% visibilité
5. **Content Architecture** - MDC parsing, Zod/Valibot validation (⚠️ @zod/mini déprécié), taxonomy (piliers/catégories/tags)
6. **Client-Side Search** - MiniSearch 7.x (~7KB), index JSON post-build, boosting/fuzzy/prefix natifs, stemming FR configurable
7. **Design System** - Tailwind 4 @theme tokens oklch, shadcn-vue Reka UI, dark-first
8. **Build & Runtime** - Node.js 22 LTS, pnpm 10+ (config via package.json ou pnpm-workspace.yaml), nitro preset cloudflare_pages
9. **Sitemap Integration** - @nuxtjs/sitemap v7.5+ avec `asSitemapCollection()` wrapper pour Content v3
10. **Data Persistence** - Cloudflare D1 OBLIGATOIRE pour Nuxt Content 3 + cloudflare_pages (runtime Worker sans accès filesystem)

## Commandes Build & Déploiement

```bash
# Développement local
npx nuxi dev

# Génération statique pure (SSG)
npx nuxi generate
# Output: .output/public/

# Build hybride avec prerendering
NITRO_PRESET=cloudflare_pages npx nuxi build

# Analyse du bundle
npx nuxi analyze

# Preview local de la build
npx nuxi preview
```

**Configuration Cloudflare Pages:**

| Paramètre | Valeur |
|-----------|--------|
| Build command | `pnpm run build` ou `npx nuxi generate` |
| Output directory | `.output/public` |
| Framework preset | Nuxt (auto-détecté) |
| Node version | `22 LTS` |

**Script package.json avec MiniSearch:**

```json
{
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build",
    "generate": "nuxt generate",
    "postgenerate": "node scripts/generate-search-index.mjs",
    "preview": "nuxt preview",
    "analyze": "nuxt analyze"
  }
}
```

## Configuration Runtime Sécurisée

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    // Privé (serveur uniquement) - NUXT_API_SECRET
    apiSecret: '',
    contentApiKey: '',

    // Public (exposé au client) - NUXT_PUBLIC_*
    public: {
      siteUrl: 'https://monblog.fr',
      siteName: 'Mon Blog Nuxt',
      googleAnalyticsId: '',
      postsPerPage: 10
    }
  }
})
```

**Variables d'environnement:**

```env
# .env
NUXT_API_SECRET=secret-api-key
NUXT_CONTENT_API_KEY=content-key
NUXT_PUBLIC_SITE_URL=https://monblog.fr
NUXT_PUBLIC_GOOGLE_ANALYTICS_ID=G-XXXXXXXXXX
```
