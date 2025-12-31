# Architecture Decision Document

Documentation d'architecture pour le projet Nuxt 4 SSG multilingue déployé sur Cloudflare Pages.

## Structure

```
architecture/
├── project-context-analysis.md          # Analyse du contexte projet
├── project-structure-boundaries.md      # Structure et frontières
├── starter-template-evaluation/         # Évaluation et configuration du starter (26 fichiers)
├── core-architectural-decisions/        # Décisions architecturales (22 fichiers)
└── implementation-patterns-consistency-rules/  # Patterns d'implémentation (10+ sous-dossiers)
```

## Navigation

### Documents Racine

| Document | Description |
|----------|-------------|
| [Project Context Analysis](./project-context-analysis.md) | Requirements, stack technique, contraintes |
| [Project Structure & Boundaries](./project-structure-boundaries.md) | Arborescence, frontières, intégrations |

### Starter Template Evaluation

Configuration et évaluation du starter Nuxt 4 : [Index](./starter-template-evaluation/index.md)

| Section | Description |
|---------|-------------|
| [Cloudflare D1](./starter-template-evaluation/cloudflare-d1-requis-pour-nuxt-content-3.md) | Configuration D1 obligatoire |
| [CI/CD GitHub Actions](./starter-template-evaluation/cicd-github-actions-recommand.md) | Pipeline de déploiement |
| [Configuration nuxt.config.ts](./starter-template-evaluation/configuration-nuxtconfigts-de-base.md) | Configuration de base |
| [Arborescence Projet](./starter-template-evaluation/arborescence-projet-nuxt-4.md) | Structure des dossiers |
| [Sécurité Cloudflare](./starter-template-evaluation/scurit-cloudflare-ssg.md) | Headers et CSP |
| [Cache Strategies](./starter-template-evaluation/stratgies-de-cache-avances-cloudflare.md) | Stratégies de cache |

### Core Architectural Decisions

Décisions architecturales fondamentales : [Index](./core-architectural-decisions/index.md)

| Section | Description |
|---------|-------------|
| [Decision Priority](./core-architectural-decisions/decision-priority-analysis.md) | Priorités des décisions |
| [Content Architecture](./core-architectural-decisions/content-architecture.md) | Structure du contenu |
| [Frontend Architecture](./core-architectural-decisions/frontend-architecture.md) | Architecture frontend |
| [SEO & GEO](./core-architectural-decisions/seo-geo-implementation.md) | Implémentation SEO/GEO |
| [Validation Strategy](./core-architectural-decisions/validation-strategy.md) | Stratégie de validation |
| [Breaking Changes Nuxt 4](./core-architectural-decisions/breaking-changes-nuxt-3-nuxt-4.md) | Migration Nuxt 3→4 |

### Implementation Patterns & Consistency Rules

Patterns d'implémentation : [Index](./implementation-patterns-consistency-rules/index.md)

#### Patterns Découpés (sous-dossiers)

| Pattern | Fichiers | Description |
|---------|----------|-------------|
| [Content Query](./implementation-patterns-consistency-rules/content-query-patterns/index.md) | 22 | Requêtes @nuxt/content |
| [Search (MiniSearch)](./implementation-patterns-consistency-rules/search-patterns-minisearch/index.md) | 21 | Recherche full-text |
| [Accessibility](./implementation-patterns-consistency-rules/shadcn-reka-accessibility-patterns/index.md) | 29 | shadcn-vue & Reka UI a11y |
| [TailwindCSS 4](./implementation-patterns-consistency-rules/tailwindcss-patterns/index.md) | 18 | Configuration et patterns |
| [Performance](./implementation-patterns-consistency-rules/performance-patterns/index.md) | 11 | Optimisation INP/LCP/CLS |
| [SEO](./implementation-patterns-consistency-rules/seo-patterns/index.md) | 20 | Meta tags, Schema.org |
| [Zod Schemas](./implementation-patterns-consistency-rules/zod-schema-composition-patterns/index.md) | 13 | Composition et validation |
| [Testing](./implementation-patterns-consistency-rules/testing-patterns/index.md) | 16 | Vitest, @nuxt/test-utils |
| [MDC Syntax](./implementation-patterns-consistency-rules/mdc-syntax-patterns/index.md) | 9 | Markdown components |
| [i18n](./implementation-patterns-consistency-rules/i18n-patterns/index.md) | 14 | Internationalisation |

#### Patterns Simples (fichiers uniques)

- [Naming Patterns](./implementation-patterns-consistency-rules/naming-patterns.md)
- [Structure Patterns](./implementation-patterns-consistency-rules/structure-patterns.md)
- [Vue 3.4+/3.5+ APIs](./implementation-patterns-consistency-rules/vue-34-35-apis.md)
- [Plugin Patterns](./implementation-patterns-consistency-rules/plugin-patterns.md)
- [Middleware Patterns](./implementation-patterns-consistency-rules/middleware-patterns.md)
- [Security Patterns](./implementation-patterns-consistency-rules/security-patterns.md)
- [ClientOnly Patterns](./implementation-patterns-consistency-rules/clientonly-patterns.md)
- [Enforcement Guidelines](./implementation-patterns-consistency-rules/enforcement-guidelines.md)

---

## Statistiques

- **Fichiers totaux** : ~245 fichiers markdown
- **Répertoires** : 14 dossiers
- **Patterns documentés** : 20+ catégories
