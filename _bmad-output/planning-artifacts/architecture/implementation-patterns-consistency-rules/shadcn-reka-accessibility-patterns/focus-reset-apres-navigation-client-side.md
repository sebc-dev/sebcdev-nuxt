# Focus Reset après Navigation Client-Side

En SSG avec hydratation, le changement de route ne déclenche pas d'annonce aux lecteurs d'écran. Le focus doit être explicitement déplacé.

```vue
<!-- layouts/default.vue -->
<script setup lang="ts">
const route = useRoute()
const pageTitle = ref<HTMLHeadingElement | null>(null)

// Déplacer le focus après chaque navigation
watch(
  () => route.path,
  async () => {
    await nextTick()
    pageTitle.value?.focus()
  }
)
</script>

<template>
  <SkipLinks />
  <TheHeader />

  <main id="main-content" tabindex="-1">
    <h1 ref="pageTitle" tabindex="-1" class="outline-hidden">
      {{ route.meta.title || 'Page' }}
    </h1>
    <slot />
  </main>
</template>
```

**Note** : Le `tabindex="-1"` sur le `<h1>` permet le focus programmatique sans l'inclure dans l'ordre de tabulation normal. `outline-hidden` évite l'outline au focus programmatique.

---
