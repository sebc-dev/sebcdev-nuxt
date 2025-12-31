# Wrapper pattern shadcn-vue

Les composants shadcn-vue doivent être wrappés pour l'usage MDC :

```vue
<!-- app/components/content/MdcAlert.vue -->
<script setup lang="ts">
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'

interface Props {
  variant?: 'default' | 'destructive'
  title?: string
}

withDefaults(defineProps<Props>(), {
  variant: 'default'
})
</script>

<template>
  <Alert :variant="variant">
    <AlertTitle v-if="title">{{ title }}</AlertTitle>
    <AlertDescription>
      <slot mdc-unwrap="p" />
    </AlertDescription>
  </Alert>
</template>
```

**Usage MDC :**

```markdown
::mdc-alert{title="Note importante" variant="destructive"}
Contenu avec accessibilité WAI-ARIA intégrée via Reka UI.
::
```

## Pattern Card avec slots multiples

```vue
<!-- app/components/content/MdcCard.vue -->
<script setup lang="ts">
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'

interface Props {
  class?: string
}

defineProps<Props>()
</script>

<template>
  <Card :class="$props.class">
    <CardHeader v-if="$slots.title || $slots.description">
      <CardTitle v-if="$slots.title">
        <slot name="title" mdc-unwrap="p" />
      </CardTitle>
      <CardDescription v-if="$slots.description">
        <slot name="description" mdc-unwrap="p" />
      </CardDescription>
    </CardHeader>
    <CardContent>
      <slot />
    </CardContent>
  </Card>
</template>
```

**Usage MDC :**

```markdown
::mdc-card{.max-w-md}
#title
Titre de la carte

#description
Description courte

Contenu principal avec **formatage Markdown**.
::
```
