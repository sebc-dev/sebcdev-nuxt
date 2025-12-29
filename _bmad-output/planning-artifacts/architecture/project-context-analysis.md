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
| **Nuxt** | 4+ | Framework full-stack, SSG mode |
| **Nuxt Content** | 3 | CMS fichiers Markdown/MDC |
| **Content Dump** | Compressé | Bundle statique téléchargé client |
| **Search Engine** | WebAssembly | SQLite WASM pour recherche full-text client-side |
| **TailwindCSS** | 4 | Styling utility-first, config CSS-native |
| **shadcn-vue** | Latest | Composants UI accessibles (Radix Vue) |
| **Hébergement** | Cloudflare Pages | Edge deployment, 100% statique |

## Implications Architecturales du Stack

**Nuxt Content 3 + WASM Search :**
- Génération statique complète (SSG) - pas de server functions pour le contenu
- Le client télécharge le dump compressé au premier chargement
- Recherche instantanée sans latence réseau après chargement initial
- Filtrage multi-critères entièrement côté client
- Zéro coût serveur (aligne parfaitement avec budget 0€)
- Mise à jour du dump = rebuild + redéploiement

**Nuxt 4+ Structure :**
- Nouvelle arborescence `app/` pour composants/pages/layouts
- `server/` pour API routes (minimal dans notre cas)
- Meilleure séparation des préoccupations
- TypeScript-first avec inférence améliorée

**TailwindCSS 4 + shadcn-vue :**
- Design system cohérent via CSS variables Tailwind
- Mode sombre unique = configuration simplifiée (pas de toggle)
- Composants shadcn copiés dans `components/ui/` - ownership total
- Accessibilité WCAG AA incluse via Radix Vue primitives
- Command component idéal pour la recherche (⌘K palette)
- Dropdown menus pour navigation header (thèmes, catégories, niveaux)

## Technical Constraints & Dependencies

| Contrainte | Impact Architectural |
|------------|---------------------|
| Budget 0€ | Architecture 100% statique, services gratuits |
| Solo dev | shadcn-vue = composants prêts, moins de code custom |
| Nuxt 4 + Content 3 | Stack moderne, SSG, client-side search |
| TailwindCSS 4 | CSS-first config, design tokens via variables |
| shadcn-vue | Composants accessibles, copy-paste ownership |
| Zod v4 | Validation schéma Content 3, Standard Schema |
| Cloudflare Pages | Edge deployment, pas de server compute |
| Mode sombre unique | Palette dark fixe, pas de theme switching |
| Bilingue natif | i18n intégré, contenu FR/EN dans Content |

## Cross-Cutting Concerns Identified

1. **Internationalization (i18n)** - Routes, contenu dual-language, UI labels, SEO hreflang
2. **Performance Optimization** - SSG, preloading dump, lazy hydration, image optimization
3. **Accessibility (a11y)** - shadcn/Radix primitives, keyboard nav, ARIA, contrasts
4. **SEO/GEO Dual Optimization** - Pre-rendered HTML, Schema, llms.txt, Answer-First
5. **Content Architecture** - MDC parsing, frontmatter schema, taxonomy (piliers/catégories/tags)
6. **Client-Side Search** - Dump size management, WASM loading, Command palette UX
7. **Design System** - Tailwind 4 tokens, shadcn components, cohérence visuelle
