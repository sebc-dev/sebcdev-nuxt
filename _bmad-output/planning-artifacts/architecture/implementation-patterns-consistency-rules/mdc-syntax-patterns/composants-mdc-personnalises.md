# Composants MDC personnalisés

## Structure de fichiers Nuxt 4

```
projet/
├── app/
│   └── components/
│       ├── content/           # Composants MDC personnalisés (directement ici)
│       │   ├── Alert.vue      # → ::alert
│       │   ├── FeatureCard.vue # → ::feature-card
│       │   ├── ProseA.vue     # Override liens Markdown
│       │   ├── ProseH2.vue    # Override h2 Markdown
│       │   └── ProsePre.vue   # Override code blocks Markdown
│       └── ui/                # shadcn-vue
├── content/                   # À la racine (pas dans app/)
│   ├── fr/
│   └── en/
└── nuxt.config.ts
```

**Mapping nommage :** PascalCase → kebab-case
- `FeatureCard.vue` → `::feature-card`
- `IconButton.vue` → `::icon-button`

**Préfixe `Prose` réservé :** Override des éléments HTML natifs du Markdown (liens, headings, code blocks, etc.)

⚠️ **Important** : Les composants Prose doivent être dans `components/content/` directement (pas dans un sous-dossier `prose/`).

## Liste complète des Prose Components

| Tag HTML | Composant Vue | Tag HTML | Composant Vue |
|----------|---------------|----------|---------------|
| `<p>` | `ProseP` | `<table>` | `ProseTable` |
| `<h1>` à `<h6>` | `ProseH1` à `ProseH6` | `<thead>` | `ProseThead` |
| `<ul>` | `ProseUl` | `<tbody>` | `ProseTbody` |
| `<ol>` | `ProseOl` | `<tr>` | `ProseTr` |
| `<li>` | `ProseLi` | `<th>` | `ProseTh` |
| `<blockquote>` | `ProseBlockquote` | `<td>` | `ProseTd` |
| `<pre>` | `ProsePre` | `<a>` | `ProseA` |
| `<code>` (inline) | `ProseCode` | `<img>` | `ProseImg` |
| `<em>` | `ProseEm` | `<hr>` | `ProseHr` |
| `<strong>` | `ProseStrong` | — | — |

Pour personnaliser un composant, créez un fichier du même nom dans `components/content/`.

## Configuration enregistrement global

Les composants dans `components/content/` ne sont **pas automatiquement globaux** en Nuxt Content v3. Configuration explicite requise :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // CRITIQUE: Enregistrement global des composants content
  components: {
    global: true,
    dirs: [
      { path: '~/components/content', global: true },
      '~/components'
    ]
  },

  content: {
    build: {
      markdown: {
        highlight: {
          theme: { default: 'github-light', dark: 'github-dark' },
          langs: ['json', 'js', 'ts', 'html', 'css', 'vue', 'shell', 'yaml']
        }
      }
    }
  }
})
```

## Pièges courants et solutions

### Erreur d'hydration avec wrappers

Ajouter des éléments wrapper aux composants inline cause des mismatches d'hydration SSR :

```vue
<!-- ❌ PROVOQUE UNE ERREUR D'HYDRATION -->
<template>
  <figure>
    <img :src="src" :alt="alt" />
    <figcaption>{{ alt }}</figcaption>
  </figure>
</template>

<!-- ✅ SOLUTION: Utiliser ClientOnly -->
<template>
  <ClientOnly>
    <figure>
      <img :src="src" :alt="alt" />
      <figcaption>{{ alt }}</figcaption>
    </figure>
    <template #fallback>
      <img :src="src" :alt="alt" />
    </template>
  </ClientOnly>
</template>
```

### Composants non rendus après `nuxi generate`

Si les composants fonctionnent en dev mais pas après génération statique :

1. Vérifier l'enregistrement global dans `nuxt.config.ts`
2. Alternative : suffixe `.global.vue` dans `components/mdc/`

```
components/mdc/ProsePre.global.vue
```

### Warning "Two component files resolving to same name"

Ce warning est **attendu** lors de la surcharge de composants Prose. Il confirme que votre composant personnalisé remplace celui par défaut.

### Remplacement de ContentSlot (migration v2 → v3)

```vue
<!-- ❌ v2 (obsolète) -->
<ContentSlot :use="$slots.default" unwrap="p" />
<MDCSlot unwrap="p" />

<!-- ✅ v3 (correct) -->
<slot mdc-unwrap="p" />
```

## Pattern TypeScript recommandé

```vue
<!-- app/components/content/Alert.vue -->
<script setup lang="ts">
interface AlertProps {
  type?: 'info' | 'warning' | 'error' | 'success'
  title?: string
}

const props = withDefaults(defineProps<AlertProps>(), {
  type: 'info',
  title: ''
})
</script>

<template>
  <div
    class="alert rounded-lg p-4"
    :class="{
      'bg-blue-100 text-blue-800': props.type === 'info',
      'bg-amber-100 text-amber-800': props.type === 'warning',
      'bg-red-100 text-red-800': props.type === 'error',
      'bg-green-100 text-green-800': props.type === 'success',
    }"
    role="alert"
  >
    <h4 v-if="props.title" class="font-semibold mb-2">
      {{ props.title }}
    </h4>
    <div class="alert-content">
      <slot mdc-unwrap="p" />
    </div>
  </div>
</template>
```

**Usage MDC :**

```markdown
::alert{type="warning" title="Attention"}
Ceci est un message d'avertissement avec du **Markdown**.
::
```
