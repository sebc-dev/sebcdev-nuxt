# Theming shadcn-vue Complet

## Configuration obligatoire

Le variant `dark` doit être explicitement défini pour shadcn-vue avec TailwindCSS 4 :

```css
/* app/assets/css/main.css */
@import "tailwindcss";
@import "tw-animate-css";

/* OBLIGATOIRE: définit le variant dark pour shadcn-vue */
@custom-variant dark (&:is(.dark *));
```

## Choix de spécificité : `:is()` vs `:where()`

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

## Palette shadcn-vue complète

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

  /* Radius global - UX Spec aligned */
  --radius: 0.375rem;  /* 6px */
  --radius-sm: 0.25rem;  /* 4px */
  --radius-md: 0.5rem;   /* 8px */
  --radius-lg: 0.75rem;  /* 12px */

  /* Layout Constants - UX Spec aligned */
  --content-max-width: 720px;
  --toc-width: 240px;
  --container-max: 1200px;
  --layout-gap: 48px;
  --header-height: 64px;
}

/* ============================================
   sebc.dev Dark Mode Theme (Dark-Only)
   Valeurs alignées avec UX Specification
   ============================================ */
.dark {
  /* Backgrounds - UX Spec aligned */
  --background: oklch(0.067 0 0);           /* #0A0A0B - Fond principal */
  --background-secondary: oklch(0.078 0 0); /* #141415 - Cartes, panneaux */
  --background-tertiary: oklch(0.118 0 0);  /* #1E1E20 - Hover states */

  /* Foreground - UX Spec aligned */
  --foreground: oklch(0.98 0 0);            /* #FAFAFA - Texte principal */
  --foreground-muted: oklch(0.67 0.01 286); /* #A1A1AA - Texte secondaire */

  /* Composants - dérivés des backgrounds */
  --card: oklch(0.078 0 0);                 /* background-secondary */
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

  /* Accent - Teal UX Spec */
  --accent: oklch(0.696 0.143 175.8);       /* #14B8A6 - Teal */
  --accent-foreground: oklch(0.067 0 0);

  /* États fonctionnels */
  --destructive: oklch(0.628 0.258 29.234); /* #EF4444 - Erreurs */
  --destructive-foreground: oklch(0.98 0 0);
  --success: oklch(0.696 0.17 142.5);       /* #22C55E - Confirmations */
  --warning: oklch(0.795 0.184 86.047);     /* #EAB308 - Avertissements */

  /* Bordures et inputs */
  --border: oklch(0.185 0 0);               /* #27272A */
  --input: oklch(0.185 0 0);
  --ring: oklch(0.696 0.143 175.8);         /* Accent teal */

  /* ============================================
     Couleurs des Piliers sebc.dev
     ============================================ */
  --pillar-ia: oklch(0.586 0.232 292);      /* #8B5CF6 - Violet */
  --pillar-engineering: oklch(0.588 0.213 254); /* #3B82F6 - Bleu */
  --pillar-ux: oklch(0.656 0.241 354);      /* #EC4899 - Rose */
}

@theme inline {
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

  /* Radius */
  --radius-sm: var(--radius-sm);
  --radius-DEFAULT: var(--radius);
  --radius-md: var(--radius-md);
  --radius-lg: var(--radius-lg);

  /* Layout */
  --content-max-width: var(--content-max-width);
  --toc-width: var(--toc-width);
  --container-max: var(--container-max);
}
```

## Focus Tokens Accessibles

Configuration complète des tokens de focus pour la technique double-ring :

```css
:root {
  /* Focus ring principal */
  --focus-ring: oklch(0.585 0.233 277.117);

  /* Double-ring : couleurs intérieure et extérieure */
  --focus-inner: oklch(1 0 0);        /* Blanc */
  --focus-outer: oklch(0.205 0 0);    /* Quasi-noir */

  /* Scroll padding pour WCAG 2.4.11 - UX Spec aligned */
  --header-height: 64px;
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
