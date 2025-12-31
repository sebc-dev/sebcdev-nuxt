# Breaking Changes v3 → v4

## Renommages de classes

| v3 | v4 |
|----|----|
| `shadow-sm` | `shadow-xs` |
| `shadow` | `shadow-sm` |
| `bg-gradient-to-*` | `bg-linear-to-*` |
| `rounded-sm` | `rounded-xs` |
| `ring` | `ring-1` (défaut passe de 3px à 1px) |
| `outline-none` | `outline-hidden` (pour WHCM) |

## Focus et Accessibilité - Changements critiques

### `ring` passe de 3px à 1px

```css
/* v3 : ring = 3px par défaut */
.element { @apply ring; }  /* → box-shadow: 0 0 0 3px */

/* v4 : ring = 1px par défaut */
.element { @apply ring; }  /* → box-shadow: 0 0 0 1px */

/* Migration : utiliser ring-3 pour conserver le comportement v3 */
.element { @apply ring-3; }
```

### `outline-none` vs `outline-hidden`

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

### `ring` vs `outline` pour l'accessibilité

| Propriété | `ring` | `outline` |
|-----------|--------|-----------|
| CSS sous-jacent | `box-shadow` | `outline` natif |
| Windows High Contrast | ❌ Non visible | ✅ Visible |
| Usage recommandé | Effets décoratifs | **Focus indicators** |

**Pattern recommandé** : Combiner `outline` (pour WHCM) et `ring` (pour style) via la technique double-ring.

## Syntaxe variables arbitraires

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

## Dark mode par défaut

| v3 | v4 |
|----|----|
| Basé sur classe `.dark` | Basé sur `prefers-color-scheme` |
| Config: `darkMode: 'class'` | Config: `@custom-variant dark (...)` |

Pour utiliser le mode classe avec shadcn-vue :

```css
/* Obligatoire pour shadcn-vue */
@custom-variant dark (&:is(.dark *));
```

## Outil de migration

```bash
# Migration automatique v3 → v4
npx @tailwindcss/upgrade@next
```
