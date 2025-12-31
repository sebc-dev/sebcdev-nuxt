# Story 1.3: Responsive Layout System

Status: ready-for-dev

## Story

As a visiteur,
I want the site to adapt to my device (mobile, tablet, desktop),
So that I can read articles comfortably on any screen.

## Acceptance Criteria

1. **AC1 - Desktop layout** : Sur desktop (≥ 1024px), le layout utilise un container max-width (1200px) centré avec le contenu à 720px
2. **AC2 - Tablet layout** : Sur tablette (768-1023px), le layout s'adapte avec des marges réduites
3. **AC3 - Mobile layout** : Sur mobile (< 768px), le layout est en pleine largeur avec padding approprié
4. **AC4 - Touch targets** : Les éléments interactifs ont une taille ≥ 44px sur mobile (48px recommandé)
5. **AC5 - Layout article** : Le layout article.vue est créé avec structure sidebar (future ToC) + content
6. **AC6 - Default layout** : Le layout default.vue est créé avec header + main + footer structure

## Tasks / Subtasks

- [ ] **Task 1: Configuration des tokens de layout dans main.css** (AC: #1, #2, #3)
  - [ ] 1.1 Ajouter les tokens de spacing fluide avec clamp()
  - [ ] 1.2 Configurer les breakpoints sémantiques dans @theme
  - [ ] 1.3 Définir les containers (prose, wide, max)

- [ ] **Task 2: Création du layout default.vue** (AC: #6)
  - [ ] 2.1 Créer `app/layouts/default.vue`
  - [ ] 2.2 Structurer avec header slot (future), main (slot), footer slot (future)
  - [ ] 2.3 Appliquer les classes de container responsive

- [ ] **Task 3: Création du layout article.vue** (AC: #1, #5)
  - [ ] 3.1 Créer `app/layouts/article.vue`
  - [ ] 3.2 Structurer avec grid: content (720px) + sidebar placeholder (240px pour future ToC)
  - [ ] 3.3 Configurer le gap de 48px entre content et sidebar
  - [ ] 3.4 Implémenter le responsive: sidebar hidden sur mobile/tablet

- [ ] **Task 4: Mise à jour de la page article [slug].vue** (AC: #1, #2, #3)
  - [ ] 4.1 Appliquer `layout: 'article'` dans definePageMeta
  - [ ] 4.2 Structurer le contenu avec les classes prose responsive
  - [ ] 4.3 Ajouter les marges et paddings adaptatifs

- [ ] **Task 5: Composant Container.vue** (AC: #1, #2, #3)
  - [ ] 5.1 Créer `app/components/layout/Container.vue`
  - [ ] 5.2 Props: `size` ('prose' | 'wide' | 'full'), `as` (tag HTML)
  - [ ] 5.3 Implémenter les variantes de largeur responsive

- [ ] **Task 6: Accessibilité touch targets** (AC: #4)
  - [ ] 6.1 Créer des classes utilitaires pour touch targets dans @layer utilities
  - [ ] 6.2 Documenter l'usage min-h-11 (44px) / min-h-12 (48px)
  - [ ] 6.3 Appliquer sur les liens et boutons existants

- [ ] **Task 7: Validation responsive** (AC: #1-6)
  - [ ] 7.1 Tester sur viewport desktop (1440px, 1024px)
  - [ ] 7.2 Tester sur viewport tablet (768px, 900px)
  - [ ] 7.3 Tester sur viewport mobile (375px, 414px)
  - [ ] 7.4 Vérifier les touch targets avec DevTools (44px highlight)

## Dev Notes

### Prérequis des Stories 1.1 et 1.2

Cette story s'appuie sur:
- Projet Nuxt 4 fonctionnel (Story 1.1)
- TailwindCSS 4 configuré avec tokens oklch (Story 1.2)
- Page article [slug].vue affichant du contenu
- main.css avec @theme et @layer base configurés

### Tokens de Layout à ajouter dans main.css

```css
/* Ajouter dans :root */
:root {
  /* ... tokens existants ... */

  /* Layout Containers - UX Spec aligned */
  --content-max-width: 720px;      /* Prose optimal */
  --toc-width: 240px;              /* Sidebar ToC */
  --container-max: 1200px;         /* Container global */
  --container-wide: 1400px;        /* Container élargi */
  --layout-gap: 48px;              /* Gap content/sidebar */
  --header-height: 64px;           /* Header fixe */

  /* Spacing fluide - Page level */
  --spacing-page-x: clamp(1rem, 5vw, 6rem);
  --spacing-page-y: clamp(2rem, 6vw, 4rem);

  /* Spacing fluide - Section level */
  --spacing-section: clamp(3rem, 6vw, 6rem);
  --spacing-section-sm: clamp(2rem, 4vw, 4rem);

  /* Spacing fluide - Component level */
  --spacing-component: clamp(1rem, 2vw, 1.5rem);
}

/* Ajouter dans @theme inline */
@theme inline {
  /* ... tokens existants ... */

  /* Layout */
  --content-max-width: var(--content-max-width);
  --toc-width: var(--toc-width);
  --container-max: var(--container-max);
  --container-wide: var(--container-wide);
  --layout-gap: var(--layout-gap);
  --header-height: var(--header-height);

  /* Spacing fluide */
  --spacing-page-x: var(--spacing-page-x);
  --spacing-page-y: var(--spacing-page-y);
  --spacing-section: var(--spacing-section);
  --spacing-section-sm: var(--spacing-section-sm);
  --spacing-component: var(--spacing-component);
}
```

### Layout default.vue

```vue
<!-- app/layouts/default.vue -->
<template>
  <div class="min-h-screen flex flex-col bg-background text-foreground">
    <!-- Header placeholder (sera implémenté Epic 3) -->
    <header class="h-(--header-height) border-b border-border">
      <div class="mx-auto max-w-(--container-max) h-full px-(--spacing-page-x) flex items-center">
        <span class="text-lg font-semibold">sebc.dev</span>
      </div>
    </header>

    <!-- Main Content -->
    <main class="flex-1">
      <slot />
    </main>

    <!-- Footer placeholder (sera implémenté plus tard) -->
    <footer class="border-t border-border py-8">
      <div class="mx-auto max-w-(--container-max) px-(--spacing-page-x) text-center text-foreground-muted">
        <p>&copy; 2025 sebc.dev</p>
      </div>
    </footer>
  </div>
</template>
```

### Layout article.vue

```vue
<!-- app/layouts/article.vue -->
<template>
  <div class="min-h-screen flex flex-col bg-background text-foreground">
    <!-- Header placeholder -->
    <header class="h-(--header-height) border-b border-border sticky top-0 z-50 bg-background/95 backdrop-blur">
      <div class="mx-auto max-w-(--container-max) h-full px-(--spacing-page-x) flex items-center">
        <NuxtLink to="/" class="text-lg font-semibold hover:text-accent transition-colors">
          sebc.dev
        </NuxtLink>
      </div>
    </header>

    <!-- Main Content avec grid pour sidebar -->
    <main class="flex-1 py-(--spacing-section-sm)">
      <div class="mx-auto max-w-(--container-max) px-(--spacing-page-x)">
        <!-- Grid: Content + Sidebar (ToC future) -->
        <div class="grid grid-cols-1 lg:grid-cols-[1fr_var(--toc-width)] gap-(--layout-gap)">
          <!-- Article Content -->
          <article class="max-w-(--content-max-width)">
            <slot />
          </article>

          <!-- Sidebar placeholder pour ToC (Story 2.3) -->
          <aside class="hidden lg:block">
            <div class="sticky top-[calc(var(--header-height)+2rem)]">
              <!-- TableOfContents sera inséré ici -->
              <div class="text-sm text-foreground-muted p-4 border border-border rounded-md">
                Table of Contents (Story 2.3)
              </div>
            </div>
          </aside>
        </div>
      </div>
    </main>

    <!-- Footer -->
    <footer class="border-t border-border py-8">
      <div class="mx-auto max-w-(--container-max) px-(--spacing-page-x) text-center text-foreground-muted">
        <p>&copy; 2025 sebc.dev</p>
      </div>
    </footer>
  </div>
</template>
```

### Page [slug].vue mise à jour

```vue
<!-- app/pages/articles/[slug].vue -->
<script setup lang="ts">
definePageMeta({
  layout: 'article'
})

const route = useRoute()
const slug = route.params.slug as string

const { data: article } = await useAsyncData(
  `article-${slug}`,
  () => queryCollection('articles_fr')
    .where('slug', '=', slug)
    .first()
)

if (!article.value) {
  throw createError({ statusCode: 404, message: 'Article not found' })
}
</script>

<template>
  <div v-if="article" class="prose-container">
    <!-- Article Header -->
    <header class="mb-(--spacing-section-sm)">
      <h1 class="text-4xl lg:text-5xl font-bold leading-tight mb-4">
        {{ article.title }}
      </h1>
      <p class="text-lg text-foreground-muted">
        {{ article.description }}
      </p>
    </header>

    <!-- Article Content -->
    <div class="prose prose-invert max-w-none">
      <ContentRenderer :value="article" />
    </div>
  </div>
</template>

<style scoped>
.prose-container {
  /* Styles spécifiques article si nécessaire */
}

/* Prose responsive */
.prose {
  @apply text-base lg:text-lg;
}

.prose :where(h2) {
  @apply text-2xl lg:text-3xl font-bold mt-(--spacing-section-sm) mb-4;
}

.prose :where(h3) {
  @apply text-xl lg:text-2xl font-semibold mt-8 mb-3;
}

.prose :where(p) {
  @apply mb-4 leading-relaxed;
}
</style>
```

### Container.vue Component

```vue
<!-- app/components/layout/Container.vue -->
<script setup lang="ts">
interface Props {
  size?: 'prose' | 'wide' | 'full'
  as?: string
}

const props = withDefaults(defineProps<Props>(), {
  size: 'wide',
  as: 'div'
})

const sizeClasses = {
  prose: 'max-w-(--content-max-width)',
  wide: 'max-w-(--container-max)',
  full: 'max-w-(--container-wide)'
}
</script>

<template>
  <component
    :is="as"
    :class="[
      'mx-auto px-(--spacing-page-x)',
      sizeClasses[size]
    ]"
  >
    <slot />
  </component>
</template>
```

### Touch Targets Utilities

```css
/* Ajouter dans @layer utilities de main.css */
@layer utilities {
  /* Touch targets accessibles (WCAG 2.5.5 Level AAA) */
  .touch-target {
    min-height: 44px;
    min-width: 44px;
  }

  .touch-target-lg {
    min-height: 48px;
    min-width: 48px;
  }

  /* Espacement entre touch targets adjacents */
  .touch-spacing {
    gap: 8px;
  }
}
```

### Breakpoints Reference

| Breakpoint | TailwindCSS | Largeur | Usage |
|------------|-------------|---------|-------|
| Default | - | < 640px | Mobile portrait |
| `sm:` | 640px | 640-767px | Mobile landscape |
| `md:` | 768px | 768-1023px | Tablet |
| `lg:` | 1024px | 1024-1279px | Desktop |
| `xl:` | 1280px | 1280-1535px | Large desktop |
| `2xl:` | 1536px | ≥ 1536px | Wide desktop |

### Container Queries (Alternative pour composants)

Pour les composants réutilisables, utiliser les container queries au lieu des media queries:

```html
<!-- Container parent -->
<div class="@container">
  <!-- Enfant qui réagit au container -->
  <div class="flex flex-col @md:flex-row @lg:grid-cols-3">
    <!-- Contenu adaptatif au container, pas au viewport -->
  </div>
</div>
```

### Spacing Fluide avec clamp()

Le système utilise `clamp()` pour un responsive sans media queries:

```css
/* Structure: clamp(minimum, preferred, maximum) */
--spacing-page-x: clamp(1rem, 5vw, 6rem);
/*                    ↑        ↑       ↑
                   mobile   fluid   desktop */
```

| Token | Mobile | Desktop | Description |
|-------|--------|---------|-------------|
| `--spacing-page-x` | 16px | 96px | Marges horizontales page |
| `--spacing-page-y` | 32px | 64px | Marges verticales page |
| `--spacing-section` | 48px | 96px | Entre sections |
| `--spacing-component` | 16px | 24px | Padding composants |

### Anti-patterns à éviter

- ❌ Ne PAS hardcoder les valeurs de breakpoints (utiliser tokens)
- ❌ Ne PAS oublier les touch targets sur mobile (min 44px)
- ❌ Ne PAS utiliser de media queries pour les composants - utiliser container queries
- ❌ Ne PAS mélanger rem et px dans les tokens de layout
- ❌ Ne PAS oublier `max-w-none` sur les éléments prose pour autoriser la pleine largeur

### Apprentissages des Stories précédentes

À appliquer:
- main.css avec @theme inline fonctionne (Story 1.2)
- Classes Tailwind 4 avec var() CSS: `max-w-(--token-name)`
- Les layouts Nuxt doivent contenir un `<slot />`

### References

- [Source: ux-design-specification/visual-design-foundation.md] - Spacing, breakpoints, layout dimensions
- [Source: ux-design-specification/component-strategy.md] - Structure composants
- [Source: architecture/project-structure-boundaries.md] - Boundaries composants, layouts
- [Source: architecture/implementation-patterns-consistency-rules/tailwindcss-patterns/spacing-semantique-fluide.md] - Spacing clamp()
- [Source: architecture/implementation-patterns-consistency-rules/tailwindcss-patterns/container-queries-natifs.md] - Container queries
- [Source: prd/13-functional-requirements-mvp.md#FR47-49] - Requirements FR47, FR48, FR49

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

### File List
