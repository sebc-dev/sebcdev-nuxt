# Design Tokens et Namespaces

## Tableau des namespaces

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

## Extension vs Remplacement

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
