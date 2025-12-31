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

## Focus Tokens Accessibles

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
