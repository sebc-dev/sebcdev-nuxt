# Core Architectural Decisions

## Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Content structure: By language with single collection
- Frontmatter schema: English constants, auto-calculated readingTime
- Component architecture: Structured folders with custom Prose components
- SEO/GEO: Auto-generated llms.txt, Schema.org via @unhead

**Important Decisions (Shape Architecture):**
- State management: Vue composables only (no Pinia)
- Image optimization: @nuxt/image + Cloudflare provider
- CI/CD: Cloudflare Pages direct Git integration

**Deferred Decisions (Post-MVP):**
- Newsletter integration
- Comments system
- Advanced analytics (heatmaps)

## Content Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **File Organization** | By language (`content/fr/`, `content/en/`) | Natural fit with @nuxtjs/i18n, clear separation |
| **Collections** | Single `articles` collection | Simpler queries, filter by pillar |
| **Frontmatter Constants** | English (`ai`, `tutorial`, `beginner`) | Consistency, i18n-agnostic |
| **Reading Time** | Auto-calculated (200 words/min) | Less maintenance, always accurate |

**Frontmatter Schema:**

```yaml
title: string
description: string
slug: string
pillar: 'ai' | 'engineering' | 'ux'
category: 'news' | 'tutorial' | 'deep-dive' | 'case-study' | 'retrospective'
level: 'all' | 'beginner' | 'intermediate' | 'advanced'
tags: string[]
publishedAt: date
updatedAt: date
image: string
draft: boolean
```

## Frontend Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Component Structure** | `ui/`, `content/`, `layout/`, `search/` | Clear separation of concerns |
| **State Management** | Vue composables only | SSG blog doesn't need global store |
| **Prose Components** | Custom (ProseCode, ProseH2, ProseDetails) | Required for FR11-13 (code blocks), FR8 (ToC) |

**Component Organization:**

```
app/components/
├── ui/                    # shadcn-vue
├── content/               # ArticleCard, TableOfContents, ReadingProgress, CodeBlock
├── layout/                # TheHeader, TheFooter, LanguageSwitcher
└── search/                # SearchCommand, SearchFilters
```

## SEO & GEO Implementation

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **llms.txt** | Auto-generated at build | Always current, zero maintenance |
| **Schema Markup** | nuxt-schema-org | Type-safe, TechArticle + FAQ support, Content 3 integration |
| **Sitemap** | @nuxtjs/sitemap | Official module, hreflang support |
| **RSS** | Server route generation | Native Content 3 integration |

## Infrastructure & Deployment

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Image Optimization** | @nuxt/image + Cloudflare provider | Auto WebP, lazy loading, CDN |
| **CI/CD** | Cloudflare Pages direct | Zero config, preview branches, fast |
| **Environment Config** | .env + runtimeConfig + CF fallback | Local dev + prod parity |

**Deployment Flow:**

```
Git Push → Cloudflare Pages Build → SSG Output → Edge Deployment
                ↓
         Preview URL (branches)
```

## Modules Nuxt Finaux

```typescript
modules: [
  '@nuxt/content',
  '@nuxtjs/i18n',
  '@nuxt/image',
  '@nuxtjs/sitemap',
  'nuxt-schema-org',
  'shadcn-nuxt',
]
```

## Validation Strategy

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Schema Validator** | Zod v4 | Standard Schema, JSON Schema natif sans dépendance extra |
| **Import Source** | `import { z } from 'zod'` | Re-export `@nuxt/content` déprécié, sera supprimé |
| **Validation Scope** | Build time | Erreurs détectées au build, pas en production |
| **Type Inference** | `z.infer<typeof schema>` | Types TypeScript générés automatiquement |
