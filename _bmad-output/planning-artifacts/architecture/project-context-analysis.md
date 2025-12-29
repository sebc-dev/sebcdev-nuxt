# Project Context Analysis

## Requirements Overview

**Functional Requirements:**
50 FRs couvrant un blog technique bilingue avec système de navigation avancé, filtrage multi-critères, expérience de lecture enrichie (ToC, progression, code blocks), et optimisation GEO pour visibilité dans les moteurs IA.

**Non-Functional Requirements:**
- Performance: Lighthouse 100/100/100/100, LCP < 2.5s, CLS < 0.1
- Accessibilité: WCAG 2.1 AA, navigation clavier 100%
- SEO: Score 100, Schema.org validé, crawl < 500ms
- Sécurité: HTTPS obligatoire, headers CSP, Mozilla Observatory B+
- Disponibilité: 99.5% uptime mensuel

**Scale & Complexity:**

- Primary domain: Full-stack Content Platform (Nuxt 4 Static)
- Complexity level: Medium-High
- Estimated architectural components: ~8-12 modules principaux

## Technical Stack Decisions

| Technologie | Version | Rôle |
|-------------|---------|------|
| **Nuxt** | 4.2.2 | Framework full-stack, SSG mode |
| **Nuxt Content** | 3.10.0+ | CMS fichiers Markdown/MDC |
| **Search Engine** | Pagefind 1.4.0+ | Recherche full-text post-build (~8KB core entry + 36KB API client + chunks) |
| **TailwindCSS** | 4.1.17 | Styling utility-first via @tailwindcss/vite |
| **shadcn-vue** | 2.4.3+ | Composants UI accessibles (Reka UI) |
| **Validation** | Zod 4 | Validation schéma (~5KB gzip / ~10KB min) ou Valibot (~1KB gzip / ~2KB min) + Standard Schema |
| **Hébergement** | Cloudflare Pages | Edge deployment, bande passante illimitée, Early Hints +30% LCP |
| **Runtime** | Node.js 22 LTS | Active LTS "Jod", version stable Cloudflare Pages |
| **Package Manager** | pnpm 10+ | Lifecycle scripts désactivés par défaut (.npmrc requis) |

## Implications Architecturales du Stack

**Nuxt Content 3 + Pagefind :**
- Génération statique complète (SSG) - pas de server functions pour le contenu
- Pagefind indexe le HTML post-build (script `package.json`: `nuxt build && npx pagefind...`)
- Bundle core ~8KB + index chargé par chunks (~40KB/chunk), <300KB total pour 10k pages
- Stemming natif français/anglais via détection automatique attribut HTML `lang`
- Filtrage multi-critères entièrement côté client
- Zéro coût serveur (aligne parfaitement avec budget 0€)
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
| Budget 0€ | Architecture 100% statique, services gratuits |
| Solo dev | shadcn-vue = composants prêts, moins de code custom |
| Nuxt 4.2 + Content 3 | Stack moderne, SSG, Pagefind post-build, `asSitemapCollection()` |
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
6. **Client-Side Search** - Pagefind 1.4+ post-build (~8KB + chunks), script package.json, stemming FR/EN natif
7. **Design System** - Tailwind 4 @theme tokens oklch, shadcn-vue Reka UI, dark-first
8. **Build & Runtime** - Node.js 22 LTS, pnpm 10+ (config via package.json ou pnpm-workspace.yaml), nitro preset cloudflare_pages
9. **Sitemap Integration** - @nuxtjs/sitemap v7.5+ avec `asSitemapCollection()` wrapper pour Content v3
