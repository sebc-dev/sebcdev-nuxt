# Story 2.4: Reading Progress Indicator

Status: ready-for-dev

## Story

As a visiteur,
I want to see my reading progress as I scroll,
So that I know how much of the article remains.

## Acceptance Criteria

1. **AC1 - Barre visible** : Une barre de progression horizontale est visible en haut de la page pendant la lecture
2. **AC2 - Progression 0-100%** : La barre progresse de 0% (haut de l'article) à 100% (fin de l'article)
3. **AC3 - Style** : La barre fait 3px de hauteur avec la couleur accent (`#14B8A6`)
4. **AC4 - Pages article uniquement** : La barre est visible uniquement sur les pages article (pas sur homepage, etc.)
5. **AC5 - Performance** : Le calcul utilise `requestAnimationFrame` ou throttling pour éviter les jank
6. **AC6 - Reduced motion** : Respecter `prefers-reduced-motion` (pas d'animation si activé)

## Tasks / Subtasks

- [ ] **Task 1: Créer le composant ReadingProgress.vue** (AC: #1-3, #5-6)
  - [ ] 1.1 Créer `app/components/content/ReadingProgress.vue`
  - [ ] 1.2 Implémenter le calcul du pourcentage de scroll
  - [ ] 1.3 Utiliser `position: fixed; top: 0` avec z-index élevé
  - [ ] 1.4 Appliquer la couleur accent et hauteur 3px
  - [ ] 1.5 Ajouter throttling ou requestAnimationFrame

- [ ] **Task 2: Créer le composable useReadingProgress** (AC: #2, #5)
  - [ ] 2.1 Créer `app/composables/useReadingProgress.ts`
  - [ ] 2.2 Calculer: `(scrollY / (documentHeight - viewportHeight)) * 100`
  - [ ] 2.3 Retourner `progress` réactif (0-100)
  - [ ] 2.4 Gérer le cleanup du listener onUnmounted

- [ ] **Task 3: Intégrer dans le layout article** (AC: #4)
  - [ ] 3.1 Modifier `app/layouts/article.vue` (créé en story 2-3)
  - [ ] 3.2 Ajouter `<ReadingProgress />` en haut du layout
  - [ ] 3.3 S'assurer qu'il n'apparaît pas sur les autres layouts

- [ ] **Task 4: Gérer reduced motion** (AC: #6)
  - [ ] 4.1 Utiliser `@media (prefers-reduced-motion: reduce)`
  - [ ] 4.2 Désactiver les transitions CSS si reduced motion
  - [ ] 4.3 Alternative: cacher la barre complètement ou mise à jour instantanée

- [ ] **Task 5: Tests et validation** (AC: #1-6)
  - [ ] 5.1 Tester le scroll sur un article long
  - [ ] 5.2 Vérifier 0% au début, 100% à la fin
  - [ ] 5.3 Vérifier que la barre n'apparaît pas sur la homepage
  - [ ] 5.4 Tester les performances (pas de jank pendant le scroll)
  - [ ] 5.5 Tester avec reduced motion activé

## Dev Notes

### Critical Architecture Requirements

**Calcul du pourcentage:**
```typescript
// Formule correcte pour le scroll progress
const scrollY = window.scrollY
const documentHeight = document.documentElement.scrollHeight
const viewportHeight = window.innerHeight
const progress = (scrollY / (documentHeight - viewportHeight)) * 100
```

**Position et z-index:**
- `position: fixed` pour rester en haut pendant le scroll
- `top: 0` pour coller au bord supérieur
- `z-index: 50` (au-dessus du contenu, sous les modals)
- `left: 0; right: 0` pour la largeur complète

### Project Structure Notes

Fichiers à créer/modifier:

```
app/
├── composables/
│   └── useReadingProgress.ts    # NOUVEAU
├── components/
│   └── content/
│       └── ReadingProgress.vue  # NOUVEAU
├── layouts/
│   └── article.vue              # MODIFIER (ajouter ReadingProgress)
```

### Technical Implementation Details

**useReadingProgress.ts:**
```typescript
// app/composables/useReadingProgress.ts
export function useReadingProgress() {
  const progress = ref(0)
  let ticking = false

  function updateProgress() {
    const scrollY = window.scrollY
    const documentHeight = document.documentElement.scrollHeight
    const viewportHeight = window.innerHeight
    const maxScroll = documentHeight - viewportHeight

    if (maxScroll > 0) {
      progress.value = Math.min(100, Math.max(0, (scrollY / maxScroll) * 100))
    }
    ticking = false
  }

  function onScroll() {
    if (!ticking) {
      requestAnimationFrame(updateProgress)
      ticking = true
    }
  }

  onMounted(() => {
    window.addEventListener('scroll', onScroll, { passive: true })
    updateProgress() // Initial calculation
  })

  onUnmounted(() => {
    window.removeEventListener('scroll', onScroll)
  })

  return { progress }
}
```

**ReadingProgress.vue:**
```vue
<script setup lang="ts">
const { progress } = useReadingProgress()
</script>

<template>
  <div
    class="fixed top-0 left-0 right-0 h-[3px] z-50 pointer-events-none"
    role="progressbar"
    :aria-valuenow="Math.round(progress)"
    aria-valuemin="0"
    aria-valuemax="100"
    :aria-label="`Progression de lecture: ${Math.round(progress)}%`"
  >
    <div
      class="h-full bg-accent transition-[width] duration-150 ease-out motion-reduce:transition-none"
      :style="{ width: `${progress}%` }"
    />
  </div>
</template>

<style scoped>
/* Couleur accent */
.bg-accent {
  background-color: #14B8A6;
}

/* Alternative: utiliser CSS variable */
/* background-color: var(--accent); */
</style>
```

**Intégration dans article.vue:**
```vue
<script setup lang="ts">
import ReadingProgress from '~/components/content/ReadingProgress.vue'
</script>

<template>
  <div class="min-h-screen flex flex-col">
    <!-- Reading Progress Bar -->
    <ReadingProgress />

    <!-- Header (si présent dans ce layout) -->

    <main class="flex-1 container mx-auto px-4 py-8">
      <div class="lg:grid lg:grid-cols-[1fr_240px] lg:gap-12">
        <article class="max-w-[720px]">
          <slot />
        </article>

        <aside class="hidden lg:block">
          <div class="sticky top-24">
            <slot name="toc" />
          </div>
        </aside>
      </div>
    </main>
  </div>
</template>
```

### CSS Design Tokens

```css
/* Couleur accent pour la barre */
--accent: #14B8A6;

/* Hauteur de la barre */
height: 3px;

/* Z-index hierarchy */
z-index: 50; /* Above content, below modals (z-index: 100+) */
```

### Package Versions (Décembre 2025)

| Package | Version |
|---------|---------|
| nuxt | 4.2.2 |

### Dépendances de cette story

**Requiert (stories précédentes):**
- Story 2-3 : Layout `article.vue` créé

**Bloque (stories futures):**
- Aucune

### Anti-patterns à éviter

- ❌ Ne PAS utiliser `scroll` event sans throttling/rAF - cause des jank
- ❌ Ne PAS oublier `{ passive: true }` sur le scroll listener
- ❌ Ne PAS utiliser `document.body.scrollHeight` - pas fiable, utiliser `documentElement.scrollHeight`
- ❌ Ne PAS oublier le cleanup du listener dans onUnmounted
- ❌ Ne PAS mettre un z-index trop élevé (éviter 9999)

### Considérations accessibilité

- `role="progressbar"` pour la sémantique
- `aria-valuenow`, `aria-valuemin`, `aria-valuemax` pour les lecteurs d'écran
- `aria-label` descriptif en français
- `pointer-events: none` car non interactif
- Respecter `prefers-reduced-motion` avec `motion-reduce:transition-none`

### Considérations performance

- `requestAnimationFrame` pour synchroniser avec le refresh rate
- `{ passive: true }` pour ne pas bloquer le scroll
- Éviter les reflows - ne modifier que `width` via `style`
- `transform: scaleX()` serait encore plus performant (GPU) mais moins intuitif

**Alternative ultra-performante avec scaleX:**
```vue
<template>
  <div class="fixed top-0 left-0 right-0 h-[3px] z-50 bg-background-secondary">
    <div
      class="h-full bg-accent origin-left will-change-transform"
      :style="{ transform: `scaleX(${progress / 100})` }"
    />
  </div>
</template>
```

### Gestion du Header Fixe

Si le header est `position: fixed`, ajuster le `top` de la barre:

```vue
<!-- Si header de 64px -->
<div class="fixed top-16 left-0 right-0 h-[3px] z-50">
```

Ou le placer sous le header dans le DOM pour qu'il apparaisse naturellement en dessous.

### References

- [Source: epics/epic-2-rich-article-experience.md#Story 2.4] - Requirements originaux
- [Source: ux-design-specification/visual-design-foundation.md#Accent] - Couleur accent #14B8A6
- [Source: ux-design-specification/component-strategy.md#ReadingProgress] - Spécification composant
- [Source: ux-design-specification/visual-design-foundation.md#Accessibility] - prefers-reduced-motion

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

### File List
