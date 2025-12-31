# Layout Accessible Complet (Référence)

Exemple de layout intégrant tous les patterns d'accessibilité :

```vue
<!-- layouts/default.vue -->
<script setup lang="ts">
const route = useRoute()
const pageTitle = ref<HTMLHeadingElement | null>(null)

// Focus reset après navigation client-side
watch(
  () => route.path,
  async () => {
    await nextTick()
    pageTitle.value?.focus()
  }
)
</script>

<template>
  <!-- Skip links - PREMIER dans le DOM -->
  <SkipLinks />

  <header class="sticky top-0 z-40 bg-background border-b">
    <nav
      id="main-nav"
      aria-label="Navigation principale"
      class="container mx-auto px-4"
    >
      <MainNavigation />
    </nav>
  </header>

  <main
    id="main-content"
    tabindex="-1"
    class="container mx-auto px-4 py-8 min-h-screen"
  >
    <h1
      ref="pageTitle"
      tabindex="-1"
      class="text-3xl font-bold mb-6 outline-hidden"
    >
      {{ route.meta.title || 'Page' }}
    </h1>

    <slot />
  </main>

  <footer class="bg-muted py-8">
    <div class="container mx-auto px-4">
      <FooterContent />
    </div>
  </footer>
</template>

<style>
/* WCAG 2.4.11 - Focus Not Obscured */
html {
  scroll-padding-top: 80px;
  scroll-padding-bottom: 40px;
}

/* WCAG 2.5.8 - Target Size Minimum */
button:not(.inline-btn),
a:not(.inline-link),
[role="button"] {
  min-width: 24px;
  min-height: 24px;
}
</style>
```

## Checklist du layout

| Critère | Implémentation |
|---------|----------------|
| Skip links premier élément | `<SkipLinks />` en tête du template |
| Landmarks sémantiques | `<header>`, `<main>`, `<footer>` avec roles |
| Focus reset navigation | `watch(route.path)` + `nextTick` + `focus()` |
| Cible skip link focusable | `<main tabindex="-1">` |
| Focus Not Obscured | `scroll-padding-top` |
| Target Size | `min-width/height: 24px` global |

---
