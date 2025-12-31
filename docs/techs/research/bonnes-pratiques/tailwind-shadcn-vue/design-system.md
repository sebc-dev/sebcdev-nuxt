# Design System moderne avec Nuxt 4, TailwindCSS 4.1.x et shadcn-vue

La combinaison TailwindCSS 4, shadcn-vue 2.4.3+ et Nuxt 4 en mode SSG représente l'état de l'art du développement frontend fin 2025. L'approche **CSS-first** de Tailwind 4 via la directive `@theme` remplace totalement le fichier `tailwind.config.js`, tandis que le format de couleur **OKLCH** offre une uniformité perceptuelle supérieure avec un support navigateur dépassant **92%**. Pour Cloudflare Pages, un bundle CSS optimisé ne dépasse pas **5-10 KB** compressé en Brotli.

---

## Configuration CSS-native avec @theme pour les color tokens

TailwindCSS 4.1.x introduit une rupture majeure : toute la configuration se fait directement en CSS. La directive `@theme` génère automatiquement les classes utilitaires correspondantes à partir de variables CSS organisées en namespaces (`--color-*` pour `bg-`, `text-`, `border-`, etc.).

```css
/* assets/css/main.css */
@import "tailwindcss";

/* Dark mode via classe .dark */
@custom-variant dark (&:where(.dark, .dark *));

:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
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
  --radius: 0.625rem;
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --primary: oklch(0.922 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --destructive: oklch(0.704 0.191 22.216);
  --border: oklch(1 0 0 / 10%);
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-destructive: var(--destructive);
  --color-border: var(--border);
  --color-ring: var(--ring);
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
}
```

Le choix **OKLCH plutôt que HSL** s'impose désormais en production. OKLCH garantit une uniformité perceptuelle (modifier la luminosité ne change pas la saturation apparente), supporte le gamut Display P3 (30% de couleurs supplémentaires), et produit des gradients sans zones grises. La syntaxe `oklch(L C H)` utilise une luminosité de 0 à 1, un chroma de 0 à ~0.4, et une teinte de 0 à 360. Pour les gris neutres, le chroma est simplement à 0.

---

## Échelle typographique avec variable fonts optimisées

L'auto-hébergement des fonts Inter et JetBrains Mono en version **variable** surpasse Google Fonts pour le SSG : même domaine (pas de connexion externe), contrôle total du cache, et preload efficace. Une variable font unique remplace 6-12 fichiers statiques tout en offrant tous les weights de 100 à 900.

```css
/* @font-face AVANT @import "tailwindcss" */
@font-face {
  font-family: "Inter Variable";
  font-style: normal;
  font-weight: 100 900;
  font-display: optional;
  src: url("/fonts/Inter-Variable-latin.woff2") format("woff2-variations");
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6,
                 U+02DA, U+02DC, U+2000-206F, U+20AC, U+2122, U+FEFF;
}

@font-face {
  font-family: "JetBrains Mono Variable";
  font-style: normal;
  font-weight: 100 800;
  font-display: optional;
  src: url("/fonts/JetBrainsMono-Variable-latin.woff2") format("woff2-variations");
}

@import "tailwindcss";

@theme {
  --font-sans: "Inter Variable", ui-sans-serif, system-ui, sans-serif;
  --font-mono: "JetBrains Mono Variable", ui-monospace, Menlo, monospace;
  --font-sans--font-feature-settings: "cv01", "cv02", "ss01";
  
  --text-xs: 0.75rem;
  --text-xs--line-height: 1rem;
  --text-xs--letter-spacing: 0.02em;
  
  --text-base: 1rem;
  --text-base--line-height: 1.625rem;
  
  --text-4xl: 2.25rem;
  --text-4xl--line-height: 2.5rem;
  --text-4xl--letter-spacing: -0.03em;
}
```

La stratégie `font-display: optional` combinée au preload **élimine totalement le CLS** (Cumulative Layout Shift). Le navigateur utilise la font si elle est déjà en cache, sinon il affiche directement la fallback sans swap. Le subsetting avec `pyftsubset` réduit Inter de **~750 KB à ~70 KB** (réduction de 90%). Le preload dans `nuxt.config.ts` avec `crossorigin="anonymous"` est obligatoire même en self-hosting.

---

## Spacing sémantique avec fluid design

Le système de spacing moderne combine des tokens sémantiques hiérarchiques (page → section → component) avec des valeurs fluides via `clamp()`. La variable centrale `--spacing` de Tailwind 4 sert de base multiplicative pour toutes les utilities numériques (`p-4` = `calc(var(--spacing) * 4)`).

```css
@theme {
  --spacing: 0.25rem;
  
  /* Tokens page-level avec clamp(min, preferred, max) */
  --spacing-page-x: clamp(1rem, 5vw, 6rem);
  --spacing-page-y: clamp(2rem, 6vw, 4rem);
  
  /* Tokens section-level */
  --spacing-section: clamp(3rem, 6vw, 6rem);
  --spacing-section-sm: clamp(2rem, 4vw, 4rem);
  
  /* Tokens component-level */
  --spacing-component: clamp(1rem, 2vw, 1.5rem);
  
  /* Container queries */
  --container-prose: 65ch;
  --container-wide: 80rem;
}
```

Les **container queries** natives de Tailwind 4 permettent un responsive au niveau du composant plutôt que du viewport :

```html
<section class="@container py-(--spacing-section)">
  <div class="grid gap-4 @md:grid-cols-2 @lg:grid-cols-3 @lg:gap-6">
    <div class="p-(--spacing-component) @sm:p-6">Contenu adaptatif</div>
  </div>
</section>
```

---

## Patterns CVA et organisation des composants Nuxt 4

La fonction `cva()` de Class Variance Authority structure les variants de composants avec typage TypeScript complet. L'utilitaire `cn()` combinant `clsx` et `tailwind-merge` résout intelligemment les conflits de classes Tailwind.

```typescript
// app/lib/utils.ts
import type { ClassValue } from 'clsx'
import { clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

```typescript
// app/components/ui/button/index.ts
import { cva, type VariantProps } from 'class-variance-authority'

export const buttonVariants = cva(
  'inline-flex items-center justify-center gap-2 rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
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
export { default as Button } from './Button.vue'
```

La structure Nuxt 4 avec `srcDir: 'app/'` (implicite avec `compatibilityVersion: 4`) place les composants dans `app/components/ui/`. Le fichier `components.json` à la racine configure shadcn-vue :

```json
{
  "$schema": "https://shadcn-vue.com/schema.json",
  "style": "new-york",
  "typescript": true,
  "tailwind": {
    "css": "app/assets/css/main.css",
    "baseColor": "neutral",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui"
  }
}
```

---

## Accessibilité native avec Reka UI primitives

Reka UI 2.7.0, la base de shadcn-vue, implémente automatiquement les patterns WAI-ARIA : attributs `aria-*` appropriés, navigation clavier complète (Tab, Escape, flèches directionnelles), gestion intelligente du focus (trap dans les modals, restauration à la fermeture), et annonces pour lecteurs d'écran.

```vue
<!-- app/components/ui/button/Button.vue -->
<script setup lang="ts">
import { Primitive, type PrimitiveProps } from 'reka-ui'
import { cn } from '@/lib/utils'
import { buttonVariants, type ButtonVariants } from '.'

interface Props extends PrimitiveProps, ButtonVariants {
  class?: string
}

const props = withDefaults(defineProps<Props>(), {
  as: 'button',
  variant: 'default',
  size: 'default',
})
</script>

<template>
  <Primitive
    :as="as"
    :as-child="asChild"
    :class="cn(buttonVariants({ variant, size }), props.class)"
  >
    <slot />
  </Primitive>
</template>
```

Le pattern `as-child` préserve la sémantique HTML native. L'état `data-state` automatique (`open`/`closed`) facilite le styling conditionnel. Pour le dark mode, `@nuxtjs/color-mode` avec `classSuffix: ''` génère la classe `.dark` attendue par les variables CSS.

---

## Configuration optimale pour SSG sur Cloudflare Pages

Le plugin `@tailwindcss/vite` offre des performances supérieures à PostCSS avec intégration native de Lightning CSS. La détection automatique des contenus scanne tous les fichiers sauf `node_modules/` et `.gitignore`. Pour le safelist de classes dynamiques, `@source inline()` remplace l'ancienne option JavaScript.

```typescript
// nuxt.config.ts
import tailwindcss from "@tailwindcss/vite"

export default defineNuxtConfig({
  css: ['~/assets/css/main.css'],
  
  vite: {
    plugins: [tailwindcss()],
    build: {
      cssMinify: 'lightningcss',
    },
  },
  
  modules: ['shadcn-nuxt', '@nuxtjs/color-mode', '@nuxtjs/critters'],
  
  shadcn: { prefix: '', componentDir: '@/components/ui' },
  colorMode: { classSuffix: '' },
  critters: { config: { preload: 'swap' } },
  
  nitro: {
    preset: 'cloudflare-pages',
    prerender: { crawlLinks: true, routes: ['/'] },
    compressPublicAssets: { brotli: true },
  },
  
  routeRules: {
    '/_nuxt/**': { 
      headers: { 'cache-control': 'public, max-age=31536000, immutable' } 
    },
  },
})
```

Le fichier `public/_headers` configure le cache Cloudflare :

```
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

/*.woff2
  Cache-Control: public, max-age=31536000, immutable

/*.html
  Cache-Control: public, max-age=0, must-revalidate
```

Le module `@nuxtjs/critters` extrait le CSS critique et l'inline dans le `<head>`, chargeant le reste de manière asynchrone. Cette technique réduit le **FCP de ~1.8s à ~0.8s** et le **LCP de ~2.5s à ~1.2s**.

---

## Conclusion

Ce stack moderne privilégie une configuration déclarative en CSS pur, éliminant la fragmentation entre fichiers de config JavaScript et styles. Les **trois choix architecturaux clés** sont : OKLCH pour les couleurs (uniformité perceptuelle et gamut étendu), variable fonts avec `font-display: optional` (zéro CLS), et spacing fluide via `clamp()` (responsive sans media queries). L'accessibilité devient automatique grâce aux primitives Reka UI, tandis que la taille du bundle CSS optimisé (**5-10 KB** en Brotli) garantit des performances excellentes sur Cloudflare Pages avec un cache immutable d'un an pour les assets hashés.