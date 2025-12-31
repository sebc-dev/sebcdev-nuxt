# Communication Patterns

**Vue Events:**
- kebab-case: `@article-selected`, `@filter-changed`

```vue
<!-- ✅ Correct -->
<ArticleCard @article-selected="handleSelect" />

<!-- ❌ Incorrect -->
<ArticleCard @articleSelected="handleSelect" />
```

**Component Props:**
- Inline `defineProps<{}>` pour composants simples
- Interface exportée si props réutilisées ailleurs

```typescript
// Simple component
defineProps<{
  article: Article
  showExcerpt?: boolean
}>()

// Reusable props
export interface ArticleCardProps {
  article: Article
  variant: 'card' | 'hero' | 'list'
}
```

**Loading States:**
- Naming: `isLoading`, `isPending`
- UI: `<Skeleton />` de shadcn-vue
- Pattern: destructure depuis `useAsyncData`

---

## Props Drilling vs Provide/Inject

**Guideline simple :** Props pour 1-2 niveaux, provide/inject pour 3+ niveaux.

| Profondeur | Solution recommandée | Raison |
|------------|---------------------|--------|
| **1-2 niveaux** | Props | Explicite, traçable, facile à débugger |
| **3+ niveaux** | Provide/Inject | Évite le "props drilling" fastidieux |
| **State global** | Composable partagé ou Pinia | Pattern dédié pour ce cas |

```vue
<!-- ❌ Props drilling excessif (3+ niveaux) -->
<GrandParent :theme="theme">
  <Parent :theme="theme">
    <Child :theme="theme">
      <GrandChild :theme="theme" />  <!-- Pénible à maintenir -->
    </Child>
  </Parent>
</GrandParent>

<!-- ✅ Provide/Inject pour 3+ niveaux -->
<GrandParent>  <!-- provide('theme', theme) -->
  <Parent>
    <Child>
      <GrandChild />  <!-- const theme = inject('theme') -->
    </Child>
  </Parent>
</GrandParent>
```

---

## InjectionKey Typées avec Readonly

**Pattern recommandé :** Utilisez `InjectionKey<T>` typées + `readonly()` pour un provide/inject type-safe et immutable.

```typescript
// types/injection-keys.ts
import type { InjectionKey, Ref } from 'vue'

// 1. Définir le type du contexte
interface ThemeContext {
  theme: Readonly<Ref<'light' | 'dark'>>
  toggleTheme: () => void
}

// 2. Créer la clé d'injection typée
export const ThemeKey: InjectionKey<ThemeContext> = Symbol('Theme')
```

```vue
<!-- Provider (composant parent) -->
<script setup lang="ts">
import { ThemeKey } from '@/types/injection-keys'

const theme = ref<'light' | 'dark'>('light')

const toggleTheme = () => {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}

// ✅ readonly() empêche les enfants de muter directement
provide(ThemeKey, {
  theme: readonly(theme),
  toggleTheme
})
</script>
```

```vue
<!-- Consumer (composant enfant à n'importe quelle profondeur) -->
<script setup lang="ts">
import { ThemeKey } from '@/types/injection-keys'

// inject avec fallback undefined + guard
const themeContext = inject(ThemeKey)

if (!themeContext) {
  throw new Error('ThemeProvider not found in component tree')
}

// TypeScript sait que theme est Readonly<Ref<'light' | 'dark'>>
const { theme, toggleTheme } = themeContext
</script>
```

**Avantages :**
- **Type-safety** : TypeScript infère le type correct du contexte
- **Immutabilité** : `readonly()` empêche les mutations directes du state
- **Erreurs explicites** : Le guard force la gestion du cas "provider absent"
- **Refactoring safe** : Renommer le Symbol ne casse rien

---

## Helper injectStrict (Fail-Fast)

Pour éviter les guards manuels répétitifs, créez un helper réutilisable :

```typescript
// app/composables/useInjectStrict.ts
import type { InjectionKey } from 'vue'

export function injectStrict<T>(key: InjectionKey<T>, fallback?: T): T {
  const resolved = inject(key, fallback)
  if (resolved === undefined) {
    throw new Error(`Missing injection for key: ${String(key.description || key)}`)
  }
  return resolved
}
```

**Utilisation :**

```vue
<script setup lang="ts">
import { ThemeKey } from '@/types/injection-keys'

// ✅ Fail-fast : erreur immédiate si ThemeProvider absent
const { theme, toggleTheme } = injectStrict(ThemeKey)

// vs. le pattern manuel plus verbeux
const ctx = inject(ThemeKey)
if (!ctx) throw new Error('ThemeProvider not found')
</script>
```

---

## Decision Framework : Patterns de Composition

| Scénario | Pattern | Raison |
|----------|---------|--------|
| **Parent → enfant (1-2 niveaux)** | Props | API explicite, validation, traçabilité |
| **Hiérarchie profonde (3+ niveaux)** | Provide/Inject | Évite drilling, type-safe avec InjectionKey |
| **Compound component (Accordion, Dialog)** | Provide/Inject | Pattern Reka UI, state partagé implicite |
| **Logique réutilisable avec state** | Composable | Encapsule réactivité + lifecycle |
| **Zones de contenu flexibles** | Slots (scoped) | Parent contrôle le rendu |
| **State partagé SSR-safe** | `useState()` | Survit à l'hydratation, pas de pollution cross-request |
| **State global complexe** | Pinia | DevTools, persistance, SSR auto (non utilisé dans ce projet) |

---

**Hydratation Lazy (Nuxt 4 natif):**

Les directives d'hydratation lazy (Nuxt 3.16+/4.x) s'appuient sur les API natives de Vue 3.5+. Elles réduisent le TTI et TBT en différant l'exécution JavaScript.

```vue
<template>
  <!-- hydrate-on-visible : avec marge de préchargement -->
  <LazyTableOfContents hydrate-on-visible />
  <LazyCommentSection :hydrate-on-visible="{ rootMargin: '100px' }" />

  <!-- hydrate-on-idle : avec timeout maximum (ms) -->
  <LazySearchCommand hydrate-on-idle />
  <LazyAnalyticsWidget :hydrate-on-idle="2000" />

  <!-- hydrate-on-interaction : événements personnalisés -->
  <LazyArticleComments hydrate-on-interaction />
  <LazySearchForm hydrate-on-interaction="focus" />
  <LazyMobileMenu :hydrate-on-interaction="['click', 'touchstart']" />

  <!-- hydrate-after : délai fixe en ms -->
  <LazyNewsletterForm :hydrate-after="2000" />

  <!-- hydrate-on-media-query : selon taille d'écran -->
  <LazyMobileNav hydrate-on-media-query="(max-width: 768px)" />

  <!-- hydrate-when : condition booléenne -->
  <LazyPremiumFeature :hydrate-when="userAuthenticated" />

  <!-- hydrate-never : HTML statique, jamais hydraté -->
  <LazyStaticContent hydrate-never />
</template>
```

**Recommandations par cas d'usage:**

| Composant | Stratégie recommandée | Raison |
|-----------|----------------------|--------|
| Below-the-fold (commentaires, footer) | `hydrate-on-visible` avec `rootMargin` | Anticipe l'hydratation |
| Widgets non critiques (analytics, carrousels) | `hydrate-on-idle` avec timeout | Utilise les temps morts |
| Menus, modals, formulaires | `hydrate-on-interaction` | Hydrate à la demande |
| Composants mobiles uniquement | `hydrate-on-media-query` | Économise JS desktop |
| Contenu purement textuel | `hydrate-never` | Zéro JS client |

**Migration Reka UI (depuis Radix Vue):**
```typescript
// ❌ Ancien import (legacy, ne plus utiliser)
import { TooltipRoot, TooltipTrigger } from 'radix-vue'

// ✅ Nouveau import (recommandé depuis février 2025)
import { TooltipRoot, TooltipTrigger } from 'reka-ui'
```

```css
/* Variables CSS également renommées */
/* ❌ Ancien */ .element { color: var(--radix-tooltip-trigger-color); }
/* ✅ Nouveau */ .element { color: var(--reka-tooltip-trigger-color); }
```
