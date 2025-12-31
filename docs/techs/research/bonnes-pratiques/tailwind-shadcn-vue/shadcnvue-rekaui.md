# Intégrer shadcn-vue et Reka UI dans Nuxt 4 avec TailwindCSS 4

**shadcn-vue 2.4.3+ combiné à Reka UI 2.7.0 constitue désormais la solution de référence pour créer des interfaces accessibles dans Nuxt 4.** Cette stack moderne exploite TailwindCSS 4 en configuration CSS-native, abandonnant le fichier `tailwind.config.js` au profit de directives `@theme` et de tokens OKLCH. Pour un déploiement SSG sur Cloudflare Pages, la clé réside dans la transpilation correcte des modules (`reka-ui`, `vee-validate`) et l'utilisation du plugin `ssr-width` pour éviter les erreurs d'hydratation. Cette architecture garantit une accessibilité WAI-ARIA complète, un tree-shaking optimal et des performances edge-ready.

## Configuration initiale shadcn-vue avec Nuxt 4

L'installation démarre par `pnpm dlx shadcn-vue@latest init` après avoir créé un projet Nuxt 4. Les réponses recommandées au CLI sont : **TypeScript activé**, framework **Nuxt**, style **new-york** (le style "default" est déprécié), couleur de base **Neutral**, CSS global dans `app/assets/css/tailwind.css`, et **CSS variables activées**. Point crucial pour TailwindCSS 4 : le champ `tailwind.config` doit rester **vide** puisque la configuration se fait désormais directement en CSS.

Le module `shadcn-nuxt` s'installe via `pnpm dlx nuxi@latest module add shadcn-nuxt`. Sa configuration dans `nuxt.config.ts` définit le préfixe des composants (vide recommandé pour utiliser `Button` directement) et le répertoire UI (`@/components/ui`). L'alias `@/` pointe vers le dossier `app/` dans Nuxt 4 — attention à ne pas écrire `@/app/components` qui créerait un double chemin.

Le fichier `components.json` généré ressemble à :

```json
{
  "$schema": "https://shadcn-vue.com/schema.json",
  "style": "new-york",
  "typescript": true,
  "tailwind": {
    "config": "",
    "css": "app/assets/css/tailwind.css",
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

## TailwindCSS 4 CSS-native remplace la configuration JavaScript

TailwindCSS 4 abandonne `tailwind.config.js` pour une approche **CSS-first**. L'import se simplifie à `@import "tailwindcss"` suivi de `@import "tw-animate-css"` (qui remplace `tailwindcss-animate`). Le dark mode utilise désormais `@custom-variant dark (&:is(.dark *))` au lieu de `darkMode: 'class'`.

Les couleurs shadcn-vue passent au format **OKLCH**, plus moderne que HSL avec un gamut plus large. La syntaxe `oklch(Lightness Chroma Hue)` définit la luminosité (0-1), la saturation/chroma (0-0.4) et la teinte (0-360). Par exemple, `--background: oklch(1 0 0)` pour le blanc et `--destructive: oklch(0.577 0.245 27.325)` pour le rouge d'erreur.

La directive `@theme inline` mappe les variables CSS vers les utilitaires Tailwind sans duplication :

```css
@theme inline {
  --color-background: var(--background);
  --color-primary: var(--primary);
  --radius-lg: var(--radius);
}
```

Cette configuration génère automatiquement les classes `bg-background`, `text-primary`, `rounded-lg`, etc. Les animations personnalisées se déclarent également dans `@theme` avec `@keyframes`, par exemple pour l'accordion :

```css
@theme {
  --animate-accordion-down: accordion-down 0.2s ease-out;
  @keyframes accordion-down {
    from { height: 0; }
    to { height: var(--reka-accordion-content-height); }
  }
}
```

## Reka UI 2.7.0 garantit l'accessibilité WAI-ARIA

Reka UI (anciennement Radix Vue) implémente les **WAI-ARIA Authoring Practices** de façon exhaustive. Chaque composant expose automatiquement les attributs ARIA appropriés : `role="dialog"` et `aria-modal="true"` pour Dialog, `role="tablist"` pour Tabs, `role="menu"` pour DropdownMenu. Le **focus trapping** s'active automatiquement dans les modales quand `modal=true`, et le **roving tabindex** gère la navigation clavier dans les menus.

La navigation clavier suit des patterns standardisés :
- **Dialog** : `Escape` ferme et restaure le focus au trigger, `Tab` navigue entre éléments focusables
- **Tabs** : flèches directionnelles selon l'orientation, `Home`/`End` pour premier/dernier onglet
- **DropdownMenu** : typeahead pour sauter aux items par lettre, `ArrowRight`/`ArrowLeft` pour sous-menus
- **Accordion** : `Space`/`Enter` pour expand/collapse, navigation par flèches entre triggers

Pour les cas où le titre visuel diffère du titre accessible, Reka UI fournit `VisuallyHidden` :

```vue
<VisuallyHidden as-child>
  <DialogTitle>Titre pour lecteurs d'écran</DialogTitle>
</VisuallyHidden>
```

## Configuration build Nuxt 4 SSG pour Cloudflare Pages

La transpilation de certains modules est **obligatoire** pour éviter les erreurs SSR. `reka-ui` utilise des exports ES modules modernes incompatibles avec le serveur Node.js sans transformation, causant l'erreur `"Cannot split a chunk"`. `vee-validate/dist/rules` provoque `"Unexpected Token: export"` sans transpilation.

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true,
  
  build: {
    transpile: ['reka-ui', 'vee-validate', '@vee-validate/rules']
  },
  
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      crawlLinks: true,
      autoSubfolderIndex: false  // Important pour Cloudflare route matching
    }
  },
  
  vite: {
    plugins: [tailwindcss()]
  }
})
```

Pour Cloudflare Pages, **désactivez impérativement** dans le dashboard : Rocket Loader™, Mirage, et Email Address Obfuscation — ces optimisations injectent des scripts qui cassent l'hydratation Vue.

Le plugin `ssr-width` prévient les erreurs d'hydratation sur mobile en fournissant une largeur SSR cohérente :

```typescript
// app/plugins/ssr-width.ts
import { provideSSRWidth } from '@vueuse/core'

export default defineNuxtPlugin((nuxtApp) => {
  provideSSRWidth(1024, nuxtApp.vueApp)
})
```

## Composants shadcn-vue essentiels pour un blog

**Dialog** convient parfaitement aux lightbox d'images et formulaires de contact. L'API `v-model:open` permet le contrôle total de l'état, tandis que les événements `@escape-key-down` et `@pointer-down-outside` personnalisent le comportement de fermeture.

**Tabs** structure la navigation par catégorie d'articles. La prop `activation-mode="automatic"` active l'onglet au focus (navigation rapide), tandis que `"manual"` requiert `Enter` (meilleur pour les onglets chargeant du contenu).

**Accordion** s'utilise pour FAQ et sections pliables. Le type `"single"` avec `collapsible` permet de tout fermer, `"multiple"` autorise plusieurs sections ouvertes. La prop `unmount-on-hide="false"` garde le contenu dans le DOM pour la recherche navigateur.

**Toast** via le composable `useToast()` affiche les notifications (lien copié, formulaire envoyé). Le composant `<Toaster />` doit être placé à la racine de `app.vue`.

**Tooltip** et **DropdownMenu** complètent l'interface avec infobulles et menus contextuels, tous bénéficiant de la même accessibilité native.

## Patterns recommandés et anti-patterns critiques

**La composition par slots** crée des wrappers réutilisables. Un `BlogCard.vue` encapsule `Card` avec des slots `default` et `actions` pour contenu flexible. Le **forwarding de props** utilise `v-bind="$attrs"` avec `defineOptions({ inheritAttrs: false })`.

Pour le **contrôle d'état**, préférez `v-model:open` quand le parent doit réagir aux changements, et `default-value` pour un état interne non observable.

Les **anti-patterns critiques** à éviter absolument :
- **Accès `window`/`document` au setup** : utilisez `onMounted()` ou les composables VueUse
- **IDs dynamiques** (`Math.random()`) : causent hydration mismatch, utilisez `useId()` de Vue 3.5+
- **Event listeners sans cleanup** : préférez `useEventListener` de VueUse qui nettoie automatiquement
- **Dates formatées différemment serveur/client** : encapsulez dans `<ClientOnly>` ou utilisez `useCookie` pour la locale

Le **lazy loading** des composants lourds (Dialog avec images, graphiques) s'effectue via `defineAsyncComponent` ou les stratégies d'hydratation Nuxt (`<LazyComponent hydrate-on-visible />`).

## Performance et tests d'accessibilité

Le **tree-shaking** fonctionne efficacement car shadcn-vue copie les composants dans votre projet — seuls ceux ajoutés via `pnpm dlx shadcn-vue@latest add button dialog` sont inclus. Évitez les imports globaux `import * as UI` qui empêchent l'élimination du code mort.

Pour les tests d'accessibilité automatisés, `vitest-axe` s'intègre à Vitest :

```typescript
import { axe, toHaveNoViolations } from 'vitest-axe'
expect.extend(toHaveNoViolations)

it('composant accessible', async () => {
  const { container } = render(MonComposant)
  expect(await axe(container)).toHaveNoViolations()
})
```

Complétez par les extensions navigateur **axe DevTools** et **Lighthouse** pour les audits manuels. La checklist manuelle vérifie les labels descriptifs, le contraste des couleurs (minimum **4.5:1** pour le texte), et les zones tactiles (**44×44px** minimum).

## Conclusion

Cette stack shadcn-vue + Reka UI + TailwindCSS 4 sur Nuxt 4 SSG représente l'état de l'art pour les interfaces Vue accessibles en décembre 2025. Les points d'attention majeurs sont : **laisser vide le champ `tailwind.config`** dans components.json, **transpiler `reka-ui` et `vee-validate`**, utiliser le **plugin `ssr-width`** pour éviter les erreurs d'hydratation, et **désactiver les optimisations Cloudflare** qui interfèrent avec Vue. L'accessibilité WAI-ARIA native de Reka UI élimine la dette technique habituelle des composants UI, tandis que le passage à OKLCH et `@theme inline` modernise significativement la gestion des couleurs et du theming.