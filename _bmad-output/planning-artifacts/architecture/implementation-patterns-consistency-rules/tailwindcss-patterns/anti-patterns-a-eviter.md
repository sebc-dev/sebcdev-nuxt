# Anti-patterns à éviter

## ❌ Utiliser @nuxtjs/tailwindcss avec TW4

```typescript
// ❌ NON compatible TailwindCSS 4
modules: ['@nuxtjs/tailwindcss']

// ✅ Utiliser le plugin Vite direct
import tailwindcss from '@tailwindcss/vite'
export default defineNuxtConfig({
  vite: { plugins: [tailwindcss()] }
})
```

## ❌ Mélanger @theme et :root pour theming

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

## ❌ Oublier @custom-variant dark

```css
/* ❌ Le dark mode shadcn-vue ne fonctionnera pas */
@import "tailwindcss";

/* ✅ Définir explicitement le variant */
@import "tailwindcss";
@custom-variant dark (&:is(.dark *));
```

## ❌ Concaténation dynamique de classes

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
