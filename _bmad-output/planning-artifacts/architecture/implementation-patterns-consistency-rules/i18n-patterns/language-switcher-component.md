# Language Switcher Component

## Pattern Toggle Accessible (WCAG 2.2 - Recommandé pour bilingue)

Pour un site **bilingue FR/EN**, le pattern toggle inline est recommandé : un seul clic, options toujours visibles, conforme aux conventions des sites gouvernementaux bilingues.

```vue
<!-- app/components/layout/LanguageSwitcher.vue -->
<script setup lang="ts">
const { locale } = useI18n()
const switchLocalePath = useSwitchLocalePath()
</script>

<template>
  <nav aria-label="Sélection de langue">
    <span
      v-if="locale === 'fr'"
      aria-current="page"
      lang="fr"
      class="font-semibold"
    >
      FR
    </span>
    <NuxtLink
      v-else
      :to="switchLocalePath('fr')"
      lang="fr"
      hreflang="fr"
    >
      FR
    </NuxtLink>

    <span aria-hidden="true" class="mx-2">|</span>

    <span
      v-if="locale === 'en'"
      aria-current="page"
      lang="en"
      class="font-semibold"
    >
      EN
    </span>
    <NuxtLink
      v-else
      :to="switchLocalePath('en')"
      lang="en"
      hreflang="en"
    >
      EN
    </NuxtLink>
  </nav>
</template>
```

**Attributs ARIA essentiels :**

| Attribut | Élément | Rôle |
|----------|---------|------|
| `aria-label` | `<nav>` | Identifie la navigation pour lecteurs d'écran |
| `aria-current="page"` | Langue active | Indique la sélection courante |
| `lang` | Chaque option | Prononciation correcte par lecteur d'écran |
| `hreflang` | Liens | SEO + indication langue cible |
| `aria-hidden="true"` | Séparateur `\|` | Masqué des lecteurs d'écran |

**⚠️ Éviter les drapeaux seuls** : Ils représentent des pays, pas des langues. Utiliser les codes ISO ou noms natifs ("Français", "English").

## Pattern ToggleGroup shadcn-vue (Alternative stylisée)

Utilise le composant ToggleGroup de Reka UI pour un rendu plus moderne :

```vue
<!-- app/components/layout/LanguageSwitcher.vue -->
<script setup lang="ts">
import { ToggleGroup, ToggleGroupItem } from '@/components/ui/toggle-group'

const { locale } = useI18n()
const switchLocalePath = useSwitchLocalePath()
</script>

<template>
  <ToggleGroup
    type="single"
    :model-value="locale"
    variant="outline"
    size="sm"
    class="border rounded-md"
  >
    <ToggleGroupItem
      value="fr"
      as-child
      aria-label="Français"
      class="px-3 data-[state=on]:bg-primary data-[state=on]:text-primary-foreground"
    >
      <NuxtLink :to="switchLocalePath('fr')" lang="fr">FR</NuxtLink>
    </ToggleGroupItem>
    <ToggleGroupItem
      value="en"
      as-child
      aria-label="English"
      class="px-3 data-[state=on]:bg-primary data-[state=on]:text-primary-foreground"
    >
      <NuxtLink :to="switchLocalePath('en')" lang="en">EN</NuxtLink>
    </ToggleGroupItem>
  </ToggleGroup>
</template>
```

**Positionnement recommandé** : Haut à droite du header, à côté des éléments utilitaires (recherche, thème).

## Pattern NuxtLink simple (liste de langues)

Pour plus de 2 langues ou un design minimaliste :

```vue
<template>
  <nav class="flex gap-2">
    <NuxtLink
      v-for="loc in locales"
      :key="loc.code"
      :to="switchLocalePath(loc.code)"
      :lang="loc.code"
      :hreflang="loc.code"
      :class="{ 'font-bold': locale === loc.code }"
    >
      {{ loc.code.toUpperCase() }}
    </NuxtLink>
  </nav>
</template>
```
