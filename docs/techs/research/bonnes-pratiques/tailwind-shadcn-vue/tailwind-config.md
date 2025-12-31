# TailwindCSS 4 et Nuxt 4 : guide complet des bonnes pratiques 2025

TailwindCSS v4 introduit un changement de paradigme majeur : la configuration passe du JavaScript au CSS natif. Pour un projet Nuxt 4 avec shadcn-vue déployé en SSG sur Cloudflare Pages, cela signifie des builds **5 à 10 fois plus rapides**, une configuration simplifiée, et un bundle CSS optimisé grâce à Lightning CSS. Ce guide couvre les six piliers essentiels : configuration CSS-first, design tokens, OKLCH, container queries, Lightning CSS, et l'intégration spécifique à votre stack.

---

## La révolution CSS-first de TailwindCSS 4

Le changement le plus significatif de TailwindCSS v4 est l'abandon du fichier `tailwind.config.js` au profit d'une configuration entièrement en CSS. L'ancienne syntaxe avec trois directives distinctes (`@tailwind base`, `@tailwind components`, `@tailwind utilities`) est remplacée par un unique import :

```css
/* app/assets/css/main.css - Configuration minimale */
@import "tailwindcss";
```

Cette seule ligne importe automatiquement les layers theme, base, components et utilities. La directive `@theme {}` devient le cœur de la personnalisation. Elle définit les design tokens qui génèrent directement les classes utilitaires :

```css
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.62 0.21 259);
  --color-brand-500: oklch(0.84 0.18 117.33);
  --font-display: "Satoshi", sans-serif;
  --radius-xl: 0.75rem;
  --ease-fluid: cubic-bezier(0.3, 0, 0, 1);
}
```

Chaque variable dans `@theme` génère automatiquement les classes correspondantes : `--color-primary` produit `bg-primary`, `text-primary`, `border-primary`, etc. Pour **étendre** le thème par défaut, ajoutez simplement vos tokens. Pour **remplacer** un namespace entier, utilisez `initial` :

```css
@theme {
  --color-*: initial;  /* Supprime TOUTES les couleurs par défaut */
  --color-white: #fff;
  --color-primary: oklch(0.62 0.21 259);
}
```

L'intégration avec Vite se fait via **@tailwindcss/vite**, le plugin officiellement recommandé pour les projets Vite et Nuxt :

```typescript
// nuxt.config.ts
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  vite: {
    plugins: [tailwindcss()],
  },
  css: ['~/assets/css/main.css'],
})
```

La migration depuis v3 s'effectue avec l'outil automatique `npx @tailwindcss/upgrade@next`. Les principales breaking changes incluent le renommage de certaines classes (`shadow-sm` → `shadow-xs`, `bg-gradient-*` → `bg-linear-*`), le dark mode basé par défaut sur `prefers-color-scheme`, et la nouvelle syntaxe pour les variables arbitraires : `bg-(--my-color)` au lieu de `bg-[--my-color]`.

---

## Design tokens et organisation des variables CSS

TailwindCSS 4 introduit une convention de nommage structurée pour les design tokens. Chaque namespace correspond à une famille d'utilitaires :

| Namespace | Classes générées |
|-----------|------------------|
| `--color-*` | `bg-*`, `text-*`, `border-*`, `fill-*` |
| `--font-*` | `font-*` (famille) |
| `--spacing-*` | `p-*`, `m-*`, `w-*`, `h-*`, `gap-*` |
| `--radius-*` | `rounded-*` |
| `--shadow-*` | `shadow-*` |
| `--breakpoint-*` | `sm:`, `md:`, `lg:`, `xl:` |

La distinction clé entre `@theme` et `:root` est que **`@theme` génère des classes utilitaires**, tandis que `:root` crée uniquement des variables CSS. Pour des variables dynamiques (comme le theming dark/light), utilisez `@theme inline` qui référence des variables externes :

```css
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
}
```

Cette approche permet de changer dynamiquement les couleurs via JavaScript tout en conservant les classes Tailwind. L'interopérabilité avec **shadcn-vue** suit exactement ce pattern : shadcn utilise une convention sémantique `--background`, `--foreground`, `--primary`, `--muted`, etc., que vous connectez à Tailwind via `@theme inline`.

---

## OKLCH : l'espace colorimétrique moderne

TailwindCSS 4 adopte **OKLCH** comme format de couleur par défaut. OKLCH (Oklab Lightness Chroma Hue) est un espace colorimétrique perceptuellement uniforme créé par Björn Ottosson en 2020. Sa syntaxe est `oklch(L C H / alpha)` où **L** (0-1) représente la luminosité perceptuelle, **C** (0-0.4) l'intensité chromatique, et **H** (0-360°) la teinte.

Les avantages par rapport à HSL/RGB sont significatifs :

- **Uniformité perceptuelle** : modifier L de 0.1 produit un changement visuel cohérent quelle que soit la teinte, contrairement à HSL où jaune et bleu à 50% paraissent très différents
- **Gamut étendu** : OKLCH peut exprimer des couleurs Display P3 et Rec2020, exploitant les écrans modernes
- **Gradients fluides** : pas de zone grise dans les transitions entre couleurs saturées
- **Prédictibilité** : ajuster L/C/H produit des résultats attendus, facilitant la création de palettes cohérentes

L'implémentation du dark mode avec OKLCH suit ce pattern :

```css
:root {
  --primary: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);
}

.dark {
  --primary: oklch(0.922 0 0);
  --destructive: oklch(0.704 0.191 22.216);
}
```

Le support navigateur d'OKLCH atteint **92.86%** globalement (Chrome 111+, Safari 15.4+, Firefox 113+). Pour les anciens navigateurs, Lightning CSS peut transpiler vers RGB avec fallbacks, mais TailwindCSS 4 cible par défaut des navigateurs qui supportent déjà OKLCH, rendant cette transpilation généralement inutile.

---

## Container queries : responsive au niveau composant

TailwindCSS 4 intègre nativement les container queries, supprimant le besoin du plugin `@tailwindcss/container-queries`. La syntaxe utilise le préfixe `@` pour distinguer des media queries traditionnelles :

```html
<div class="@container">
  <div class="flex flex-col @md:flex-row @lg:grid @lg:grid-cols-3">
    <!-- Layout adaptatif selon la taille du container parent -->
  </div>
</div>
```

Les variantes disponibles couvrent une large gamme : `@3xs:` (256px), `@xs:` (320px), `@sm:` (384px), `@md:` (448px), `@lg:` (512px), `@xl:` (576px), `@2xl:` (672px), jusqu'à `@7xl:` (1280px). TailwindCSS 4 ajoute également `@max-*` pour les queries descendantes et les containers nommés :

```html
<div class="@container/sidebar">
  <div class="@sm/sidebar:flex-row">
    <!-- Réagit à la taille du container 'sidebar' -->
  </div>
</div>
```

La différence fondamentale avec les media queries est la **référence** : les media queries se basent sur le viewport (fenêtre), tandis que les container queries réagissent à l'élément parent. Cela rend les composants véritablement réutilisables — une carte produit s'adapte automatiquement qu'elle soit dans une sidebar étroite ou un main content large.

**Cas d'usage recommandés** pour les container queries :
- Composants réutilisables (cartes, widgets)
- Design systems avec composants "self-aware"
- Layouts complexes avec zones de tailles variables

**Conserver les media queries** pour les layouts page-level (navigation, footer) et les changements basés sur les capacités device. Le support navigateur des container queries atteint **93.92%** en décembre 2025.

---

## Lightning CSS : performance et transpilation automatique

Lightning CSS est le moteur CSS de TailwindCSS 4, un outil Rust capable de traiter **2.7 millions de lignes CSS par seconde**. Il combine parsing, transpilation, bundling et minification en une seule passe, remplaçant postcss-import, autoprefixer, postcss-nesting, et cssnano.

Les **fonctionnalités de transpilation automatique** incluent :

| Feature moderne | Transpilation |
|-----------------|---------------|
| CSS Nesting | Sélecteurs aplatis |
| oklch/lab/lch | RGB + Display P3 fallbacks |
| color-mix() | Couleur calculée statiquement |
| Media query ranges `(480px <= width <= 768px)` | Syntaxe min/max |
| light-dark() | CSS variables fallback |

Les **préfixes vendeurs** sont gérés automatiquement : Lightning CSS ajoute les préfixes nécessaires selon vos browser targets et supprime les préfixes obsolètes. Les optimisations de **minification** sont agressives : conversion shorthand, fusion de règles adjacentes, réduction des expressions calc(), suppression des valeurs inférables.

**Benchmarks officiels** sur bootstrap-4.css : cssnano prend 544ms pour 159KB, esbuild 17ms pour 160KB, Lightning CSS **4ms pour 143KB**. C'est 100x plus rapide que cssnano avec un output plus petit.

Pour Vite/Nuxt, aucune configuration supplémentaire n'est nécessaire — `@tailwindcss/vite` utilise Lightning CSS automatiquement :

```typescript
import tailwindcss from '@tailwindcss/vite'
export default defineConfig({ plugins: [tailwindcss()] })
```

---

## Configuration optimale Nuxt 4 + shadcn-vue + Cloudflare Pages

Pour votre stack spécifique (Nuxt 4.2.x, shadcn-vue 2.4.3+, Reka UI 2.7.0, SSG, Cloudflare Pages), voici la configuration complète.

**Installation** :
```bash
pnpm add -D tailwindcss @tailwindcss/vite
pnpm dlx nuxi@latest module add shadcn-nuxt
pnpm dlx shadcn-vue@latest init
```

**nuxt.config.ts** :
```typescript
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  compatibilityDate: '2024-11-01',
  future: { compatibilityVersion: 4 },
  
  css: ['~/assets/css/main.css'],
  vite: { plugins: [tailwindcss()] },
  
  modules: ['shadcn-nuxt'],
  shadcn: {
    prefix: '',
    componentDir: '@/components/ui'
  },
  
  build: {
    transpile: ['reka-ui', 'vee-validate'],
  },
  
  nitro: {
    preset: 'cloudflare_pages',
    minify: true,
    compressPublicAssets: true,
    prerender: { crawlLinks: true, routes: ['/'] },
  },
  
  routeRules: {
    '/**': { prerender: true },
  },
})
```

**N'utilisez pas @nuxtjs/tailwindcss** avec TailwindCSS 4 — le module n'est pas encore officiellement compatible et repose sur l'ancienne configuration JS. Le plugin Vite direct est l'approche recommandée.

**app/assets/css/main.css** avec shadcn-vue :
```css
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&:is(.dark *));

:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.97 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --border: oklch(0.922 0 0);
  --radius: 0.5rem;
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --primary: oklch(0.985 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.704 0.191 22.216);
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
  --radius-lg: var(--radius);
}
```

**Structure de fichiers Nuxt 4** :
```
my-app/
├── app/                    # srcDir (code client)
│   ├── assets/css/main.css
│   ├── components/ui/      # Composants shadcn-vue
│   ├── pages/
│   ├── layouts/
│   └── app.vue
├── server/                 # Code Nitro (vide en SSG pur)
├── public/
├── components.json         # Config shadcn-vue
└── nuxt.config.ts
```

**Déploiement Cloudflare Pages** : build command `pnpm generate`, output directory `.output/public`. Désactivez Rocket Loader, Mirage et Email Obfuscation dans les paramètres Cloudflare pour éviter les conflits avec l'hydratation Vue.

---

## Conclusion

TailwindCSS 4 représente une évolution majeure vers une architecture **CSS-native** qui simplifie drastiquement la configuration tout en améliorant les performances. Les points essentiels à retenir :

- **@tailwindcss/vite** est le seul plugin à utiliser avec Nuxt 4, pas @nuxtjs/tailwindcss
- **`@theme {}`** remplace tailwind.config.js pour définir les design tokens
- **OKLCH** offre une meilleure uniformité perceptuelle et un gamut étendu
- **Container queries** (`@md:`, `@lg:`) sont intégrés nativement pour le responsive au niveau composant
- **Lightning CSS** assure transpilation automatique, prefixes vendeurs, et minification optimale
- Le pattern **`:root` + `.dark` + `@theme inline`** permet un theming dynamique compatible shadcn-vue

Pour un budget 0€ sur Cloudflare Pages, le mode SSG avec `pnpm generate` et le preset `cloudflare_pages` offre des performances optimales sans server functions. La combinaison Lightning CSS + tree-shaking Vite garantit un bundle CSS minimal.