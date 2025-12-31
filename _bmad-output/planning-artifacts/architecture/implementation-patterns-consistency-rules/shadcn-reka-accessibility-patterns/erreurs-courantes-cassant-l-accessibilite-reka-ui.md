# Erreurs Courantes Cassant l'Accessibilité Reka UI

| Erreur | Problème | Solution |
|--------|----------|----------|
| Omission de `DialogTitle` | Association ARIA supprimée, lecteurs d'écran perdus | Toujours inclure `DialogTitle` (visible ou `VisuallyHidden`) |
| Omission de `DialogDescription` | Manque de contexte pour les utilisateurs | Ajouter description ou `VisuallyHidden` si non nécessaire visuellement |
| `<div>` au lieu de `as-child` sur triggers | Élément non-interactif créé | Utiliser `as-child` avec `<Button>` |
| `@keydown.prevent` sans alternative | Navigation clavier cassée | Implémenter alternative ou ne pas bloquer |
| Modification DOM via CSS (`display: none`) | Éléments masqués aux lecteurs d'écran | Utiliser les props Reka UI (`open`, `hidden`) |
| IDs générés avec `Math.random()` | Erreurs hydration SSG | Utiliser `useId()` de Vue 3.5+ |
| Focus perdu après fermeture modale | Utilisateur désorienté | Laisser Reka UI gérer ou restaurer manuellement |

## Exemple Dialog correctement accessible

```vue
<template>
  <Dialog>
    <DialogTrigger as-child>
      <Button>Ouvrir</Button> <!-- ✅ as-child avec Button -->
    </DialogTrigger>

    <DialogContent>
      <DialogHeader>
        <DialogTitle>Confirmation</DialogTitle> <!-- ✅ Obligatoire -->
        <DialogDescription>
          Cette action est irréversible.
        </DialogDescription> <!-- ✅ Obligatoire -->
      </DialogHeader>

      <DialogFooter>
        <DialogClose as-child>
          <Button variant="outline">Annuler</Button>
        </DialogClose>
        <Button variant="destructive">Confirmer</Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>
```

---
