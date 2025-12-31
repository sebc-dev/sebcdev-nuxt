# Composant Icon SVG Accessible

Pattern pour icônes décoratives vs informatives :

```vue
<!-- components/Icon.vue -->
<script setup lang="ts">
interface Props {
  name: string
  label?: string
  decorative?: boolean
  size?: 'sm' | 'md' | 'lg'
}

const props = withDefaults(defineProps<Props>(), {
  decorative: true,
  size: 'md'
})

const sizeClasses = {
  sm: 'w-4 h-4',
  md: 'w-5 h-5',
  lg: 'w-6 h-6'
}
</script>

<template>
  <!-- SVG décoratif (accompagne du texte) -->
  <svg
    v-if="decorative"
    aria-hidden="true"
    focusable="false"
    :class="sizeClasses[size]"
  >
    <use :href="`/icons.svg#${name}`" />
  </svg>

  <!-- SVG informatif (porteur de sens) -->
  <svg
    v-else
    role="img"
    :aria-label="label"
    focusable="false"
    :class="sizeClasses[size]"
  >
    <use :href="`/icons.svg#${name}`" />
  </svg>
</template>
```

| Type | Attributs | Quand utiliser |
|------|-----------|----------------|
| Décoratif | `aria-hidden="true"` `focusable="false"` | Icône accompagnant du texte visible |
| Informatif | `role="img"` `aria-label="..."` | Icône seule porteuse de sens |

**Note** : `focusable="false"` prévient les problèmes de navigation clavier dans IE/Edge legacy.

---
