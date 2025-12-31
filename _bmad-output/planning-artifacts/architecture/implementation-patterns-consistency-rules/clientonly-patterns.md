# ClientOnly Patterns

Le composant `<ClientOnly>` exclut son contenu du rendu SSR. **Le slot `#fallback` avec dimensions explicites est crucial pour éviter le CLS** (Cumulative Layout Shift).

**Pattern de base avec dimensions réservées:**

```vue
<template>
  <!-- ✅ CORRECT : fallback avec dimensions explicites -->
  <ClientOnly>
    <DisqusComments :identifier="postSlug" />
    <template #fallback>
      <div class="comments-placeholder" style="min-height: 450px;">
        <p>Chargement des commentaires...</p>
      </div>
    </template>
  </ClientOnly>

  <!-- Widget de partage social -->
  <ClientOnly>
    <ShareButtons :url="post.url" :title="post.title" />
    <template #fallback>
      <div class="share-skeleton" style="height: 48px;">
        Partager cet article
      </div>
    </template>
  </ClientOnly>

  <!-- Embed Twitter/X avec lien fallback -->
  <ClientOnly>
    <TwitterEmbed :tweet-id="tweetId" />
    <template #fallback>
      <a :href="tweetUrl">Voir le tweet</a>
    </template>
  </ClientOnly>
</template>
```

**Pattern Dark Mode Toggle :**

Le toggle de thème nécessite `<ClientOnly>` car `useColorMode()` n'est pas disponible côté serveur. Le fallback dimensionné évite le CLS.

```vue
<template>
  <ClientOnly>
    <button
      class="w-9 h-9 flex items-center justify-center rounded-md hover:bg-accent"
      @click="toggleTheme"
      :aria-label="colorMode.value === 'dark' ? 'Activer le mode clair' : 'Activer le mode sombre'"
    >
      <Sun v-if="colorMode.value === 'dark'" class="h-5 w-5" />
      <Moon v-else class="h-5 w-5" />
    </button>
    <template #fallback>
      <!-- Dimensions identiques au bouton pour éviter CLS -->
      <div class="w-9 h-9 rounded-md bg-muted animate-pulse" />
    </template>
  </ClientOnly>
</template>

<script setup>
import { Sun, Moon } from 'lucide-vue-next'

const colorMode = useColorMode()

function toggleTheme() {
  colorMode.preference = colorMode.value === 'dark' ? 'light' : 'dark'
}
</script>
```

**Pattern skeleton loader pour meilleure UX:**

```vue
<template>
  <ClientOnly>
    <TwitterTimeline />
    <template #fallback>
      <div class="skeleton-feed" style="aspect-ratio: 1/1.5;">
        <div v-for="i in 3" :key="i" class="skeleton-item">
          <div class="skeleton-avatar h-10 w-10 rounded-full bg-muted animate-pulse" />
          <div class="skeleton-text flex-1 space-y-2">
            <div class="h-4 bg-muted rounded animate-pulse" />
            <div class="h-4 bg-muted rounded w-3/4 animate-pulse" />
          </div>
        </div>
      </div>
    </template>
  </ClientOnly>
</template>
```

**Anti-patterns à éviter:**

```vue
<!-- ❌ INCORRECT : pas de fallback = CLS élevé -->
<ClientOnly>
  <HeavyWidget />
</ClientOnly>

<!-- ❌ INCORRECT : fallback sans dimensions -->
<ClientOnly>
  <DisqusComments />
  <template #fallback>
    <p>Chargement...</p>
  </template>
</ClientOnly>

<!-- ❌ INCORRECT : contenu SEO-critique dans ClientOnly -->
<ClientOnly>
  <ArticleContent :content="post.body" />
</ClientOnly>

<!-- ❌ INCORRECT : masquer erreurs d'hydratation -->
<ClientOnly>
  <!-- Au lieu de corriger l'erreur, on la cache -->
  <ProblematicComponent />
</ClientOnly>
```

**Alternative `.client.vue`:**

Le suffixe `.client.vue` rend automatiquement un composant client-only, mais **sans fallback natif**. Préférer `<ClientOnly>` avec fallback pour les widgets tiers volumineux.

## Anti-patterns Hydratation Vue/Nuxt

### Tableau des erreurs courantes

| Anti-pattern | Erreur | Solution |
|--------------|--------|----------|
| `window`/`document` au setup | `window is not defined` | `onMounted()` ou composables VueUse |
| IDs dynamiques `Math.random()` | Hydration mismatch | `useId()` de Vue 3.5+ |
| Event listeners sans cleanup | Memory leaks | `useEventListener` de VueUse |
| Dates formatées serveur≠client | Texte différent SSR/client | `<ClientOnly>` ou `useCookie` pour locale |

### Accès window/document

```typescript
// ❌ INCORRECT : erreur SSR
const width = window.innerWidth

// ✅ CORRECT : dans onMounted
onMounted(() => {
  const width = window.innerWidth
})

// ✅ CORRECT : avec VueUse (gère SSR automatiquement)
const { width } = useWindowSize()
```

### IDs dynamiques

```vue
<script setup>
// ❌ INCORRECT : ID différent serveur/client
const id = `input-${Math.random()}`

// ✅ CORRECT : useId() de Vue 3.5+
const id = useId()
</script>

<template>
  <label :for="id">Label</label>
  <input :id="id" />
</template>
```

### Event listeners

```typescript
// ❌ INCORRECT : pas de cleanup
onMounted(() => {
  window.addEventListener('resize', handleResize)
})
// Memory leak si composant unmount

// ✅ CORRECT : cleanup automatique avec VueUse
import { useEventListener } from '@vueuse/core'

useEventListener(window, 'resize', handleResize)
// Automatiquement nettoyé à l'unmount
```

### Dates et locale

```vue
<!-- ❌ INCORRECT : format peut différer serveur/client -->
<template>
  <time>{{ new Date().toLocaleDateString() }}</time>
</template>

<!-- ✅ CORRECT : encapsuler dans ClientOnly -->
<template>
  <ClientOnly>
    <time>{{ formattedDate }}</time>
    <template #fallback>
      <time>{{ isoDate }}</time>
    </template>
  </ClientOnly>
</template>

<script setup>
const isoDate = post.publishedAt.toISOString().split('T')[0]
const formattedDate = computed(() =>
  new Intl.DateTimeFormat('fr-FR').format(post.publishedAt)
)
</script>
```

### Détection erreurs hydratation

En développement, Vue affiche des warnings dans la console :

```
[Vue warn]: Hydration node mismatch
[Vue warn]: Hydration text content mismatch
```

**Stratégie de debug :**
1. Identifier le composant dans le warning
2. Chercher les accès `window`/`document` au setup
3. Vérifier les valeurs dynamiques (dates, IDs, Math.random)
4. Utiliser Vue DevTools pour comparer SSR vs client

---

## Plugin SSR Width pour Composants Responsive

Les composants qui utilisent `useWindowSize()` ou `useBreakpoints()` de VueUse peuvent causer des erreurs d'hydration car la largeur est inconnue côté serveur.

### Le problème

```vue
<script setup>
import { useWindowSize } from '@vueuse/core'

const { width } = useWindowSize()
// SSR: width = Infinity (pas de fenêtre)
// Client: width = 1920 (par exemple)
// → Hydration mismatch si le rendu dépend de width
</script>

<template>
  <!-- ❌ Erreur hydration : rendu différent SSR/client -->
  <MobileMenu v-if="width < 768" />
  <DesktopMenu v-else />
</template>
```

### Solution : Plugin provideSSRWidth

```typescript
// plugins/ssr-width.ts
import { provideSSRWidth } from '@vueuse/core'

export default defineNuxtPlugin((nuxtApp) => {
  // Assume une largeur desktop (1024px) pour le SSR
  // Le client corrigera immédiatement après hydration
  provideSSRWidth(1024, nuxtApp.vueApp)
})
```

### Quand utiliser

| Situation | Solution |
|-----------|----------|
| Menu responsive | Plugin SSR Width |
| Sidebar collapsible | Plugin SSR Width |
| Composant différent mobile/desktop | `<ClientOnly>` avec fallback |
| Affichage conditionnel critique SEO | Éviter responsive JS, utiliser CSS media queries |

### Pattern alternatif avec ClientOnly

```vue
<template>
  <!-- Fallback desktop pour le SSR, client adapte ensuite -->
  <ClientOnly>
    <component :is="isMobile ? MobileMenu : DesktopMenu" />
    <template #fallback>
      <DesktopMenu />
    </template>
  </ClientOnly>
</template>

<script setup>
import { useBreakpoints, breakpointsTailwind } from '@vueuse/core'

const breakpoints = useBreakpoints(breakpointsTailwind)
const isMobile = breakpoints.smaller('md')
</script>
```

### Recommandation

**Préférer CSS media queries** pour le responsive quand possible — pas d'erreur d'hydration et meilleure performance :

```vue
<template>
  <!-- ✅ CSS pur : pas de JS, pas d'erreur hydration -->
  <nav class="hidden md:flex"><!-- Desktop --></nav>
  <nav class="flex md:hidden"><!-- Mobile --></nav>
</template>
```
