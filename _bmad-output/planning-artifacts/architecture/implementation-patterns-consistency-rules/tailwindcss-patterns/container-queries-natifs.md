# Container Queries Natifs

TailwindCSS 4 intègre nativement les container queries (plus besoin du plugin `@tailwindcss/container-queries`).

## Syntaxe de base

```html
<!-- Définir un container -->
<div class="@container">
  <!-- Réagir à la taille du container parent -->
  <div class="flex flex-col @md:flex-row @lg:grid @lg:grid-cols-3">
    ...
  </div>
</div>
```

## Breakpoints disponibles

| Variant | Largeur min | Variant | Largeur min |
|---------|-------------|---------|-------------|
| `@3xs:` | 256px | `@xl:` | 576px |
| `@xs:` | 320px | `@2xl:` | 672px |
| `@sm:` | 384px | `@3xl:` | 768px |
| `@md:` | 448px | `@4xl:` | 896px |
| `@lg:` | 512px | `@5xl:` - `@7xl:` | 1024px - 1280px |

## Containers nommés

```html
<div class="@container/sidebar">
  <div class="@sm/sidebar:flex-row">
    <!-- Réagit à la taille du container 'sidebar' spécifiquement -->
  </div>
</div>
```

## Queries descendantes (max-width)

```html
<div class="@container">
  <div class="grid @max-md:grid-cols-1 @md:grid-cols-2">
    <!-- 1 colonne si container < 448px, 2 colonnes sinon -->
  </div>
</div>
```

## Cas d'usage recommandés

| Utiliser container queries | Utiliser media queries |
|---------------------------|------------------------|
| Composants réutilisables (cartes, widgets) | Layouts page-level (nav, footer) |
| Design systems "self-aware" | Changements basés sur device |
| Zones de tailles variables | Orientation viewport |

## Support navigateur

**93.92%** global (décembre 2025).
