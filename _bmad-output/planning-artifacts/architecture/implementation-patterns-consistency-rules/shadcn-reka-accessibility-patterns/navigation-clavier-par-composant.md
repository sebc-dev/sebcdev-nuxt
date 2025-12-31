# Navigation Clavier par Composant

## Tableau récapitulatif

| Composant | Navigation | Actions | Notes |
|-----------|------------|---------|-------|
| **Dialog** | `Tab` entre éléments | `Escape` ferme | Focus trap automatique |
| **Tabs** | `←` `→` (horizontal) / `↑` `↓` (vertical) | `Home`/`End` premier/dernier | `Enter`/`Space` si `activation-mode="manual"` |
| **Accordion** | `↑` `↓` entre triggers | `Space`/`Enter` toggle | Roving tabindex auto |
| **DropdownMenu** | `↑` `↓` entre items | Typeahead par lettre | `→` `←` sous-menus |
| **Select** | `↑` `↓` dans la liste | `Enter` sélectionne | Typeahead supporté |
| **Tooltip** | N/A (hover/focus) | `Escape` ferme | Délai configurable |

## Attributs ARIA automatiques

Reka UI injecte automatiquement les attributs appropriés :

| Composant | Attributs |
|-----------|-----------|
| **Dialog** | `role="dialog"`, `aria-modal="true"`, `aria-labelledby`, `aria-describedby` |
| **Tabs** | `role="tablist"`, `role="tab"`, `role="tabpanel"`, `aria-selected` |
| **Accordion** | `role="region"`, `aria-expanded`, `aria-controls` |
| **DropdownMenu** | `role="menu"`, `role="menuitem"`, `aria-haspopup` |
