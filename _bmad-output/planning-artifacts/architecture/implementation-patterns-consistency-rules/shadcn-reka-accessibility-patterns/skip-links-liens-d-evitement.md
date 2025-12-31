# Skip Links (Liens d'évitement)

WCAG 2.4.1 (Bypass Blocks) exige un moyen de sauter directement au contenu principal, évitant de traverser la navigation à chaque page.

## Implémentation complète

```vue
<!-- components/SkipLinks.vue -->
<script setup lang="ts">
interface SkipLink {
  label: string
  targetId: string
}

const links: SkipLink[] = [
  { label: 'Aller au contenu principal', targetId: 'main-content' },
  { label: 'Aller à la navigation', targetId: 'main-nav' },
]

function skipTo(targetId: string) {
  const target = document.getElementById(targetId)
  if (target) {
    if (!target.hasAttribute('tabindex')) {
      target.setAttribute('tabindex', '-1')
    }
    target.focus()
    target.scrollIntoView({ behavior: 'smooth' })
  }
}
</script>

<template>
  <nav aria-label="Liens d'évitement">
    <a
      v-for="link in links"
      :key="link.targetId"
      :href="`#${link.targetId}`"
      class="
        sr-only focus:not-sr-only
        focus:fixed focus:top-4 focus:left-4 focus:z-50
        focus:px-4 focus:py-3 focus:rounded-lg
        focus:bg-primary focus:text-primary-foreground
        focus-visible:outline-2 focus-visible:outline-offset-2
        motion-safe:transition-all
      "
      @click.prevent="skipTo(link.targetId)"
    >
      {{ link.label }}
    </a>
  </nav>
</template>
```

## Positionnement dans le layout

```vue
<!-- layouts/default.vue -->
<template>
  <!-- Skip links EN PREMIER dans le DOM -->
  <SkipLinks />

  <header>
    <nav id="main-nav" aria-label="Navigation principale">
      <MainNavigation />
    </nav>
  </header>

  <main id="main-content" tabindex="-1">
    <slot />
  </main>
</template>
```

---
