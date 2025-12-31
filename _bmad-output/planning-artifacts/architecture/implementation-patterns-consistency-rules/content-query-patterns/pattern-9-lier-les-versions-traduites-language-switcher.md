# Pattern 9 : Lier les versions traduites (Language Switcher)

## Problème

Il n'existe **pas de connexion native** entre les versions localisées d'un même contenu dans Nuxt Content 3. Le language switcher ne peut pas automatiquement trouver la version traduite d'un article.

## Solution : UID de traduction dans le frontmatter

Ajouter un champ `translationUid` identique dans les deux versions :

```yaml
# content/en/blog/getting-started.md
---
title: "Getting Started with Nuxt"
slug: "getting-started"
translationUid: "nuxt-intro-001"  # ← UID partagé
---

# content/fr/blog/demarrage.md
---
title: "Démarrer avec Nuxt"
slug: "demarrage"
translationUid: "nuxt-intro-001"  # ← Même UID
---
```

## Schema Zod avec translationUid

```typescript
// content.config.ts
const articleSchema = z.object({
  title: z.string(),
  slug: z.string(),
  // ... autres champs
  translationUid: z.string().optional(),  // UID pour lier les traductions
})
```

## Composable pour trouver la version traduite

```typescript
// composables/useTranslatedArticle.ts
import type { Collections } from '@nuxt/content'

export async function useTranslatedArticle(
  translationUid: string | undefined,
  targetLocale: string
) {
  if (!translationUid) return null

  const collection = `articles_${targetLocale}` as keyof Collections

  const translated = await queryCollection(collection)
    .where('translationUid', '=', translationUid)
    .first()

  return translated
}
```

## Language Switcher avec lien vers traduction

```vue
<!-- app/components/content/ArticleLanguageSwitcher.vue -->
<script setup lang="ts">
import type { Collections } from '@nuxt/content'

interface Props {
  translationUid?: string
}

const props = defineProps<Props>()
const { locale, locales } = useI18n()
const localePath = useLocalePath()

// Trouver les traductions disponibles
const { data: translations } = await useAsyncData(
  `translations-${props.translationUid}`,
  async () => {
    if (!props.translationUid) return {}

    const results: Record<string, string> = {}

    for (const loc of locales.value) {
      if (loc.code === locale.value) continue

      const collection = `articles_${loc.code}` as keyof Collections
      const translated = await queryCollection(collection)
        .where('translationUid', '=', props.translationUid)
        .select('path')
        .first()

      if (translated) {
        results[loc.code] = translated.path
      }
    }

    return results
  }
)
</script>

<template>
  <nav v-if="translations && Object.keys(translations).length > 0" class="flex gap-2">
    <span class="text-muted-foreground">Lire en :</span>
    <NuxtLink
      v-for="(path, code) in translations"
      :key="code"
      :to="path"
      class="text-primary hover:underline"
    >
      {{ code.toUpperCase() }}
    </NuxtLink>
  </nav>
</template>
```

## Alternative : slugs identiques

Si les slugs sont identiques entre langues, `useSwitchLocalePath()` fonctionne directement :

```vue
<script setup>
const switchLocalePath = useSwitchLocalePath()
</script>

<template>
  <!-- Fonctionne si /blog/my-article et /fr/blog/my-article existent -->
  <NuxtLink :to="switchLocalePath('fr')">
    Lire en français
  </NuxtLink>
</template>
```
