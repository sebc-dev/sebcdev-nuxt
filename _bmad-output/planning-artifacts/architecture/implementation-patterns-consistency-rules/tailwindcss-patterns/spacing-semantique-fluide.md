# Spacing Sémantique Fluide

## Tokens hiérarchiques avec clamp()

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

## Syntaxe clamp()

```css
clamp(minimum, preferred, maximum)
```

| Paramètre | Description | Exemple |
|-----------|-------------|---------|
| `minimum` | Valeur plancher (mobile) | `1rem` |
| `preferred` | Valeur fluide (viewport-relative) | `5vw` |
| `maximum` | Valeur plafond (desktop) | `6rem` |

## Usage dans les templates

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

## Combinaison avec Container Queries

```html
<section class="@container py-(--spacing-section)">
  <div class="grid gap-4 @md:grid-cols-2 @lg:grid-cols-3 @lg:gap-6">
    <div class="p-(--spacing-component) @sm:p-6">
      Contenu adaptatif au container
    </div>
  </div>
</section>
```

## Avantages du spacing fluide

| Approche | Media queries | Spacing fluide |
|----------|---------------|----------------|
| Code | `p-4 md:p-6 lg:p-8` | `p-(--spacing-component)` |
| Breakpoints | Sauts brusques | Transitions continues |
| Maintenance | Multiples classes | Un seul token |
| Consistance | Variable | Garantie |

---
