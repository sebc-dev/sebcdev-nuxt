# Focus Management

## Focus Trapping (Dialog, AlertDialog, Sheet)

Le focus est automatiquement piégé quand `modal=true` (défaut). À la fermeture, le focus retourne au trigger.

```vue
<Dialog>
  <DialogTrigger as-child>
    <Button>Ouvrir</Button>  <!-- Focus retourne ici à la fermeture -->
  </DialogTrigger>
  <DialogContent>
    <!-- Focus piégé dans ce conteneur -->
    <input autofocus />  <!-- Premier élément focusable -->
  </DialogContent>
</Dialog>
```

## Roving Tabindex (Tabs, RadioGroup, Toolbar)

Un seul élément du groupe est dans le tab order. Les flèches naviguent entre éléments.

```vue
<!-- Tab order: [autres éléments] → [tab actif] → [autres éléments] -->
<Tabs default-value="tab1">
  <TabsList>
    <TabsTrigger value="tab1">Un</TabsTrigger>   <!-- tabindex="0" si actif -->
    <TabsTrigger value="tab2">Deux</TabsTrigger> <!-- tabindex="-1" si inactif -->
  </TabsList>
</Tabs>
```

## RovingFocusGroup pour widgets custom

Reka UI expose `RovingFocusGroup` pour créer des toolbars accessibles :

```vue
<script setup lang="ts">
import { RovingFocusGroup, RovingFocusItem } from 'reka-ui'

const tools = [
  { id: 'bold', label: 'Gras', icon: Bold },
  { id: 'italic', label: 'Italique', icon: Italic },
  { id: 'underline', label: 'Souligné', icon: Underline },
]
</script>

<template>
  <RovingFocusGroup
    orientation="horizontal"
    :loop="true"
    as="div"
    role="toolbar"
    aria-label="Outils de formatage"
  >
    <RovingFocusItem
      v-for="tool in tools"
      :key="tool.id"
      as-child
    >
      <button
        :aria-label="tool.label"
        class="p-2 hover:bg-accent focus-visible:outline-2"
      >
        <component :is="tool.icon" class="w-4 h-4" />
      </button>
    </RovingFocusItem>
  </RovingFocusGroup>
</template>
```

| Prop | Valeur | Description |
|------|--------|-------------|
| `orientation` | `"horizontal"` / `"vertical"` | Direction des flèches |
| `loop` | `true` / `false` | Boucle au dernier/premier élément |

---
