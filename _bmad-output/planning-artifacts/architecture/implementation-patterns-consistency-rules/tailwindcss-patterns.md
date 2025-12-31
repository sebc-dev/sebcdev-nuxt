# TailwindCSS 4 Patterns

Patterns de configuration TailwindCSS 4 pour Nuxt 4 avec shadcn-vue et theming dynamique.

## Variable Fonts avec Stratégie Anti-CLS

### Configuration @font-face optimisée

La stratégie `font-display: optional` **élimine 100% du CLS** (Cumulative Layout Shift) :

```css
/* app/assets/css/main.css - AVANT @import "tailwindcss" */
@font-face {
  font-family: "Satoshi Variable";
  font-style: normal;
  font-weight: 100 900;
  font-display: optional;  /* ← Clé pour zéro CLS */
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
```

### Comparaison font-display

| Valeur | Comportement | CLS | Recommandation |
|--------|--------------|-----|----------------|
| `swap` | Flash de font fallback puis swap | **Élevé** | ⚠️ Acceptable si fallbacks métriques configurés |
| `block` | Texte invisible puis apparition | Moyen | ❌ Mauvaise UX |
| `fallback` | Swap si chargé < 100ms, sinon fallback | Faible | ⚠️ Compromis |
| `optional` | Utilise cache ou fallback direct | **Zéro** | ✅ **Recommandé SSG** |

**Recommandations contextuelles :**

| Contexte | font-display | Raison |
|----------|--------------|--------|
| **SSG blog** (ce projet) | `optional` | Zéro CLS, performance maximale |
| **App avec branding fort** | `swap` + fallbacks métriques | Web font visible, CLS minimisé par fallbacks |
| **Texte critique (navigation)** | `optional` | Ne jamais bloquer la navigation |
| **Titres décoratifs** | `swap` | Le swap est acceptable si non-critique |

**Note** : `@nuxt/fonts` utilise `swap` par défaut mais génère automatiquement des fallbacks avec métriques ajustées via fontaine/capsize, réduisant significativement le CLS. Pour zéro CLS garanti, utiliser `font-display: optional` dans les @font-face manuels.

### Fallback métrique pour CLS zéro (Capsize/Fontaine)

Pour éliminer complètement le CLS typographique avec `font-display: swap`, utiliser des fallbacks métriquement ajustés :

```css
/* assets/css/fonts.css */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter-var.woff2') format('woff2');
  font-weight: 100 900;
  font-display: swap;
}

/* Fallback métrique généré par Capsize/Fontaine */
@font-face {
  font-family: 'Inter Fallback';
  src: local('Arial');
  ascent-override: 90.49%;
  descent-override: 22.56%;
  line-gap-override: 0%;
  size-adjust: 107.64%;
}

@theme {
  --font-sans: 'Inter', 'Inter Fallback', ui-sans-serif, system-ui, sans-serif;
}
```

**Propriétés de fallback métrique :**

| Propriété | Description | Valeur typique |
|-----------|-------------|----------------|
| `ascent-override` | Hauteur au-dessus de la baseline | 85-95% |
| `descent-override` | Profondeur sous la baseline | 20-25% |
| `line-gap-override` | Espace entre lignes | 0% |
| `size-adjust` | Ajustement de taille globale | 100-110% |

**Génération automatique :**
- [Capsize](https://seek-oss.github.io/capsize/) - Calcul des métriques depuis un fichier font
- [@nuxt/fonts](https://nuxt.com/modules/fonts) - Génère automatiquement les fallbacks via fontaine

### Subsetting fonts avec Glyphhanger

Réduire **96% du poids** en conservant uniquement les caractères nécessaires :

```bash
# Installation
npm install -g glyphhanger
pip install fonttools brotli

# Subset Latin uniquement
glyphhanger --LATIN --subset=Inter.ttf --formats=woff2

# Résultat : Inter 765KB → ~15KB

# Subset avec caractères français (accents)
glyphhanger --LATIN --whitelist="àâäéèêëïîôùûüç" --subset=Inter.ttf --formats=woff2

# Subset depuis contenu réel du site
glyphhanger https://monsite.com --spider --subset=Inter.ttf --formats=woff2
```

**Presets de subsetting :**

| Preset | Caractères | Taille typique |
|--------|------------|----------------|
| `--LATIN` | A-Z, a-z, 0-9, ponctuation | ~15KB |
| `--LATIN` + accents FR | + àâäéèêëïîôùûüç | ~18KB |
| `--US_ASCII` | ASCII 7-bit uniquement | ~12KB |
| Subset custom | Crawl du site réel | Variable |

### Preload obligatoire

Le preload avec `crossorigin="anonymous"` est **obligatoire** même en self-hosting :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  app: {
    head: {
      link: [
        {
          rel: 'preload',
          href: '/fonts/Satoshi-Variable.woff2',
          as: 'font',
          type: 'font/woff2',
          crossorigin: 'anonymous'  // ← Obligatoire même en self-hosting
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
})
```

### Connexion à Tailwind

```css
@theme {
  --font-sans: "Satoshi Variable", ui-sans-serif, system-ui, sans-serif;
  --font-mono: "JetBrains Mono Variable", ui-monospace, Menlo, monospace;

  /* OpenType features pour Inter/Satoshi */
  --font-sans--font-feature-settings: "cv01", "cv02", "ss01";
}
```

---

## Configuration CSS-first

TailwindCSS v4 remplace `tailwind.config.js` par une configuration entièrement en CSS :

```css
/* app/assets/css/main.css */
@import "tailwindcss";

/* Scanner les fichiers MDC pour les classes Tailwind */
@source "../../../content/**/*";
```

## Design Tokens et Namespaces

### Tableau des namespaces

Chaque namespace dans `@theme` génère automatiquement les classes utilitaires correspondantes :

| Namespace | Classes générées | Exemple |
|-----------|-----------------|---------|
| `--color-*` | `bg-*`, `text-*`, `border-*`, `fill-*`, `stroke-*` | `--color-primary` → `bg-primary` |
| `--font-*` | `font-*` (famille) | `--font-display` → `font-display` |
| `--font-size-*` | `text-*` (taille) | `--font-size-xl` → `text-xl` |
| `--spacing-*` | `p-*`, `m-*`, `w-*`, `h-*`, `gap-*` | `--spacing-4` → `p-4`, `m-4` |
| `--radius-*` | `rounded-*` | `--radius-lg` → `rounded-lg` |
| `--shadow-*` | `shadow-*` | `--shadow-md` → `shadow-md` |
| `--breakpoint-*` | `sm:`, `md:`, `lg:`, `xl:` | `--breakpoint-3xl` → `3xl:` |
| `--ease-*` | `ease-*` (transitions) | `--ease-fluid` → `ease-fluid` |

### Extension vs Remplacement

```css
@theme {
  /* EXTENSION: ajoute aux valeurs par défaut */
  --color-brand: oklch(0.84 0.18 117.33);
  --breakpoint-3xl: 1920px;
}

@theme {
  /* REMPLACEMENT: supprime toutes les couleurs par défaut */
  --color-*: initial;
  --color-white: #fff;
  --color-primary: oklch(0.62 0.21 259);
}
```

## Spacing Sémantique Fluide

### Tokens hiérarchiques avec clamp()

Le système de spacing moderne combine des tokens sémantiques hiérarchiques (page → section → component) avec des valeurs fluides via `clamp()`. Cela permet un responsive **sans media queries**.

```css
@theme {
  /* Base multiplicative Tailwind 4 */
  --spacing: 0.25rem;

  /* Tokens page-level : marges externes */
  --spacing-page-x: clamp(1rem, 5vw, 6rem);
  --spacing-page-y: clamp(2rem, 6vw, 4rem);

  /* Tokens section-level : espacement entre sections */
  --spacing-section: clamp(3rem, 6vw, 6rem);
  --spacing-section-sm: clamp(2rem, 4vw, 4rem);

  /* Tokens component-level : padding interne */
  --spacing-component: clamp(1rem, 2vw, 1.5rem);

  /* Containers sémantiques */
  --container-prose: 65ch;    /* Largeur optimale lecture */
  --container-wide: 80rem;    /* Layout large */
}
```

### Syntaxe clamp()

```css
clamp(minimum, preferred, maximum)
```

| Paramètre | Description | Exemple |
|-----------|-------------|---------|
| `minimum` | Valeur plancher (mobile) | `1rem` |
| `preferred` | Valeur fluide (viewport-relative) | `5vw` |
| `maximum` | Valeur plafond (desktop) | `6rem` |

### Usage dans les templates

```html
<!-- Utilisation des tokens personnalisés -->
<main class="px-(--spacing-page-x) py-(--spacing-page-y)">
  <section class="py-(--spacing-section)">
    <article class="max-w-(--container-prose) p-(--spacing-component)">
      Contenu adaptatif sans media queries
    </article>
  </section>
</main>
```

### Combinaison avec Container Queries

```html
<section class="@container py-(--spacing-section)">
  <div class="grid gap-4 @md:grid-cols-2 @lg:grid-cols-3 @lg:gap-6">
    <div class="p-(--spacing-component) @sm:p-6">
      Contenu adaptatif au container
    </div>
  </div>
</section>
```

### Avantages du spacing fluide

| Approche | Media queries | Spacing fluide |
|----------|---------------|----------------|
| Code | `p-4 md:p-6 lg:p-8` | `p-(--spacing-component)` |
| Breakpoints | Sauts brusques | Transitions continues |
| Maintenance | Multiples classes | Un seul token |
| Consistance | Variable | Garantie |

---

## Typographie Fluide

### Tokens typographiques avec clamp()

L'échelle typographique fluide utilise `clamp()` avec des propriétés CSS associées (line-height, font-weight, letter-spacing) :

```css
@theme {
  /* Corps de texte fluide */
  --text-fluid-body: clamp(1rem, 0.913rem + 0.4348vw, 1.2rem);
  --text-fluid-body--line-height: 1.6;

  /* Titres fluides avec propriétés associées */
  --text-fluid-h1: clamp(2.488rem, 1.943rem + 2.726vw, 3.5rem);
  --text-fluid-h1--line-height: 1.15;
  --text-fluid-h1--font-weight: 700;
  --text-fluid-h1--letter-spacing: -0.02em;

  --text-fluid-h2: clamp(1.953rem, 1.586rem + 1.835vw, 2.625rem);
  --text-fluid-h2--line-height: 1.2;
  --text-fluid-h2--font-weight: 600;
  --text-fluid-h2--letter-spacing: -0.015em;

  --text-fluid-h3: clamp(1.563rem, 1.318rem + 1.222vw, 2rem);
  --text-fluid-h3--line-height: 1.3;
  --text-fluid-h3--font-weight: 600;

  /* Display pour hero sections */
  --text-fluid-display: clamp(3rem, 2rem + 5vw, 6rem);
  --text-fluid-display--line-height: 1.05;
  --text-fluid-display--font-weight: 800;
  --text-fluid-display--letter-spacing: -0.03em;

  /* Small/caption fluide */
  --text-fluid-small: clamp(0.833rem, 0.769rem + 0.321vw, 0.9rem);
  --text-fluid-small--line-height: 1.5;
}
```

### Usage dans les templates

```html
<!-- Utilisation directe du token -->
<h1 class="text-(--text-fluid-h1) leading-(--text-fluid-h1--line-height) font-(--text-fluid-h1--font-weight) tracking-(--text-fluid-h1--letter-spacing)">
  Titre principal
</h1>

<!-- Avec classes custom dans @layer -->
<h1 class="text-fluid-h1">Titre principal</h1>
```

### Classes utilitaires recommandées

Pour simplifier l'usage, définir des classes dans `@layer utilities` :

```css
@layer utilities {
  .text-fluid-body {
    font-size: var(--text-fluid-body);
    line-height: var(--text-fluid-body--line-height);
  }

  .text-fluid-h1 {
    font-size: var(--text-fluid-h1);
    line-height: var(--text-fluid-h1--line-height);
    font-weight: var(--text-fluid-h1--font-weight);
    letter-spacing: var(--text-fluid-h1--letter-spacing);
  }

  .text-fluid-h2 {
    font-size: var(--text-fluid-h2);
    line-height: var(--text-fluid-h2--line-height);
    font-weight: var(--text-fluid-h2--font-weight);
    letter-spacing: var(--text-fluid-h2--letter-spacing);
  }

  .text-fluid-display {
    font-size: var(--text-fluid-display);
    line-height: var(--text-fluid-display--line-height);
    font-weight: var(--text-fluid-display--font-weight);
    letter-spacing: var(--text-fluid-display--letter-spacing);
  }
}
```

### Calcul des valeurs clamp()

Formule pour calculer la valeur préférée `vw` :

```
preferred = (maxSize - minSize) / (maxViewport - minViewport) × 100vw
```

| Paramètre | Valeur typique |
|-----------|----------------|
| `minViewport` | 320px (mobile) |
| `maxViewport` | 1440px (desktop) |
| `minSize` | Taille minimum souhaitée |
| `maxSize` | Taille maximum souhaitée |

**Outil recommandé** : [Utopia Fluid Type Scale](https://utopia.fyi/type/calculator/) pour générer les valeurs automatiquement.

---

## Différence @theme vs @theme inline vs :root

### Tableau comparatif

| Déclaration | Génère classes Tailwind | Utilisable via `var()` | Modifiable par JS |
|-------------|-------------------------|------------------------|-------------------|
| `@theme {}` | ✅ Oui | ❌ Non | ❌ Non |
| `@theme inline {}` | ✅ Oui | ✅ Oui | ✅ Oui |
| `:root {}` | ❌ Non | ✅ Oui | ✅ Oui |

### Quand utiliser quoi

| Cas d'usage | Déclaration recommandée |
|-------------|------------------------|
| Valeurs statiques (fonts, breakpoints) | `@theme {}` |
| Theming dynamique (dark mode, marques) | `:root` + `@theme inline` |
| Variables CSS pures (non-Tailwind) | `:root {}` |

### Pattern theming dynamique

```css
/* 1. Définir les variables dans :root et .dark */
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
}

/* 2. Connecter à Tailwind via @theme inline */
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
}
```

**Résultat** : Les classes `bg-background`, `text-foreground` changent automatiquement quand la classe `.dark` est ajoutée au `<html>`.

## Theming shadcn-vue Complet

### Configuration obligatoire

Le variant `dark` doit être explicitement défini pour shadcn-vue avec TailwindCSS 4 :

```css
/* app/assets/css/main.css */
@import "tailwindcss";
@import "tw-animate-css";

/* OBLIGATOIRE: définit le variant dark pour shadcn-vue */
@custom-variant dark (&:is(.dark *));
```

### Choix de spécificité : `:is()` vs `:where()`

| Syntaxe | Spécificité | Quand utiliser |
|---------|-------------|----------------|
| `&:is(.dark *)` | Normale (0,1,0) | Défaut shadcn-vue, robuste |
| `&:where(.dark *)` | Zéro (0,0,0) | Plus facile à overrider |
| `&:where(.dark, .dark *)` | Zéro | Inclut l'élément `.dark` lui-même |

```css
/* Alternative basse spécificité (plus flexible pour overrides) */
@custom-variant dark (&:where(.dark, .dark *));

/* Combinaison préférence système + toggle manuel */
@custom-variant dark {
  &:where(.dark, .dark *) { @slot; }
  @media (prefers-color-scheme: dark) {
    &:where(:not(.light *)) { @slot; }
  }
}
```

**Recommandation** : Utiliser `:is()` (défaut shadcn-vue) sauf besoin spécifique d'overrides faciles.

### Palette shadcn-vue complète

```css
:root {
  /* Backgrounds */
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);

  /* Composants */
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.145 0 0);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.145 0 0);

  /* Primaire/Secondaire */
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.205 0 0);

  /* États */
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --destructive-foreground: oklch(0.985 0 0);

  /* Bordures et inputs */
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --ring: oklch(0.708 0 0);

  /* Radius global */
  --radius: 0.5rem;
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.145 0 0);
  --card-foreground: oklch(0.985 0 0);
  --popover: oklch(0.145 0 0);
  --popover-foreground: oklch(0.985 0 0);
  --primary: oklch(0.985 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --secondary: oklch(0.269 0 0);
  --secondary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --accent: oklch(0.269 0 0);
  --accent-foreground: oklch(0.985 0 0);
  --destructive: oklch(0.704 0.191 22.216);
  --destructive-foreground: oklch(0.985 0 0);
  --border: oklch(0.269 0 0);
  --input: oklch(0.269 0 0);
  --ring: oklch(0.439 0 0);
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-popover: var(--popover);
  --color-popover-foreground: var(--popover-foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-destructive-foreground: var(--destructive-foreground);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);
  --radius-lg: var(--radius);
}
```

### Focus Tokens Accessibles

Configuration complète des tokens de focus pour la technique double-ring :

```css
:root {
  /* Focus ring principal */
  --focus-ring: oklch(0.585 0.233 277.117);

  /* Double-ring : couleurs intérieure et extérieure */
  --focus-inner: oklch(1 0 0);        /* Blanc */
  --focus-outer: oklch(0.205 0 0);    /* Quasi-noir */

  /* Scroll padding pour WCAG 2.4.11 */
  --header-height: 80px;
}

.dark {
  --focus-ring: oklch(0.673 0.182 276.935);
  --focus-inner: oklch(0.145 0 0);    /* Quasi-noir */
  --focus-outer: oklch(0.985 0 0);    /* Quasi-blanc */
}

@theme inline {
  --color-focus-ring: var(--focus-ring);
  --color-focus-inner: var(--focus-inner);
  --color-focus-outer: var(--focus-outer);
}

@layer base {
  html {
    scroll-padding-top: var(--header-height);
  }

  /* Focus global double-ring */
  *:focus-visible {
    outline: 2px solid var(--color-focus-inner);
    outline-offset: 0;
    box-shadow: 0 0 0 4px var(--color-focus-outer);
  }

  /* Respect prefers-reduced-motion */
  @media (prefers-reduced-motion: reduce) {
    *, ::before, ::after {
      animation-duration: 0.01ms !important;
      transition-duration: 0.01ms !important;
    }
  }
}
```

**Usage avec classes Tailwind** (pour override local) :

```html
<button class="
  focus-visible:outline-2
  focus-visible:outline-focus-inner
  focus-visible:outline-offset-0
  focus-visible:ring-4
  focus-visible:ring-focus-outer
">
  Bouton avec focus personnalisé
</button>
```

## Utilitaire cn() pour Classes Conditionnelles

### Installation

```bash
pnpm add clsx tailwind-merge
```

### Implémentation

L'utilitaire `cn()` combine `clsx` (classes conditionnelles) et `tailwind-merge` (résolution de conflits Tailwind) :

```typescript
// app/lib/utils.ts
import type { ClassValue } from 'clsx'
import { clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

### Pourquoi cn() est essentiel

| Problème | Sans cn() | Avec cn() |
|----------|-----------|-----------|
| Classes conflictuelles | `p-4 p-2` → applique les deux | `p-4 p-2` → `p-2` (dernière gagne) |
| Classes conditionnelles | Template complexe | Syntaxe claire |
| Props de composants | Écrasement imprévisible | Merge intelligent |

### Usage dans les composants

```vue
<script setup lang="ts">
import { cn } from '@/lib/utils'

const props = defineProps<{
  class?: string
  variant?: 'default' | 'destructive'
}>()
</script>

<template>
  <button
    :class="cn(
      'inline-flex items-center justify-center rounded-md',
      variant === 'destructive' && 'bg-destructive text-destructive-foreground',
      variant === 'default' && 'bg-primary text-primary-foreground',
      props.class  // ← Permet l'override par le parent
    )"
  >
    <slot />
  </button>
</template>
```

### Exemples de résolution de conflits

```typescript
// Conflits de padding
cn('p-4', 'p-2')                    // → 'p-2'
cn('px-4 py-2', 'p-3')              // → 'p-3'

// Conflits de couleurs
cn('text-red-500', 'text-blue-500') // → 'text-blue-500'

// Classes conditionnelles
cn('base', isActive && 'active')    // → 'base active' ou 'base'
cn('base', { active: isActive })    // → même résultat

// Arrays
cn(['flex', 'items-center'], gap && 'gap-4')
```

### Pattern shadcn-vue avec CVA

```typescript
// app/components/ui/button/index.ts
import { cva, type VariantProps } from 'class-variance-authority'

export const buttonVariants = cva(
  'inline-flex items-center justify-center gap-2 rounded-md text-sm font-medium',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground',
        outline: 'border border-input bg-background hover:bg-accent',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 px-3',
        lg: 'h-11 px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: { variant: 'default', size: 'default' },
  }
)

export type ButtonVariants = VariantProps<typeof buttonVariants>
```

```vue
<!-- app/components/ui/button/Button.vue -->
<script setup lang="ts">
import { cn } from '@/lib/utils'
import { buttonVariants, type ButtonVariants } from '.'

interface Props extends ButtonVariants {
  class?: string
}

const props = withDefaults(defineProps<Props>(), {
  variant: 'default',
  size: 'default',
})
</script>

<template>
  <button :class="cn(buttonVariants({ variant, size }), props.class)">
    <slot />
  </button>
</template>
```

---

## OKLCH : Espace Colorimétrique Moderne

### Syntaxe

```css
oklch(L C H / alpha)
```

| Paramètre | Plage | Description |
|-----------|-------|-------------|
| **L** (Lightness) | 0-1 | Luminosité perceptuelle (0 = noir, 1 = blanc) |
| **C** (Chroma) | 0-0.4 | Intensité chromatique (saturation) |
| **H** (Hue) | 0-360° | Teinte sur le cercle chromatique |
| **alpha** | 0-1 | Opacité (optionnel) |

### Avantages par rapport à HSL/RGB

| Aspect | HSL/RGB | OKLCH |
|--------|---------|-------|
| **Uniformité perceptuelle** | ❌ Jaune et bleu à 50% paraissent très différents | ✅ Changement de L produit résultat visuel cohérent |
| **Gamut** | sRGB uniquement | Display P3, Rec2020 |
| **Gradients** | Zones grises entre couleurs saturées | Transitions fluides |
| **Prédictibilité** | Ajustements imprévisibles | Résultats attendus |

### Exemples de palettes

```css
/* Nuances de bleu cohérentes */
--blue-100: oklch(0.95 0.05 250);
--blue-300: oklch(0.75 0.12 250);
--blue-500: oklch(0.55 0.18 250);
--blue-700: oklch(0.35 0.15 250);
--blue-900: oklch(0.20 0.10 250);

/* Couleurs sémantiques */
--success: oklch(0.65 0.15 145);   /* Vert */
--warning: oklch(0.80 0.15 85);    /* Orange */
--error: oklch(0.55 0.20 25);      /* Rouge */
--info: oklch(0.60 0.15 250);      /* Bleu */
```

### Support navigateur

**92.86%** global (Chrome 111+, Safari 15.4+, Firefox 113+).

Lightning CSS transpile automatiquement vers RGB avec fallbacks si nécessaire.

## Aspect-ratio CSS Moderne

En **décembre 2025**, `aspect-ratio` CSS bénéficie d'un support navigateur de **95.33%**. Aucun fallback padding-bottom n'est nécessaire.

### Classes natives et ratios custom

```css
/* app/assets/css/main.css */
@import "tailwindcss";

@theme {
  /* Ratios personnalisés pour le blog */
  --aspect-retro: 4 / 3;
  --aspect-cinema: 21 / 9;
  --aspect-instagram-feed: 4 / 5;
  --aspect-story: 9 / 16;
  --aspect-og-image: 1200 / 630;
}
```

### Usage dans les templates

```html
<!-- Classes natives TailwindCSS 4 -->
<img class="aspect-video w-full object-cover" src="/cover.jpg" />
<div class="aspect-square bg-muted">Carré</div>
<iframe class="aspect-[21/9] w-full" src="https://youtube.com/embed/..."></iframe>

<!-- Ratios custom depuis @theme -->
<img class="aspect-retro w-full" src="/photo.jpg" />
<div class="aspect-story max-w-sm mx-auto">Story format</div>
<div class="aspect-og-image bg-muted">Preview OG Image</div>
```

### Classes aspect-ratio disponibles

| Classe | Ratio | Cas d'usage |
|--------|-------|-------------|
| `aspect-auto` | auto | Ratio intrinsèque de l'image |
| `aspect-square` | 1 / 1 | Avatars, thumbnails carrés |
| `aspect-video` | 16 / 9 | YouTube, Vimeo, vidéos standard |
| `aspect-[4/3]` | 4 / 3 | Photos classiques |
| `aspect-[21/9]` | 21 / 9 | Films cinématographiques |
| `aspect-[9/16]` | 9 / 16 | Stories, Reels, TikTok |

### Pattern avec object-fit

```html
<!-- Image couvrant tout l'espace avec crop -->
<div class="aspect-video">
  <img src="/hero.jpg" class="w-full h-full object-cover" />
</div>

<!-- Image contenue sans crop -->
<div class="aspect-video bg-muted">
  <img src="/diagram.png" class="w-full h-full object-contain" />
</div>
```

---

## Container Queries Natifs

TailwindCSS 4 intègre nativement les container queries (plus besoin du plugin `@tailwindcss/container-queries`).

### Syntaxe de base

```html
<!-- Définir un container -->
<div class="@container">
  <!-- Réagir à la taille du container parent -->
  <div class="flex flex-col @md:flex-row @lg:grid @lg:grid-cols-3">
    ...
  </div>
</div>
```

### Breakpoints disponibles

| Variant | Largeur min | Variant | Largeur min |
|---------|-------------|---------|-------------|
| `@3xs:` | 256px | `@xl:` | 576px |
| `@xs:` | 320px | `@2xl:` | 672px |
| `@sm:` | 384px | `@3xl:` | 768px |
| `@md:` | 448px | `@4xl:` | 896px |
| `@lg:` | 512px | `@5xl:` - `@7xl:` | 1024px - 1280px |

### Containers nommés

```html
<div class="@container/sidebar">
  <div class="@sm/sidebar:flex-row">
    <!-- Réagit à la taille du container 'sidebar' spécifiquement -->
  </div>
</div>
```

### Queries descendantes (max-width)

```html
<div class="@container">
  <div class="grid @max-md:grid-cols-1 @md:grid-cols-2">
    <!-- 1 colonne si container < 448px, 2 colonnes sinon -->
  </div>
</div>
```

### Cas d'usage recommandés

| Utiliser container queries | Utiliser media queries |
|---------------------------|------------------------|
| Composants réutilisables (cartes, widgets) | Layouts page-level (nav, footer) |
| Design systems "self-aware" | Changements basés sur device |
| Zones de tailles variables | Orientation viewport |

### Support navigateur

**93.92%** global (décembre 2025).

## Vue Transitions avec Tailwind

### Composant `<Transition>` avec classes Tailwind

Le composant `<Transition>` de Vue 3 s'intègre parfaitement avec les classes Tailwind via les props de classes personnalisées. Cette approche est idéale pour le SSG car les transitions CSS fonctionnent immédiatement sans hydratation :

```vue
<script setup lang="ts">
const isOpen = ref(false)
</script>

<template>
  <Transition
    enter-active-class="transition-all duration-300 ease-out"
    enter-from-class="opacity-0 scale-95 translate-y-4"
    enter-to-class="opacity-100 scale-100 translate-y-0"
    leave-active-class="transition-all duration-200 ease-in"
    leave-from-class="opacity-100 scale-100 translate-y-0"
    leave-to-class="opacity-0 scale-95 translate-y-4"
  >
    <div v-if="isOpen" class="modal-content">
      Contenu animé
    </div>
  </Transition>
</template>
```

### TransitionGroup pour listes animées

Pour `<TransitionGroup>`, ajouter `move-class` pour animer le repositionnement des éléments :

```vue
<TransitionGroup
  tag="ul"
  enter-active-class="transition-all duration-300"
  enter-from-class="opacity-0 translate-x-8"
  leave-active-class="transition-all duration-200"
  leave-to-class="opacity-0 translate-x-8"
  move-class="transition-all duration-300"
>
  <li v-for="item in items" :key="item.id">
    {{ item.name }}
  </li>
</TransitionGroup>
```

### Composable réutilisable useTransitions

```typescript
// composables/useTransitions.ts
export function useScaleFadeTransition(duration = 200) {
  return {
    enterActiveClass: `transition-all duration-${duration} ease-out`,
    enterFromClass: 'opacity-0 scale-95',
    enterToClass: 'opacity-100 scale-100',
    leaveActiveClass: `transition-all duration-${duration} ease-in`,
    leaveFromClass: 'opacity-100 scale-100',
    leaveToClass: 'opacity-0 scale-95'
  }
}

export function useSlideTransition(direction: 'left' | 'right' | 'up' | 'down' = 'right', duration = 300) {
  const translations = {
    left: 'translate-x-8',
    right: '-translate-x-8',
    up: 'translate-y-8',
    down: '-translate-y-8'
  }
  return {
    enterActiveClass: `transition-all duration-${duration} ease-out`,
    enterFromClass: `opacity-0 ${translations[direction]}`,
    enterToClass: 'opacity-100 translate-x-0 translate-y-0',
    leaveActiveClass: `transition-all duration-${duration} ease-in`,
    leaveFromClass: 'opacity-100 translate-x-0 translate-y-0',
    leaveToClass: `opacity-0 ${translations[direction]}`
  }
}
```

**Usage :**

```vue
<script setup>
import { useScaleFadeTransition } from '~/composables/useTransitions'
const transitionProps = useScaleFadeTransition(300)
</script>

<template>
  <Transition v-bind="transitionProps">
    <div v-if="show">Contenu</div>
  </Transition>
</template>
```

---

## Anti-patterns Animations

### ❌ Animer des propriétés de layout

```css
/* ❌ Déclenche des recalculs coûteux (reflow) */
.element {
  transition: width 300ms, height 300ms, margin 300ms;
}

/* ✅ Utiliser transform et opacity (accélérés GPU) */
.element {
  transition: transform 300ms, opacity 300ms;
}
```

| Propriété | Performance | Alternative |
|-----------|-------------|-------------|
| `width`, `height` | ❌ Reflow | `transform: scale()` |
| `margin`, `padding` | ❌ Reflow | `transform: translate()` |
| `top`, `left`, `right`, `bottom` | ❌ Reflow | `transform: translate()` |
| `transform`, `opacity` | ✅ Composite only | — |

### ❌ Conflits Vue Transition + CSS animations

```vue
<!-- ❌ Les deux systèmes entrent en conflit -->
<Transition enter-active-class="transition-all duration-300">
  <div class="animate-in fade-in">Conflit</div>
</Transition>

<!-- ✅ Utiliser un seul système -->
<Transition enter-active-class="transition-all duration-300 ease-out">
  <div>Un seul système</div>
</Transition>
```

### ❌ Oublier tw-animate-css

Les animations shadcn-vue (`animate-in`, `fade-in-0`, etc.) **échouent silencieusement** sans l'import :

```css
/* ❌ Animations shadcn-vue ne fonctionnent pas */
@import "tailwindcss";

/* ✅ Import obligatoire */
@import "tailwindcss";
@import "tw-animate-css";
```

### ❌ Memory leaks animations JavaScript

```typescript
// ❌ Animation non nettoyée
onMounted(() => {
  gsap.to('.element', { x: 100, duration: 1 })
})

// ✅ Cleanup dans onUnmounted
const animation = ref<gsap.core.Tween | null>(null)

onMounted(() => {
  animation.value = gsap.to('.element', { x: 100, duration: 1 })
})

onUnmounted(() => {
  animation.value?.kill()
})
```

### Pattern anti-flash SSG

Les animations peuvent déclencher un flash visuel au chargement SSG si elles s'exécutent avant hydratation. Différer avec `onMounted` :

```vue
<script setup>
const mounted = ref(false)
onMounted(() => mounted.value = true)
</script>

<template>
  <!-- Animation différée après hydratation -->
  <div :class="{ 'animate-in fade-in duration-500': mounted }">
    Contenu sans flash
  </div>

  <!-- Alternative : motion-safe pour respecter aussi les préférences -->
  <div :class="mounted && 'motion-safe:animate-in motion-safe:fade-in'">
    Contenu accessible
  </div>
</template>
```

**Pourquoi c'est nécessaire** : En SSG, le HTML statique contient les classes d'animation. Sans `onMounted`, l'animation se déclenche au parsing CSS, avant que Vue ne soit hydraté, causant un flash visuel.

---

## CSS Global prefers-reduced-motion (Filet de Sécurité)

En complément des variants `motion-safe:` et `motion-reduce:` de Tailwind, ce CSS global agit comme **filet de sécurité** pour les animations non gérées individuellement.

### Configuration dans main.css

```css
/* app/assets/css/main.css */
@import "tailwindcss";

@layer base {
  /* Filet de sécurité global pour reduced motion */
  @media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
      scroll-behavior: auto !important;
    }
  }
}
```

### Pourquoi ce pattern est nécessaire

| Sans filet de sécurité | Avec filet de sécurité |
|------------------------|------------------------|
| Chaque animation doit être gérée individuellement | Fallback automatique pour animations oubliées |
| CSS tiers peut ignorer les préférences | Tout le CSS est couvert |
| Risque d'accessibilité si un dev oublie | Accessibilité garantie par défaut |

### Combinaison avec variants Tailwind

```html
<!-- Approach recommandée : opt-in explicite -->
<button class="
  bg-primary
  motion-safe:transition
  motion-safe:duration-200
  motion-safe:hover:scale-105
">
  Bouton animé si autorisé
</button>

<!-- Le filet de sécurité capture les oublis -->
<div class="animate-spin">
  <!-- Si motion-reduce: pas demandé explicitement,
       le filet de sécurité l'arrête quand même -->
</div>
```

### Composable VueUse pour animations JavaScript

```typescript
// composables/useReducedMotion.ts
import { useMediaQuery } from '@vueuse/core'

export function useReducedMotion() {
  const prefersReducedMotion = useMediaQuery('(prefers-reduced-motion: reduce)')

  const transitionDuration = computed(() =>
    prefersReducedMotion.value ? 0 : 300
  )

  const shouldAnimate = computed(() => !prefersReducedMotion.value)

  return {
    prefersReducedMotion,
    transitionDuration,
    shouldAnimate
  }
}
```

```vue
<script setup>
import { useReducedMotion } from '~/composables/useReducedMotion'

const { shouldAnimate, transitionDuration } = useReducedMotion()
</script>

<template>
  <Transition :duration="transitionDuration">
    <div v-if="show && shouldAnimate">
      Contenu animé conditionnellement
    </div>
    <div v-else-if="show">
      Contenu sans animation
    </div>
  </Transition>
</template>
```

---

## Breaking Changes v3 → v4

### Renommages de classes

| v3 | v4 |
|----|----|
| `shadow-sm` | `shadow-xs` |
| `shadow` | `shadow-sm` |
| `bg-gradient-to-*` | `bg-linear-to-*` |
| `rounded-sm` | `rounded-xs` |
| `ring` | `ring-1` (défaut passe de 3px à 1px) |
| `outline-none` | `outline-hidden` (pour WHCM) |

### Focus et Accessibilité - Changements critiques

#### `ring` passe de 3px à 1px

```css
/* v3 : ring = 3px par défaut */
.element { @apply ring; }  /* → box-shadow: 0 0 0 3px */

/* v4 : ring = 1px par défaut */
.element { @apply ring; }  /* → box-shadow: 0 0 0 1px */

/* Migration : utiliser ring-3 pour conserver le comportement v3 */
.element { @apply ring-3; }
```

#### `outline-none` vs `outline-hidden`

| Classe | CSS généré | Windows High Contrast |
|--------|------------|----------------------|
| `outline-none` | `outline-style: none` | ❌ Focus invisible |
| `outline-hidden` | `outline: 2px solid transparent` | ✅ Focus visible |

```html
<!-- ❌ v3 pattern - cassé en WHCM -->
<button class="outline-none focus:ring-2">...</button>

<!-- ✅ v4 pattern - accessible -->
<button class="outline-hidden focus-visible:ring-2">...</button>
```

**Règle** : Toujours utiliser `outline-hidden` au lieu de `outline-none` pour maintenir l'accessibilité en mode High Contrast.

#### `ring` vs `outline` pour l'accessibilité

| Propriété | `ring` | `outline` |
|-----------|--------|-----------|
| CSS sous-jacent | `box-shadow` | `outline` natif |
| Windows High Contrast | ❌ Non visible | ✅ Visible |
| Usage recommandé | Effets décoratifs | **Focus indicators** |

**Pattern recommandé** : Combiner `outline` (pour WHCM) et `ring` (pour style) via la technique double-ring.

### Syntaxe variables arbitraires

```css
/* v3 (obsolète) */
.element {
  @apply bg-[--my-color];
}

/* v4 (correct) */
.element {
  @apply bg-(--my-color);
}
```

### Dark mode par défaut

| v3 | v4 |
|----|----|
| Basé sur classe `.dark` | Basé sur `prefers-color-scheme` |
| Config: `darkMode: 'class'` | Config: `@custom-variant dark (...)` |

Pour utiliser le mode classe avec shadcn-vue :

```css
/* Obligatoire pour shadcn-vue */
@custom-variant dark (&:is(.dark *));
```

### Outil de migration

```bash
# Migration automatique v3 → v4
npx @tailwindcss/upgrade@next
```

## Anti-patterns à éviter

### ❌ Utiliser @nuxtjs/tailwindcss avec TW4

```typescript
// ❌ NON compatible TailwindCSS 4
modules: ['@nuxtjs/tailwindcss']

// ✅ Utiliser le plugin Vite direct
import tailwindcss from '@tailwindcss/vite'
export default defineNuxtConfig({
  vite: { plugins: [tailwindcss()] }
})
```

### ❌ Mélanger @theme et :root pour theming

```css
/* ❌ Les valeurs @theme ne sont pas dynamiques */
@theme {
  --color-primary: oklch(0.62 0.21 259);
}
.dark {
  /* N'aura aucun effet sur --color-primary */
}

/* ✅ Utiliser :root + @theme inline */
:root { --primary: oklch(0.62 0.21 259); }
.dark { --primary: oklch(0.85 0.15 259); }
@theme inline { --color-primary: var(--primary); }
```

### ❌ Oublier @custom-variant dark

```css
/* ❌ Le dark mode shadcn-vue ne fonctionnera pas */
@import "tailwindcss";

/* ✅ Définir explicitement le variant */
@import "tailwindcss";
@custom-variant dark (&:is(.dark *));
```

### ❌ Concaténation dynamique de classes

Le JIT de TailwindCSS v4 ne peut pas détecter les classes construites dynamiquement par concaténation de strings. Ces classes seront **absentes du CSS généré**.

```vue
<!-- ❌ NON détectable par le JIT - classes manquantes -->
<div :class="`bg-${color}-500`">
<div :class="`text-${size}`">
<div :class="'p-' + padding">

<!-- ✅ Détectable - objets conditionnels -->
<div :class="{ 'bg-red-500': color === 'red', 'bg-blue-500': color === 'blue' }">

<!-- ✅ Détectable - mapping explicite -->
<div :class="colorClasses[color]">

<!-- ✅ Détectable - computed avec classes complètes -->
<div :class="buttonSizeClass">
```

**Pattern recommandé : mappings explicites**

```typescript
// ✅ Le JIT détecte toutes les classes listées
const colorClasses: Record<string, string> = {
  red: 'bg-red-500 hover:bg-red-600',
  blue: 'bg-blue-500 hover:bg-blue-600',
  green: 'bg-green-500 hover:bg-green-600',
}

const sizeClasses: Record<string, string> = {
  sm: 'text-sm px-2 py-1',
  md: 'text-base px-4 py-2',
  lg: 'text-lg px-6 py-3',
}
```

**Alternative : `@source inline()` pour cas dynamiques**

Si vous avez besoin de classes dynamiques (ex: données CMS), utilisez la directive safelist :

```css
/* app/assets/css/main.css */
@import "tailwindcss";

/* Safelist explicite pour classes dynamiques */
@source inline("{sm:,md:,lg:,}grid-cols-{1,2,3,4}");
@source inline("{hover:,}text-{red,blue,green}-{500,600,700}");
@source inline("bg-{red,blue,green}-{100,200,500}");
```

| Situation | Solution |
|-----------|----------|
| Props de composant (variant, size) | Mapping explicite |
| Données utilisateur/CMS | `@source inline()` |
| Classes calculées à runtime | Mapping + computed |

## Checklist Configuration

```markdown
## Fichier main.css
- [ ] `@import "tailwindcss";`
- [ ] `@source` pour fichiers MDC si applicable
- [ ] `@custom-variant dark (&:is(.dark *));` pour shadcn-vue
- [ ] Variables `:root` et `.dark` pour theming
- [ ] `@theme inline` pour connecter variables à Tailwind

## nuxt.config.ts
- [ ] `@tailwindcss/vite` dans vite.plugins
- [ ] NE PAS utiliser @nuxtjs/tailwindcss
- [ ] `css: ['~/assets/css/main.css']`

## Migrations v3 → v4
- [ ] Exécuter `npx @tailwindcss/upgrade@next`
- [ ] Vérifier renommages (shadow-sm → shadow-xs, etc.)
- [ ] Mettre à jour syntaxe variables arbitraires
```
