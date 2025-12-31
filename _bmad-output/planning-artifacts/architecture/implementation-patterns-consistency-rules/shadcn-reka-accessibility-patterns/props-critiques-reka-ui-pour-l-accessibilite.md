# Props Critiques Reka UI pour l'Accessibilité

## Dialog

| Prop / Event | Valeur | Usage accessibilité |
|--------------|--------|---------------------|
| `modal` | `true` (défaut) | Active focus trap, `aria-hidden` sur contenu externe |
| `@open-auto-focus` | `(e) => e.preventDefault()` | Personnaliser focus initial |
| `@close-auto-focus` | `(e) => e.preventDefault()` | Personnaliser focus au retour |

```vue
<DialogContent
  @open-auto-focus="(e) => {
    e.preventDefault()
    emailInput?.focus()
  }"
>
```

## Tabs

| Prop | Valeur | Usage accessibilité |
|------|--------|---------------------|
| `activationMode="automatic"` | Défaut | Active tab au focus (flèches) |
| `activationMode="manual"` | | Requiert Enter/Space - **préférer pour formulaires longs** |
| `orientation` | `horizontal` / `vertical` | Change navigation ←→ vs ↑↓ |
| `unmountOnHide` | `true` (défaut) | Démonte contenu caché pour performances |

## Accordion

| Prop | Valeur | Usage accessibilité |
|------|--------|---------------------|
| `type="single"` | | Un seul item ouvert |
| `type="multiple"` | | Plusieurs items ouverts |
| `collapsible` | `true` | Permet de tout fermer (avec `type="single"`) |

---
