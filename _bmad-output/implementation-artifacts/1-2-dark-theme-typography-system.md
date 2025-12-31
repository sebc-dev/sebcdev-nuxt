# Story 1.2: Dark Theme & Typography System

Status: ready-for-dev

## Story

As a visiteur,
I want to see a visually consistent dark-themed interface with professional typography,
So that I have a comfortable reading experience.

## Acceptance Criteria

1. **AC1 - Fond sombre** : Le fond principal est sombre (#0A0A0B / oklch(0.067 0 0)) avec du texte clair (#FAFAFA)
2. **AC2 - Contraste WCAG AA** : Le contraste texte/fond est ≥ 4.5:1 (valeur atteinte: 7.2:1 minimum)
3. **AC3 - Police Satoshi** : La police Satoshi Variable est utilisée pour le texte principal (titres, paragraphes)
4. **AC4 - Police JetBrains Mono** : La police JetBrains Mono Variable est utilisée pour le code
5. **AC5 - Pas de toggle** : Aucun toggle clair/sombre n'est présent (thème sombre uniquement)
6. **AC6 - shadcn-vue initialisé** : shadcn-vue 2.4.3+ est initialisé avec Reka UI et configuré pour le dark mode

## Tasks / Subtasks

- [ ] **Task 1: Configuration TailwindCSS 4 avec tokens oklch** (AC: #1, #2)
  - [ ] 1.1 Mettre à jour `app/assets/css/main.css` avec la configuration @theme complète
  - [ ] 1.2 Définir le custom-variant dark: `@custom-variant dark (&:is(.dark *));`
  - [ ] 1.3 Configurer les tokens de couleur shadcn-vue complets (backgrounds, foregrounds, accents)
  - [ ] 1.4 Ajouter les couleurs piliers (IA violet, Engineering bleu, UX rose)
  - [ ] 1.5 Configurer les tokens de focus accessibles (double-ring pattern)

- [ ] **Task 2: Installation et configuration des fonts** (AC: #3, #4)
  - [ ] 2.1 Télécharger Satoshi Variable (.woff2) dans `public/fonts/`
  - [ ] 2.2 Télécharger JetBrains Mono Variable (.woff2) dans `public/fonts/`
  - [ ] 2.3 Ajouter les @font-face avec `font-display: optional` (anti-CLS)
  - [ ] 2.4 Configurer le preload des fonts dans nuxt.config.ts
  - [ ] 2.5 Définir --font-sans et --font-mono dans @theme

- [ ] **Task 3: Initialisation shadcn-vue** (AC: #6)
  - [ ] 3.1 Exécuter `pnpm dlx shadcn-vue@latest init`
  - [ ] 3.2 Vérifier que components.json pointe vers `app/components/ui`
  - [ ] 3.3 Créer l'utilitaire `app/utils/cn.ts` (class merge)
  - [ ] 3.4 Installer tw-animate-css pour les animations shadcn

- [ ] **Task 4: Application du thème dark-only** (AC: #1, #5)
  - [ ] 4.1 Ajouter la classe `dark` sur l'élément `<html>` dans app.vue
  - [ ] 4.2 Configurer les styles de base dans @layer base
  - [ ] 4.3 Supprimer tout toggle de thème existant
  - [ ] 4.4 Configurer scroll-padding-top pour la navigation ToC future

- [ ] **Task 5: Configuration typographique** (AC: #3, #4)
  - [ ] 5.1 Définir l'échelle typographique (text-base à text-5xl)
  - [ ] 5.2 Configurer les line-heights optimaux pour chaque taille
  - [ ] 5.3 Appliquer la font-sans par défaut sur body
  - [ ] 5.4 Appliquer la font-mono sur les éléments code/pre

- [ ] **Task 6: Styles de base et accessibilité** (AC: #2)
  - [ ] 6.1 Configurer le focus-visible global avec double-ring
  - [ ] 6.2 Ajouter le support prefers-reduced-motion
  - [ ] 6.3 Vérifier les contrastes avec les outils de vérification

- [ ] **Task 7: Validation visuelle** (AC: #1-6)
  - [ ] 7.1 Vérifier le rendu du thème sombre sur la page article
  - [ ] 7.2 Vérifier que Satoshi est appliquée au texte
  - [ ] 7.3 Vérifier que JetBrains Mono est appliquée au code
  - [ ] 7.4 Vérifier l'absence de CLS au chargement

## Dev Notes

### Prérequis de la Story 1.1

Cette story s'appuie sur le projet initialisé dans la Story 1.1:
- Projet Nuxt 4.2.2 fonctionnel
- TailwindCSS 4.1.17 via @tailwindcss/vite
- Structure app/ avec main.css
- Page article [slug].vue affichant du contenu

### Configuration main.css Complète

```css
/* app/assets/css/main.css */

/* ============================================
   FONTS - Avant @import tailwindcss
   font-display: optional élimine 100% du CLS
   ============================================ */
@font-face {
  font-family: "Satoshi Variable";
  font-style: normal;
  font-weight: 100 900;
  font-display: optional;
  src: url("/fonts/Satoshi-Variable.woff2") format("woff2-variations");
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6,
                 U+02DA, U+02DC, U+2000-206F, U+20AC, U+2122, U+FEFF;
}

@font-face {
  font-family: "JetBrains Mono Variable";
  font-style: normal;
  font-weight: 100 800;
  font-display: optional;
  src: url("/fonts/JetBrainsMono-Variable.woff2") format("woff2-variations");
}

@import "tailwindcss";
@import "tw-animate-css";

/* Scanner les fichiers Markdown pour les classes Tailwind */
@source "../../../content/**/*";

/* OBLIGATOIRE: définit le variant dark pour shadcn-vue */
@custom-variant dark (&:is(.dark *));

/* ============================================
   ROOT TOKENS - Light mode (pour complétude)
   ============================================ */
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.145 0 0);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --destructive-foreground: oklch(0.985 0 0);
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --ring: oklch(0.708 0 0);
  --radius: 0.375rem;
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;

  /* Layout - UX Spec aligned */
  --content-max-width: 720px;
  --toc-width: 240px;
  --container-max: 1200px;
  --layout-gap: 48px;
  --header-height: 64px;

  /* Focus tokens */
  --focus-ring: oklch(0.585 0.233 277.117);
  --focus-inner: oklch(1 0 0);
  --focus-outer: oklch(0.205 0 0);
}

/* ============================================
   DARK MODE - sebc.dev Theme (Dark-Only)
   Valeurs alignées avec UX Specification
   ============================================ */
.dark {
  /* Backgrounds - UX Spec aligned */
  --background: oklch(0.067 0 0);           /* #0A0A0B */
  --background-secondary: oklch(0.078 0 0); /* #141415 */
  --background-tertiary: oklch(0.118 0 0);  /* #1E1E20 */

  /* Foreground - UX Spec aligned */
  --foreground: oklch(0.98 0 0);            /* #FAFAFA */
  --foreground-muted: oklch(0.67 0.01 286); /* #A1A1AA */

  /* Composants */
  --card: oklch(0.078 0 0);
  --card-foreground: oklch(0.98 0 0);
  --popover: oklch(0.078 0 0);
  --popover-foreground: oklch(0.98 0 0);

  /* Primary/Secondary */
  --primary: oklch(0.98 0 0);
  --primary-foreground: oklch(0.067 0 0);
  --secondary: oklch(0.118 0 0);
  --secondary-foreground: oklch(0.98 0 0);

  /* Muted */
  --muted: oklch(0.118 0 0);
  --muted-foreground: oklch(0.67 0.01 286);

  /* Accent - Teal #14B8A6 */
  --accent: oklch(0.696 0.143 175.8);
  --accent-foreground: oklch(0.067 0 0);

  /* États fonctionnels */
  --destructive: oklch(0.628 0.258 29.234); /* #EF4444 */
  --destructive-foreground: oklch(0.98 0 0);
  --success: oklch(0.696 0.17 142.5);       /* #22C55E */
  --warning: oklch(0.795 0.184 86.047);     /* #EAB308 */

  /* Bordures et inputs */
  --border: oklch(0.185 0 0);               /* #27272A */
  --input: oklch(0.185 0 0);
  --ring: oklch(0.696 0.143 175.8);         /* Accent teal */

  /* Couleurs des Piliers sebc.dev */
  --pillar-ia: oklch(0.586 0.232 292);      /* #8B5CF6 - Violet */
  --pillar-engineering: oklch(0.588 0.213 254); /* #3B82F6 - Bleu */
  --pillar-ux: oklch(0.656 0.241 354);      /* #EC4899 - Rose */

  /* Focus tokens dark */
  --focus-ring: oklch(0.673 0.182 276.935);
  --focus-inner: oklch(0.145 0 0);
  --focus-outer: oklch(0.985 0 0);
}

/* ============================================
   TAILWIND @theme inline - Bridge CSS vars to utilities
   ============================================ */
@theme inline {
  /* Fonts */
  --font-sans: "Satoshi Variable", ui-sans-serif, system-ui, sans-serif;
  --font-mono: "JetBrains Mono Variable", ui-monospace, Menlo, monospace;

  /* Backgrounds */
  --color-background: var(--background);
  --color-background-secondary: var(--background-secondary);
  --color-background-tertiary: var(--background-tertiary);
  --color-foreground: var(--foreground);
  --color-foreground-muted: var(--foreground-muted);

  /* Composants */
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-popover: var(--popover);
  --color-popover-foreground: var(--popover-foreground);

  /* Primary/Secondary */
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);

  /* Muted */
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);

  /* Accent & States */
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-destructive-foreground: var(--destructive-foreground);
  --color-success: var(--success);
  --color-warning: var(--warning);

  /* Bordures */
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);

  /* Piliers sebc.dev */
  --color-pillar-ia: var(--pillar-ia);
  --color-pillar-engineering: var(--pillar-engineering);
  --color-pillar-ux: var(--pillar-ux);

  /* Focus */
  --color-focus-ring: var(--focus-ring);
  --color-focus-inner: var(--focus-inner);
  --color-focus-outer: var(--focus-outer);

  /* Radius */
  --radius-sm: var(--radius-sm);
  --radius-DEFAULT: var(--radius);
  --radius-md: var(--radius-md);
  --radius-lg: var(--radius-lg);
}

/* ============================================
   BASE LAYER - Global styles
   ============================================ */
@layer base {
  html {
    scroll-padding-top: var(--header-height);
    scroll-behavior: smooth;
  }

  body {
    @apply bg-background text-foreground font-sans antialiased;
  }

  /* Focus global double-ring accessible */
  *:focus-visible {
    outline: 2px solid var(--focus-inner);
    outline-offset: 0;
    box-shadow: 0 0 0 4px var(--focus-outer);
  }

  /* Code elements */
  code, pre, kbd, samp {
    @apply font-mono;
  }

  /* Respect prefers-reduced-motion */
  @media (prefers-reduced-motion: reduce) {
    *, ::before, ::after {
      animation-duration: 0.01ms !important;
      transition-duration: 0.01ms !important;
      scroll-behavior: auto !important;
    }
  }
}
```

### Preload des fonts dans nuxt.config.ts

```typescript
// nuxt.config.ts - Ajouter dans app.head.link
app: {
  head: {
    htmlAttrs: {
      class: 'dark',  // Thème sombre uniquement
    },
    link: [
      {
        rel: 'preload',
        href: '/fonts/Satoshi-Variable.woff2',
        as: 'font',
        type: 'font/woff2',
        crossorigin: 'anonymous'
      },
      {
        rel: 'preload',
        href: '/fonts/JetBrainsMono-Variable.woff2',
        as: 'font',
        type: 'font/woff2',
        crossorigin: 'anonymous'
      }
    ]
  }
}
```

### Utilitaire cn.ts pour shadcn-vue

```typescript
// app/utils/cn.ts
import { type ClassValue, clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

### Installation shadcn-vue

```bash
# Initialiser shadcn-vue
pnpm dlx shadcn-vue@latest init

# Installer les dépendances pour cn()
pnpm add clsx tailwind-merge

# Installer tw-animate-css pour animations
pnpm add tw-animate-css
```

### components.json attendu

```json
{
  "$schema": "https://shadcn-vue.com/schema.json",
  "style": "default",
  "typescript": true,
  "tailwind": {
    "config": "",
    "css": "app/assets/css/main.css",
    "baseColor": "zinc"
  },
  "framework": "nuxt",
  "aliases": {
    "components": "@/components",
    "utils": "@/utils",
    "ui": "@/components/ui",
    "lib": "@/lib"
  }
}
```

**Note importante**: En Nuxt 4, `@/` résout vers `app/` grâce au srcDir.

### Téléchargement des fonts

**Satoshi Variable:**
- Source: https://www.fontshare.com/fonts/satoshi
- Télécharger: Satoshi-Variable.woff2
- Placer dans: `public/fonts/Satoshi-Variable.woff2`

**JetBrains Mono Variable:**
- Source: https://www.jetbrains.com/lp/mono/
- Télécharger: JetBrainsMono-Variable.woff2
- Placer dans: `public/fonts/JetBrainsMono-Variable.woff2`

### Palette de couleurs UX alignée

| Token | Valeur oklch | Hex | Usage |
|-------|--------------|-----|-------|
| `--background` | `oklch(0.067 0 0)` | #0A0A0B | Fond principal |
| `--background-secondary` | `oklch(0.078 0 0)` | #141415 | Cartes, panneaux |
| `--foreground` | `oklch(0.98 0 0)` | #FAFAFA | Texte principal |
| `--foreground-muted` | `oklch(0.67 0.01 286)` | #A1A1AA | Texte secondaire |
| `--accent` | `oklch(0.696 0.143 175.8)` | #14B8A6 | Liens, focus, CTA |
| `--border` | `oklch(0.185 0 0)` | #27272A | Bordures |
| `--pillar-ia` | `oklch(0.586 0.232 292)` | #8B5CF6 | Pilier IA (violet) |
| `--pillar-engineering` | `oklch(0.588 0.213 254)` | #3B82F6 | Pilier Engineering (bleu) |
| `--pillar-ux` | `oklch(0.656 0.241 354)` | #EC4899 | Pilier UX (rose) |

### Anti-patterns à éviter

- ❌ Ne PAS utiliser `font-display: swap` - cause du CLS
- ❌ Ne PAS oublier `crossorigin="anonymous"` sur le preload des fonts
- ❌ Ne PAS utiliser radix-vue - rebrandé en reka-ui
- ❌ Ne PAS utiliser HSL - OKLCH est plus uniforme perceptuellement
- ❌ Ne PAS oublier @custom-variant dark - shadcn-vue ne fonctionnera pas

### Apprentissages de la Story 1.1

À appliquer dans cette story:
- Structure `app/` confirmée fonctionnelle
- @tailwindcss/vite configuré
- content/ et server/ à la racine (pas dans app/)

### References

- [Source: ux-design-specification/visual-design-foundation.md] - Palette couleurs, typographie
- [Source: architecture/implementation-patterns-consistency-rules/tailwindcss-patterns/theming-shadcn-vue-complet.md] - Configuration complète shadcn
- [Source: architecture/implementation-patterns-consistency-rules/tailwindcss-patterns/variable-fonts-avec-strategie-anti-cls.md] - Fonts anti-CLS
- [Source: architecture/implementation-patterns-consistency-rules/tailwindcss-patterns/oklch-espace-colorimetrique-moderne.md] - OKLCH
- [Source: architecture/starter-template-evaluation/migration-reka-ui.md] - Migration Reka UI
- [Source: architecture/core-architectural-decisions/frontend-architecture.md] - Structure composants
- [Source: prd/13-functional-requirements-mvp.md#FR46] - Requirement FR46

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

### File List
