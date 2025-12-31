# Gestion de la Touche Escape

## Comportement par défaut Reka UI

Tous les composants overlay ferment automatiquement avec Escape et émettent `@escape-key-down` pour personnalisation.

```vue
<template>
  <!-- Comportement par défaut : Escape ferme -->
  <DialogContent>
    <!-- Rien à faire -->
  </DialogContent>

  <!-- Fermeture conditionnelle (formulaire non sauvegardé) -->
  <DialogContent @escape-key-down="handleEscape">
    <!-- Escape ferme sauf si données non sauvegardées -->
  </DialogContent>

  <!-- Escape désactivé (fournir bouton de fermeture visible!) -->
  <DialogContent @escape-key-down.prevent>
    <DialogClose as-child>
      <Button>Fermer</Button> <!-- OBLIGATOIRE si Escape désactivé -->
    </DialogClose>
  </DialogContent>
</template>

<script setup lang="ts">
const hasUnsavedChanges = ref(false)

function handleEscape(event: KeyboardEvent) {
  if (hasUnsavedChanges.value) {
    event.preventDefault()
    // Afficher confirmation avant fermeture
  }
}
</script>
```

## ⚠️ Anti-pattern : Escape désactivé sans alternative

```vue
<!-- ❌ Utilisateur piégé dans la modale -->
<DialogContent @escape-key-down.prevent>
  <!-- Pas de bouton de fermeture visible -->
</DialogContent>
```

---
