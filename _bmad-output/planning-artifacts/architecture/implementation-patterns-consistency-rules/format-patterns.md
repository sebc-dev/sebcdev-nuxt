# Format Patterns

**Date Formatting:**
- Frontmatter: ISO 8601 (`2025-01-15`)
- Display: `Intl.DateTimeFormat` natif (zéro dépendance)

```typescript
// ✅ Correct
const formatDate = (date: Date, locale: string) =>
  new Intl.DateTimeFormat(locale, {
    day: 'numeric',
    month: 'long',
    year: 'numeric'
  }).format(date)
```

**Error Format (server routes):**

```typescript
// Format simple
{ error: string, statusCode: number }

// Exemple
{ error: "Article not found", statusCode: 404 }
```

**JSON Conventions:**
- camelCase pour toutes les propriétés
- Cohérent avec TypeScript/JavaScript

## CSS Patterns

### Shiki dual-theme (mode sombre)

Shiki génère des CSS variables pour chaque token lors du dual-theme. Intégration avec TailwindCSS 4 et `@nuxtjs/color-mode` :

```css
/* app/assets/css/shiki.css */

/* Switch automatique vers thème dark via classe html.dark */
html.dark .shiki,
html.dark .shiki span {
  color: var(--shiki-dark) !important;
  background-color: var(--shiki-dark-bg) !important;
  font-style: var(--shiki-dark-font-style) !important;
}

/* Styling des lignes highlight/diff */
.shiki .line.highlighted {
  background: oklch(0.95 0.01 250);
}
html.dark .shiki .line.highlighted {
  background: oklch(0.25 0.02 250);
}

.shiki .line.diff.add {
  background: oklch(0.95 0.05 145);
}
html.dark .shiki .line.diff.add {
  background: oklch(0.25 0.05 145);
}

.shiki .line.diff.remove {
  background: oklch(0.95 0.05 25);
}
html.dark .shiki .line.diff.remove {
  background: oklch(0.25 0.05 25);
}

/* Focus blur pour // [!code focus] */
.shiki.has-focused .line:not(.focused) {
  opacity: 0.5;
  transition: opacity 0.2s;
}
.shiki.has-focused:hover .line:not(.focused) {
  opacity: 1;
}
```

### Accessibilité code blocks

```css
/* Focus visible pour navigation clavier */
pre.shiki:focus-visible {
  outline: 2px solid oklch(0.6 0.15 250);
  outline-offset: 2px;
}

/* Amélioration scroll horizontal */
pre.shiki {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

/* Indicateur scroll (optionnel) */
pre.shiki::after {
  content: '';
  position: absolute;
  right: 0;
  top: 0;
  bottom: 0;
  width: 24px;
  background: linear-gradient(to right, transparent, var(--shiki-dark-bg, #1e1e1e));
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.2s;
}

pre.shiki:has(:horizontal-overflow)::after {
  opacity: 1;
}
```

**Note :** Shiki ajoute automatiquement `tabindex="0"` aux éléments `<pre>` pour la navigation clavier des blocs scrollables.
