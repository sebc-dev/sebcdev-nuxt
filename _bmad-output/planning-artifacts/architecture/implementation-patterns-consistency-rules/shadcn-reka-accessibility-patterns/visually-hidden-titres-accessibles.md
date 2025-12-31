# VisuallyHidden - Titres Accessibles

Quand le design ne permet pas de titre visible, utiliser `VisuallyHidden` :

```vue
<script setup>
import { VisuallyHidden } from 'reka-ui'
</script>

<template>
  <Dialog>
    <DialogContent>
      <!-- Titre masqué visuellement mais lu par lecteurs d'écran -->
      <VisuallyHidden as-child>
        <DialogTitle>Confirmation de suppression</DialogTitle>
      </VisuallyHidden>

      <!-- Contenu visible -->
      <p>Voulez-vous vraiment supprimer cet élément ?</p>
      <DialogFooter>
        <Button variant="destructive">Supprimer</Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>
```

**Cas d'usage VisuallyHidden :**

| Situation | Solution |
|-----------|----------|
| Dialog sans titre visible | `<VisuallyHidden><DialogTitle>` |
| Icône seule comme bouton | `<VisuallyHidden>Label accessible</VisuallyHidden>` |
| Description technique | `<VisuallyHidden><DialogDescription>` |
