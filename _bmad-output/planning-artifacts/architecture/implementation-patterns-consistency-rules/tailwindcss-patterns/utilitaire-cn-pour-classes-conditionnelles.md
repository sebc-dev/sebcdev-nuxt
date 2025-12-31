# Utilitaire cn() pour Classes Conditionnelles

## Installation

```bash
pnpm add clsx tailwind-merge
```

## Implémentation

L'utilitaire `cn()` combine `clsx` (classes conditionnelles) et `tailwind-merge` (résolution de conflits Tailwind) :

```typescript
// app/lib/utils.ts
import type { ClassValue } from 'clsx'
import { clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

## Pourquoi cn() est essentiel

| Problème | Sans cn() | Avec cn() |
|----------|-----------|-----------|
| Classes conflictuelles | `p-4 p-2` → applique les deux | `p-4 p-2` → `p-2` (dernière gagne) |
| Classes conditionnelles | Template complexe | Syntaxe claire |
| Props de composants | Écrasement imprévisible | Merge intelligent |

## Usage dans les composants

```vue
<script setup lang="ts">
import { cn } from '@/lib/utils'

const props = defineProps<{
  class?: string
  variant?: 'default' | 'destructive'
}>()
</script>

<template>
  <button
    :class="cn(
      'inline-flex items-center justify-center rounded-md',
      variant === 'destructive' && 'bg-destructive text-destructive-foreground',
      variant === 'default' && 'bg-primary text-primary-foreground',
      props.class  // ← Permet l'override par le parent
    )"
  >
    <slot />
  </button>
</template>
```

## Exemples de résolution de conflits

```typescript
// Conflits de padding
cn('p-4', 'p-2')                    // → 'p-2'
cn('px-4 py-2', 'p-3')              // → 'p-3'

// Conflits de couleurs
cn('text-red-500', 'text-blue-500') // → 'text-blue-500'

// Classes conditionnelles
cn('base', isActive && 'active')    // → 'base active' ou 'base'
cn('base', { active: isActive })    // → même résultat

// Arrays
cn(['flex', 'items-center'], gap && 'gap-4')
```

## Pattern shadcn-vue avec CVA

```typescript
// app/components/ui/button/index.ts
import { cva, type VariantProps } from 'class-variance-authority'

export const buttonVariants = cva(
  'inline-flex items-center justify-center gap-2 rounded-md text-sm font-medium',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground',
        outline: 'border border-input bg-background hover:bg-accent',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 px-3',
        lg: 'h-11 px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: { variant: 'default', size: 'default' },
  }
)

export type ButtonVariants = VariantProps<typeof buttonVariants>
```

```vue
<!-- app/components/ui/button/Button.vue -->
<script setup lang="ts">
import { cn } from '@/lib/utils'
import { buttonVariants, type ButtonVariants } from '.'

interface Props extends ButtonVariants {
  class?: string
}

const props = withDefaults(defineProps<Props>(), {
  variant: 'default',
  size: 'default',
})
</script>

<template>
  <button :class="cn(buttonVariants({ variant, size }), props.class)">
    <slot />
  </button>
</template>
```

---
