# Pattern 11 : Vue `defineProps` et limitations du compilateur SFC

Le compilateur Vue SFC ne peut pas résoudre les types complexes importés depuis des fichiers externes. Utiliser `z.infer<typeof Schema>` importé cause l'erreur : `[@vue/compiler-sfc] Unresolvable type reference`.

## ❌ Anti-pattern : Import de type inféré

```typescript
// schemas/article.ts
export const ArticlePropsSchema = z.object({
  title: z.string(),
  slug: z.string()
})
export type ArticleProps = z.infer<typeof ArticlePropsSchema>
```

```vue
<!-- components/ArticleCard.vue -->
<script setup lang="ts">
// ❌ ERREUR: [@vue/compiler-sfc] Unresolvable type reference
import type { ArticleProps } from '@/schemas/article'
const props = defineProps<ArticleProps>()
</script>
```

## ✅ Solution 1 : Type alias dans le même fichier

```vue
<script setup lang="ts">
import { z } from 'zod/v4'

// Schema et type DANS LE MÊME FICHIER
const PropsSchema = z.object({
  title: z.string(),
  slug: z.string(),
  publishedAt: z.string()  // ISO date string après parsing Content
})

type Props = z.infer<typeof PropsSchema>
const props = defineProps<Props>()
</script>
```

## ✅ Solution 2 : Interface explicite séparée du schema

```typescript
// lib/content-schemas.ts
import { z } from 'zod/v4'

export const ArticleSchema = z.object({
  title: z.string(),
  slug: z.string(),
  publishedAt: z.iso.date()
})

// Interface EXPLICITE (pas z.infer) - résolvable par le compilateur Vue
export interface ArticleProps {
  title: string
  slug: string
  publishedAt: string
}
```

```vue
<script setup lang="ts">
// ✅ Interface explicite fonctionne
import type { ArticleProps } from '@/lib/content-schemas'
const props = defineProps<ArticleProps>()
</script>
```

## Quand utiliser quelle solution

| Situation | Solution recommandée |
|-----------|---------------------|
| Props simples (2-3 champs) | Solution 1 : Inline |
| Props réutilisées dans plusieurs composants | Solution 2 : Interface explicite |
| Schema avec transforms complexes | Solution 2 : Interface reflétant l'output |
| Données venant de Nuxt Content | Solution 2 : Interface matchant le type queryCollection |

## Pattern pour props venant de Nuxt Content

```typescript
// lib/content-schemas.ts
import { z } from 'zod/v4'

export const ArticleSchema = z.object({
  title: z.string(),
  description: z.string().optional(),
  publishedAt: z.iso.date(),
  pillar: z.enum(['ai', 'engineering', 'ux'])
})

// Interface pour les props de composants (après parsing Content)
export interface ArticleCardProps {
  title: string
  description?: string
  publishedAt: string  // ISO string après queryCollection
  pillar: 'ai' | 'engineering' | 'ux'
  path: string  // Ajouté par Content, pas dans le schema frontmatter
}
```

```vue
<!-- components/content/ArticleCard.vue -->
<script setup lang="ts">
import type { ArticleCardProps } from '@/lib/content-schemas'

const props = defineProps<ArticleCardProps>()
</script>

<template>
  <NuxtLink :to="props.path" class="block p-4 rounded-lg hover:bg-muted">
    <h3 class="font-semibold">{{ props.title }}</h3>
    <p v-if="props.description" class="text-muted-foreground">
      {{ props.description }}
    </p>
    <time :datetime="props.publishedAt" class="text-sm">
      {{ new Date(props.publishedAt).toLocaleDateString() }}
    </time>
  </NuxtLink>
</template>
```

---
