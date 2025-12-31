# Epic 1: Core Reading Experience

**Objectif :** Un visiteur peut lire un article complet sur une page dédiée avec un design responsive et sombre.

## Story 1.1: Project Foundation & First Article Display

As a visiteur,
I want to see a blog article displayed on a dedicated page,
So that I can read technical content.

**Acceptance Criteria:**

**Given** le projet Nuxt 4 est initialisé avec la structure `app/`
**When** je navigue vers `/articles/test-article`
**Then** l'article est affiché avec le contenu Markdown rendu (titres, paragraphes, listes, liens)
**And** les images ont un attribut alt et sont lazy-loaded
**And** le temps de chargement initial est < 2s (LCP)

**Technical Scope:**
- Initialisation Nuxt 4.2.2 avec pnpm 10.26+
- Configuration Nuxt Content 3.10+ avec content.config.ts
- Structure `app/`, `content/`, `server/`, `shared/`
- Page `app/pages/articles/[slug].vue`
- Un article de test dans `content/`
- Configuration Cloudflare D1 (wrangler.toml)

**FRs:** FR1
**ARCH:** ARCH-1, ARCH-2, ARCH-4, ARCH-5, ARCH-6, ARCH-7, ARCH-14

---

## Story 1.2: Dark Theme & Typography System

As a visiteur,
I want to see a visually consistent dark-themed interface with professional typography,
So that I have a comfortable reading experience.

**Acceptance Criteria:**

**Given** le site utilise un thème sombre unique
**When** je visite n'importe quelle page
**Then** le fond est sombre (#0A0A0B) avec du texte clair (#FAFAFA)
**And** le contraste texte/fond est ≥ 4.5:1 (WCAG AA)
**And** la police Satoshi est utilisée pour le texte principal
**And** la police JetBrains Mono est utilisée pour le code
**And** aucun toggle clair/sombre n'est présent

**Technical Scope:**
- TailwindCSS 4.1.17 via @tailwindcss/vite
- Tokens oklch dans `app/assets/css/main.css` avec @theme
- Intégration fonts Satoshi Variable + JetBrains Mono
- shadcn-vue 2.4.3+ initialisé avec Reka UI

**FRs:** FR46 (partiel)
**ARCH:** ARCH-8, ARCH-9, ARCH-16
**UX:** UX-1, UX-3, UX-4, UX-12

---

## Story 1.3: Responsive Layout System

As a visiteur,
I want the site to adapt to my device (mobile, tablet, desktop),
So that I can read articles comfortably on any screen.

**Acceptance Criteria:**

**Given** je visite le site sur desktop (≥ 1024px)
**When** je lis un article
**Then** le layout utilise un container max-width (1200px) centré
**And** le contenu a une largeur optimale pour la lecture (720px)

**Given** je visite le site sur tablette (768-1023px)
**When** je lis un article
**Then** le layout s'adapte avec des marges réduites

**Given** je visite le site sur mobile (< 768px)
**When** je lis un article
**Then** le layout est en pleine largeur avec padding approprié
**And** les éléments interactifs ont une taille ≥ 44px

**Technical Scope:**
- Breakpoints TailwindCSS 4 : sm (640px), md (768px), lg (1024px)
- Layout article avec container responsive
- Focus states 2px ring accent
- Support `prefers-reduced-motion`

**FRs:** FR46 (complet), FR47, FR48, FR49
**UX:** UX-5, UX-13, UX-14, UX-15

---

## Story 1.4: Cloudflare Pages Deployment

As a visiteur,
I want the site to load quickly from anywhere in the world,
So that I have a fast, reliable reading experience.

**Acceptance Criteria:**

**Given** le site est déployé sur Cloudflare Pages
**When** je visite le site depuis n'importe quelle région
**Then** le TTFB est < 200ms
**And** le site est servi via HTTPS
**And** les headers de sécurité (CSP, X-Frame-Options) sont configurés

**Technical Scope:**
- Configuration Cloudflare Pages dans `wrangler.toml`
- Fichier `public/_headers` pour sécurité
- GitHub Actions CI/CD
- Preset nitro `cloudflare_pages`

**ARCH:** ARCH-3
**NFRs:** NFR4, NFR20, NFR21, NFR22, NFR23

---
