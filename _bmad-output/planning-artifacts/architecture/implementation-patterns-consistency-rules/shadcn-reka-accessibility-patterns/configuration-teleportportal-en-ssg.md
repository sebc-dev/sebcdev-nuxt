# Configuration Teleport/Portal en SSG

Reka UI utilise `<Teleport>` via `DialogPortal`, `DropdownMenuPortal`, etc. En SSG, la cible doit exister au moment du rendu.

## Prop `defer` (Vue 3.5+)

```vue
<!-- Attend que la cible soit montée avant de téléporter -->
<DialogPortal defer>
  <DialogContent>...</DialogContent>
</DialogPortal>
```

## Désactiver le Teleport si problématique

```vue
<!-- Rend le contenu in-place sans téléportation -->
<DialogPortal disabled>
  <DialogContent>...</DialogContent>
</DialogPortal>
```

| Prop | Comportement | Quand utiliser |
|------|--------------|----------------|
| `defer` | Attend que la cible existe | SSG avec timing complexe |
| `disabled` | Pas de téléportation | Debugging, cas spéciaux |
| (défaut) | Téléporte immédiatement | Cas standard CSR |

---
