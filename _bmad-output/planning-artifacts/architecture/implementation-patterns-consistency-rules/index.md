# Implementation Patterns & Consistency Rules

Patterns d'implémentation et règles de cohérence pour le projet Nuxt 4 SSG.

## Patterns Découpés (sous-dossiers)

Ces patterns volumineux ont été découpés en fichiers auto-suffisants pour une meilleure utilisation par Claude Code.

| Pattern | Index | Description |
|---------|-------|-------------|
| **Content Query** | [→ index](./content-query-patterns/index.md) | 22 patterns de requêtes @nuxt/content v3 |
| **Search (MiniSearch)** | [→ index](./search-patterns-minisearch/index.md) | 21 patterns de recherche full-text |
| **Accessibility** | [→ index](./shadcn-reka-accessibility-patterns/index.md) | 29 patterns shadcn-vue & Reka UI a11y |
| **TailwindCSS 4** | [→ index](./tailwindcss-patterns/index.md) | 18 patterns CSS-first, theming |
| **Performance** | [→ index](./performance-patterns/index.md) | 11 patterns INP/LCP/CLS |
| **SEO** | [→ index](./seo-patterns/index.md) | 20 patterns meta tags, Schema.org, GEO |
| **Zod Schemas** | [→ index](./zod-schema-composition-patterns/index.md) | 13 patterns composition et validation |
| **Testing** | [→ index](./testing-patterns/index.md) | 16 patterns Vitest, @nuxt/test-utils |
| **MDC Syntax** | [→ index](./mdc-syntax-patterns/index.md) | 9 patterns Markdown components |
| **i18n** | [→ index](./i18n-patterns/index.md) | 14 patterns internationalisation |

## Patterns Simples (fichiers uniques)

### Structure & Naming

| Pattern | Description |
|---------|-------------|
| [Pattern Categories](./pattern-categories-defined.md) | Définition des catégories de patterns |
| [Naming Patterns](./naming-patterns.md) | Conventions de nommage |
| [Script Setup Structure](./script-setup-structure-pattern.md) | Structure `<script setup>` |
| [Structure Patterns](./structure-patterns.md) | Patterns de structure de fichiers |
| [Format Patterns](./format-patterns.md) | Formatage et conventions |

### Vue & Nuxt

| Pattern | Description |
|---------|-------------|
| [Vue 3.4+/3.5+ APIs](./vue-34-35-apis.md) | APIs modernes Vue (shallowRef, v-memo, etc.) |
| [Communication Patterns](./communication-patterns.md) | Communication entre composants |
| [Page Routing Patterns](./page-routing-patterns.md) | Patterns de routing Nuxt |
| [Middleware Patterns](./middleware-patterns.md) | Middleware Nuxt |

### Plugins & Server

| Pattern | Description |
|---------|-------------|
| [Plugin Patterns](./plugin-patterns.md) | Création de plugins Nuxt |
| [Plugin Suffixes & Context](./plugin-suffixes-context.md) | `.client`, `.server` et contexte |
| [Plugin TypeScript Declarations](./plugin-typescript-declarations.md) | Déclarations TypeScript |
| [Server Components Patterns](./server-components-patterns.md) | Composants serveur |
| [ClientOnly Patterns](./clientonly-patterns.md) | Composants client-only |

### Autres

| Pattern | Description |
|---------|-------------|
| [Process Patterns](./process-patterns.md) | Processus et workflows |
| [Prose Components Patterns](./prose-components-patterns.md) | Composants MDC Prose |
| [Security Patterns](./security-patterns.md) | Sécurité (DOMPurify, CSP, XSS) |
| [Enforcement Guidelines](./enforcement-guidelines.md) | Règles d'application |
| [Erreurs D1 Courantes](./erreurs-d1-courantes-et-solutions.md) | Solutions aux erreurs D1 |

---

## Statistiques

- **Sous-dossiers patterns** : 10 dossiers
- **Fichiers patterns simples** : 20 fichiers
- **Total fichiers** : ~200 fichiers markdown
