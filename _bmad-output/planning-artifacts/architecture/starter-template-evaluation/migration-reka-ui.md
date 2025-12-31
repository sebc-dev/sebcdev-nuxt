# Migration Reka UI

```typescript
// Ancien import (legacy)
import { TooltipRoot, TooltipTrigger } from 'radix-vue'

// Nouveau import (recommandé depuis février 2025)
import { TooltipRoot, TooltipTrigger } from 'reka-ui'
```

```css
/* Variables CSS également renommées */
/* Ancien */
.element { color: var(--radix-tooltip-trigger-color); }

/* Nouveau */
.element { color: var(--reka-tooltip-trigger-color); }
```
