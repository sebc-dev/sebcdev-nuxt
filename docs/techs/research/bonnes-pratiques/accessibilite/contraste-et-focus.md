# Accessibilité WCAG 2.2 pour Nuxt 4, TailwindCSS 4 et shadcn-vue

L'accessibilité web moderne repose sur trois piliers fondamentaux : des **ratios de contraste conformes** (4.5:1 texte, 3:1 UI), des **focus indicators visibles sur tous les fonds** via la technique double-ring, et l'**exploitation intelligente des primitives accessibles** de Reka UI. En décembre 2025, WCAG 2.2 introduit des critères cruciaux sur le focus (2.4.11 Focus Not Obscured) qui imposent de repenser les headers sticky et modales. Le stack TailwindCSS 4.1 + shadcn-vue 2.4+ offre une base solide avec `focus-visible:`, `outline` natif, et les attributs ARIA gérés automatiquement — mais atteindre la conformité AA nécessite une configuration intentionnelle et des tests automatisés dans votre pipeline CI.

## Les ratios de contraste WCAG 2.2 sont stricts mais stables

Les critères de contraste n'ont pas changé entre WCAG 2.1 et 2.2, garantissant une rétrocompatibilité totale. Le critère **1.4.3 (Contrast Minimum)** exige un ratio de **4.5:1 pour le texte normal** (inférieur à 24px ou 18.67px en bold) et **3:1 pour le texte large** (≥24px ou ≥19px bold). Ce seuil de "texte large" correspond exactement à **1.5rem** en base 16px. Le critère **1.4.11 (Non-text Contrast)** étend l'exigence de 3:1 aux composants UI, bordures, états graphiques et focus indicators — incluant les états hover, focus et active.

Pour un projet utilisant oklch avec TailwindCSS 4, la vérification des contrastes nécessite l'outil **OddContrast** qui supporte nativement cet espace colorimétrique moderne. Les outils classiques (WebAIM Contrast Checker) fonctionnent uniquement avec les conversions RGB. En mode sombre, privilégiez des tokens oklch avec une luminosité (première valeur) clairement différenciée : `oklch(0.985 0 0)` pour les surfaces claires versus `oklch(0.21 0.006 285)` pour les surfaces sombres.

## WCAG 2.2 transforme la gestion du focus avec trois nouveaux critères

Le critère **2.4.11 Focus Not Obscured (Minimum)** au niveau AA impose qu'un élément focusé ne soit jamais **entièrement masqué** par du contenu créé par l'auteur — headers sticky, bannières cookies, chatbots flottants. La solution CSS est élégante :

```css
html {
  scroll-padding-top: var(--header-height, 80px);
  scroll-padding-bottom: 50px;
}
```

Le critère **2.4.13 Focus Appearance** (niveau AAA mais recommandé comme bonne pratique) définit précisément la surface minimale d'un focus indicator : l'aire doit être **au moins égale au périmètre de l'élément × 2px**. Pour un bouton de 150×75px, cela représente 900px² minimum. Un outline solide de 2px d'épaisseur satisfait automatiquement cette exigence.

Ces critères imposent également un contraste **3:1 entre l'état focusé et non-focusé** (changement de contraste), distinct du contraste adjacent mesuré par le critère 1.4.11. Un focus indicator doit donc satisfaire les deux : contraste suffisant contre les couleurs adjacentes ET changement visuel perceptible entre les états.

## La technique double-ring garantit la visibilité universelle du focus

Un seul outline coloré échoue inévitablement sur certains fonds — le bleu disparaît sur fond sombre, le blanc sur fond clair. La technique double-ring résout ce problème mathématiquement : en superposant deux anneaux de couleurs ayant un contraste **9:1 entre eux**, au moins l'un des deux est garanti d'avoir un contraste 3:1 contre n'importe quel fond solide.

L'implémentation combine `outline` (préservé en Windows High Contrast Mode) et `box-shadow` :

```css
*:focus-visible {
  outline: 2px solid #FFFFFF;
  outline-offset: 0;
  box-shadow: 0 0 0 4px #193146;
}
```

Le `box-shadow` de 4px est nécessaire car l'outline de 2px le chevauche — seuls 2px du shadow seront visibles, créant l'effet deux couleurs. En mode sombre, inversez simplement les couleurs : outline noir `#0A0A0A`, shadow clair `#FAFAFA`.

**Attention critique** : ne jamais utiliser `outline: none` avec uniquement `box-shadow`. Les user agents suppriment souvent les box-shadows en mode couleurs forcées (High Contrast). Utilisez plutôt `outline: 2px solid transparent` pour maintenir l'accessibilité.

Avec TailwindCSS 4, cette technique s'implémente ainsi :

```html
<button class="
  focus-visible:outline-2 
  focus-visible:outline-white 
  focus-visible:outline-offset-0
  focus-visible:ring-4 
  focus-visible:ring-slate-900
  dark:focus-visible:outline-black
  dark:focus-visible:ring-white
">
  Bouton accessible
</button>
```

## TailwindCSS 4.1 modernise profondément la gestion du focus

Le changement majeur entre v3 et v4 concerne la classe `ring` : elle passe de **3px à 1px par défaut**. Lors d'une migration, remplacez `ring` par `ring-3` pour conserver le comportement précédent. Plus important, `outline-none` devient `outline-hidden` — cette nouvelle classe maintient un outline transparent visible en Windows High Contrast Mode, contrairement à `outline-none` qui applique littéralement `outline-style: none`.

La distinction entre `ring` et `outline` est fondamentale pour l'accessibilité :

| Propriété | ring | outline |
|-----------|------|---------|
| CSS sous-jacent | `box-shadow` | `outline` natif |
| Windows High Contrast | ❌ Non visible | ✅ Visible |
| Usage recommandé | Effets décoratifs | **Focus indicators** |

La configuration `@theme` CSS-native de TailwindCSS 4 simplifie la définition de tokens accessibles :

```css
@import "tailwindcss";

:root {
  --focus-color: oklch(0.623 0.214 259.815);
  --focus-offset-color: oklch(0.985 0 0);
}

.dark {
  --focus-color: oklch(0.707 0.165 254.624);
  --focus-offset-color: oklch(0.141 0.005 285.823);
}

@theme inline {
  --color-focus: var(--focus-color);
  --color-focus-offset: var(--focus-offset-color);
}

@layer base {
  *:focus-visible {
    outline: 2px solid var(--color-focus-outline);
    outline-offset: 2px;
  }
  
  @media (prefers-reduced-motion: reduce) {
    *, ::before, ::after {
      animation-duration: 0.01ms !important;
      transition-duration: 0.01ms !important;
    }
  }
}
```

Les classes `motion-safe:` et `motion-reduce:` permettent de respecter la préférence système pour les animations. Appliquez systématiquement `motion-safe:animate-spin` plutôt que `animate-spin` seul, et ajoutez `motion-reduce:transition-none` sur les transitions non essentielles.

## Reka UI et shadcn-vue gèrent automatiquement les attributs ARIA

Reka UI (anciennement Radix Vue) implémente les WAI-ARIA Authoring Practices sans configuration. Pour un composant Dialog, les attributs `role="dialog"`, `aria-modal="true"`, `aria-labelledby` et `aria-describedby` sont ajoutés automatiquement. Le focus trapping est actif par défaut, et les événements `@openAutoFocus`, `@closeAutoFocus` permettent de personnaliser le comportement du focus.

Les Tabs gèrent `role="tablist"`, `role="tab"`, `role="tabpanel"`, `aria-selected` et `aria-controls` sans intervention. La navigation clavier (flèches, Home, End) fonctionne nativement, avec un mode d'activation configurable (`automatic` pour sélection au focus, `manual` pour sélection explicite).

shadcn-vue utilise la variable CSS `--ring` pour tous les focus rings :

```css
:root {
  --ring: oklch(0.708 0 0);
  --ring-inner: oklch(1 0 0);
  --ring-outer: oklch(0.205 0 0);
}

.dark {
  --ring-inner: oklch(0.145 0 0);
  --ring-outer: oklch(0.985 0 0);
}
```

Le pattern standard dans les composants shadcn-vue utilise `focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2`. Pour implémenter le double-ring, modifiez les variants dans `cva()` ou créez un override global dans votre CSS.

**Migration Radix Vue → Reka UI** : les préfixes CSS passent de `--radix-*` à `--reka-*`, et le v-model se simplifie (`v-model:checked` devient `v-model`). shadcn-vue 2.4.3+ utilise Reka UI par défaut ; pour l'ancienne version, utilisez `npx shadcn-vue@radix`.

## Les tests d'accessibilité automatisés couvrent environ 30% des problèmes

axe-core reste la référence pour les tests automatisés, avec un support WCAG 2.2 via le tag `wcag22aa`. L'intégration avec Playwright est la plus efficace pour un projet Nuxt SSG :

```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('Page sans violations WCAG 2.2 AA', async ({ page }) => {
  await page.goto('/');
  
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'])
    .analyze();
  
  expect(results.violations).toEqual([]);
});
```

Pour les tests unitaires avec Vitest, utilisez `vitest-axe` — mais notez que la règle `color-contrast` ne fonctionne pas avec JSDOM. Activez **Vitest Browser Mode** pour des tests de contraste réels avec rendu CSS.

Lighthouse CI s'intègre facilement dans GitHub Actions avec un seuil d'accessibilité configurable :

```json
{
  "ci": {
    "assert": {
      "assertions": {
        "categories:accessibility": ["error", { "minScore": 0.9 }]
      }
    }
  }
}
```

Les simulateurs de daltonisme intégrés aux DevTools Chrome (Rendering → Emulate vision deficiencies) permettent de tester protanopia, deuteranopia et tritanopia directement. L'extension **WAVE** (v3.3.0+) supporte maintenant les couleurs avec alpha/opacity et aligne ses erreurs sur WCAG 2.2.

Le stack CI recommandé pour votre projet Nuxt 4 + Cloudflare Pages combine :
- **Développement** : eslint-plugin-vuejs-accessibility + WAVE extension
- **Tests unitaires** : vitest-axe en Browser Mode
- **Tests E2E** : @axe-core/playwright sur les pages SSG générées
- **CI** : Lighthouse CI (score ≥90%) + Pa11y-ci sur le sitemap

## Configuration complète pour un projet accessible

Voici la configuration `app.css` intégrant toutes les bonnes pratiques :

```css
@import "tailwindcss";
@plugin "@tailwindcss/forms";

:root {
  --focus-ring: oklch(0.585 0.233 277.117);
  --focus-inner: oklch(1 0 0);
  --focus-outer: oklch(0.205 0 0);
  --header-height: 80px;
}

.dark {
  --focus-ring: oklch(0.673 0.182 276.935);
  --focus-inner: oklch(0.145 0 0);
  --focus-outer: oklch(0.985 0 0);
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
  
  *:focus-visible {
    outline: 2px solid var(--color-focus-inner);
    outline-offset: 0;
    box-shadow: 0 0 0 4px var(--color-focus-outer);
  }
  
  @media (prefers-reduced-motion: reduce) {
    *, ::before, ::after {
      animation-duration: 0.01ms !important;
      transition-duration: 0.01ms !important;
    }
  }
}
```

## Conclusion et checklist de validation

La conformité WCAG 2.2 AA pour un projet moderne exige une approche systématique. Les primitives Reka UI gèrent l'accessibilité sémantique (ARIA, focus trapping, navigation clavier), mais les aspects visuels — contraste et focus indicators — relèvent de votre responsabilité de configuration.

Les points critiques à valider avant mise en production :
- Tous les textes normaux ont un contraste ≥4.5:1, les textes larges et UI ≥3:1
- Les focus indicators utilisent la technique double-ring avec couleurs inversées en mode sombre
- Les headers sticky n'obscurcissent pas les éléments focusés (`scroll-padding-top`)
- `focus-visible:` est utilisé plutôt que `focus:` pour éviter les focus rings au clic souris
- Les animations respectent `prefers-reduced-motion`
- axe-core s'exécute dans le pipeline CI avec tag `wcag22aa`

L'automatisation détecte environ 30% des problèmes d'accessibilité ; les 70% restants nécessitent des tests manuels avec clavier et lecteur d'écran (VoiceOver sur macOS, NVDA sur Windows). Un score Lighthouse de 100 ne garantit pas l'accessibilité — il valide uniquement les critères automatisables.