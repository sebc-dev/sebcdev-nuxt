# MDC Syntax Patterns

Patterns de syntaxe MDC (Markdown Components) pour Nuxt Content 3.10+ avec Nuxt 4.

## Syntaxe de base

### Composants inline vs block

| Type | Syntaxe | Usage |
|------|---------|-------|
| **Inline** | `:component[contenu]{props}` | Éléments simples sans slots (badges, icônes) |
| **Block** | `::component{props}` ... `::` | Contenu Markdown complet avec slots |

```markdown
<!-- Inline - badge simple -->
:badge[Nouveau]{type="success"}

<!-- Block - carte avec contenu -->
::card{title="Ma carte"}
Contenu Markdown avec **formatage**.
::
```

### Props et attributs

```markdown
<!-- Props simples inline -->
::alert{type="warning" title="Attention"}

<!-- Props complexes avec YAML -->
::icon-card
---
icon: IconNuxt
title: Architecture Nuxt
description: Exploitez toute la puissance de Nuxt
---
::

<!-- Binding dynamique depuis frontmatter -->
::alert{:type="frontmatterType"}

<!-- Tableaux JSON (guillemets simples externes, doubles internes) -->
:tags{:items='["Nuxt", "Vue", "TypeScript"]'}

<!-- Classes et ID -->
::div{.bg-blue-100 .p-4 .rounded-lg #section-intro}
```

### Nesting (imbrication)

Ajouter des colons pour chaque niveau d'imbrication :

```markdown
::container
:::grid{cols="2"}
::::card{title="Carte 1"}
Contenu carte 1
::::

::::card{title="Carte 2"}
Contenu carte 2
::::
:::
::
```

**Limite recommandée : 3-4 niveaux maximum.** Au-delà, restructurer en sous-composants.

### Piège du spécificateur vide

Quand un composant inline est suivi de `-`, `_` ou `:`, ajouter un spécificateur vide :

```markdown
<!-- Problème : le - est interprété comme partie du composant -->
:icon-world

<!-- Solution : spécificateur vide {} -->
:icon{}-world
```

## Named Slots

### Syntaxe slots nommés

```markdown
::hero{image="/images/bg.jpg"}
#title
Bienvenue sur notre site

#description
Une description avec du **Markdown** formaté.

Contenu du slot par défaut ici.
::
```

### Attribut `mdc-unwrap`

Le parser Markdown enveloppe automatiquement le texte dans des balises `<p>`. Pour les slots où `<p>` serait invalide (ex: dans un `<h1>`), utiliser `mdc-unwrap` :

```vue
<!-- app/components/content/Hero.vue -->
<template>
  <section class="hero">
    <!-- Sans mdc-unwrap: <h1><p>Titre</p></h1> = HTML invalide -->
    <!-- Avec mdc-unwrap: <h1>Titre</h1> = HTML valide -->
    <h1 v-if="$slots.title" class="hero__title">
      <slot name="title" mdc-unwrap="p" />
    </h1>

    <p v-if="$slots.description" class="hero__description">
      <slot name="description" mdc-unwrap="p" />
    </p>

    <!-- Slot par défaut - garder les <p> -->
    <div class="hero__content">
      <slot />
    </div>
  </section>
</template>
```

**Valeurs possibles pour `mdc-unwrap` :**
- `"p"` - Supprime les balises `<p>`
- `"li"` - Supprime les balises `<li>` (pour listes)
- `"p li"` - Supprime les deux

### Vérification conditionnelle des slots

```vue
<template>
  <!-- Afficher seulement si le slot est fourni -->
  <footer v-if="$slots.footer" class="card-footer">
    <slot name="footer" mdc-unwrap="p" />
  </footer>
</template>
```

**Conventions de nommage slots :**
- `#title` - Titre principal
- `#description` - Description courte
- `#content` - Contenu principal étendu
- `#footer` - Pied de composant
- `#default` - Slot principal (implicite)

## Composants MDC personnalisés

### Structure de fichiers Nuxt 4

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

### Liste complète des Prose Components

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

### Configuration enregistrement global

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

### Pièges courants et solutions

#### Erreur d'hydration avec wrappers

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

#### Composants non rendus après `nuxi generate`

Si les composants fonctionnent en dev mais pas après génération statique :

1. Vérifier l'enregistrement global dans `nuxt.config.ts`
2. Alternative : suffixe `.global.vue` dans `components/mdc/`

```
components/mdc/ProsePre.global.vue
```

#### Warning "Two component files resolving to same name"

Ce warning est **attendu** lors de la surcharge de composants Prose. Il confirme que votre composant personnalisé remplace celui par défaut.

#### Remplacement de ContentSlot (migration v2 → v3)

```vue
<!-- ❌ v2 (obsolète) -->
<ContentSlot :use="$slots.default" unwrap="p" />
<MDCSlot unwrap="p" />

<!-- ✅ v3 (correct) -->
<slot mdc-unwrap="p" />
```

### Pattern TypeScript recommandé

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

## Wrapper pattern shadcn-vue

Les composants shadcn-vue doivent être wrappés pour l'usage MDC :

```vue
<!-- app/components/content/MdcAlert.vue -->
<script setup lang="ts">
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'

interface Props {
  variant?: 'default' | 'destructive'
  title?: string
}

withDefaults(defineProps<Props>(), {
  variant: 'default'
})
</script>

<template>
  <Alert :variant="variant">
    <AlertTitle v-if="title">{{ title }}</AlertTitle>
    <AlertDescription>
      <slot mdc-unwrap="p" />
    </AlertDescription>
  </Alert>
</template>
```

**Usage MDC :**

```markdown
::mdc-alert{title="Note importante" variant="destructive"}
Contenu avec accessibilité WAI-ARIA intégrée via Reka UI.
::
```

### Pattern Card avec slots multiples

```vue
<!-- app/components/content/MdcCard.vue -->
<script setup lang="ts">
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'

interface Props {
  class?: string
}

defineProps<Props>()
</script>

<template>
  <Card :class="$props.class">
    <CardHeader v-if="$slots.title || $slots.description">
      <CardTitle v-if="$slots.title">
        <slot name="title" mdc-unwrap="p" />
      </CardTitle>
      <CardDescription v-if="$slots.description">
        <slot name="description" mdc-unwrap="p" />
      </CardDescription>
    </CardHeader>
    <CardContent>
      <slot />
    </CardContent>
  </Card>
</template>
```

**Usage MDC :**

```markdown
::mdc-card{.max-w-md}
#title
Titre de la carte

#description
Description courte

Contenu principal avec **formatage Markdown**.
::
```

## Composants async imbriqués

Pour les composants avec données async imbriqués dans MDC :

```vue
<script setup lang="ts">
const { data } = await useAsyncData('key', () => fetchData(), {
  immediate: true  // Requis pour composants imbriqués
})
</script>

<template>
  <Suspense suspensible>
    <div v-if="data">{{ data }}</div>
  </Suspense>
</template>
```

## Résumé

| Pattern | Cas d'usage | Élément clé |
|---------|-------------|-------------|
| Inline `:comp[]{props}` | Badges, icônes | Sans slots |
| Block `::comp{props}` | Cartes, alertes | Avec contenu Markdown |
| Named slots `#name` | Sections distinctes | `mdc-unwrap="p"` |
| Wrapper shadcn-vue | Intégration UI library | Préfixe `Mdc` |
| Prose overrides | Éléments natifs MD | Préfixe `Prose` |

## Styling Shiki pour TailwindCSS 4

### CSS Variables Dark Mode

Shiki génère automatiquement des CSS variables pour chaque token lors du dual-theme. L'intégration avec TailwindCSS 4 et la classe `html.dark` nécessite ce CSS :

```css
/* app/assets/css/shiki.css (à importer dans main.css) */

/* Switch automatique dark mode via CSS variables Shiki */
html.dark .shiki,
html.dark .shiki span {
  color: var(--shiki-dark) !important;
  background-color: var(--shiki-dark-bg) !important;
  font-style: var(--shiki-dark-font-style) !important;
}
```

### Styling lignes highlight/diff (oklch)

Les transformers Shiki ajoutent des classes CSS mais ne fournissent pas le styling. Ajouter ces styles pour les lignes surlignées et les diffs :

```css
/* Lignes surlignées */
.shiki .line.highlighted {
  background: oklch(0.95 0.01 250);
}
html.dark .shiki .line.highlighted {
  background: oklch(0.25 0.02 250);
}

/* Diffs add/remove */
.shiki .line.diff.add {
  background: oklch(0.95 0.05 145);
}
html.dark .shiki .line.diff.add {
  background: oklch(0.30 0.08 145);
}

.shiki .line.diff.remove {
  background: oklch(0.95 0.05 25);
}
html.dark .shiki .line.diff.remove {
  background: oklch(0.30 0.08 25);
}

/* Focus avec blur des autres lignes */
.shiki.has-focused .line:not(.focused) {
  opacity: 0.5;
  filter: blur(1px);
  transition: opacity 0.3s, filter 0.3s;
}

/* Annotations erreur */
.shiki .line.highlighted.error {
  background: oklch(0.95 0.08 25);
}
html.dark .shiki .line.highlighted.error {
  background: oklch(0.30 0.10 25);
}
```

### Accessibilité : Focus Visible

Shiki ajoute automatiquement `tabindex="0"` aux éléments `<pre>` pour la navigation clavier. Ajouter un outline visible pour le focus :

```css
/* Focus visible pour navigation clavier */
pre.shiki:focus-visible {
  outline: 2px solid oklch(0.6 0.15 250);
  outline-offset: 2px;
}

html.dark pre.shiki:focus-visible {
  outline-color: oklch(0.7 0.15 250);
}
```

### Icônes de langage (Nuxt UI)

Pour afficher des icônes de langage dans les headers de code blocks avec Nuxt UI, configurer `app.config.ts` :

```typescript
// app.config.ts
export default defineAppConfig({
  ui: {
    prose: {
      codeIcon: {
        // Fichiers spécifiques
        'nuxt.config.ts': 'i-vscode-icons-file-type-nuxt',
        'package.json': 'i-vscode-icons-file-type-npm',
        'tsconfig.json': 'i-vscode-icons-file-type-tsconfig',
        'tailwind.config.ts': 'i-vscode-icons-file-type-tailwind',
        '.env': 'i-vscode-icons-file-type-dotenv',

        // Langages
        ts: 'i-vscode-icons-file-type-typescript',
        typescript: 'i-vscode-icons-file-type-typescript',
        vue: 'i-vscode-icons-file-type-vue',
        js: 'i-vscode-icons-file-type-js',
        json: 'i-vscode-icons-file-type-json',
        css: 'i-vscode-icons-file-type-css',
        html: 'i-vscode-icons-file-type-html',
        md: 'i-vscode-icons-file-type-markdown',
        yaml: 'i-vscode-icons-file-type-yaml',
        shell: 'i-vscode-icons-file-type-shell',
        bash: 'i-vscode-icons-file-type-shell',
      }
    }
  }
})
```

**Prérequis :** Installer `@iconify-json/vscode-icons` pour les icônes VS Code.

---

## Code Blocks avancés

### Syntaxe filename

Afficher le nom de fichier dans le header du code block :

```markdown
```typescript [nuxt.config.ts]
export default defineNuxtConfig({
  // ...
})
```
```

Le composant `ProsePre` reçoit la prop `filename` parsée automatiquement.

### Line highlighting

Plusieurs syntaxes disponibles pour mettre en surbrillance des lignes :

| Syntaxe | Effet | Classes CSS |
|---------|-------|-------------|
| `{2}` | Ligne unique | `.line.highlighted` |
| `{1,3,5}` | Lignes multiples | `.line.highlighted` |
| `{3-7}` | Plage de lignes | `.line.highlighted` |
| `// [!code highlight]` | Via commentaire inline | `.line.highlighted` |
| `// [!code focus]` | Focus avec blur autres lignes | `.line.focused` |
| `// [!code ++]` | Diff add (vert) | `.line.diff.add` |
| `// [!code --]` | Diff remove (rouge) | `.line.diff.remove` |
| `// [!code error]` | Annotation erreur | `.line.highlighted.error` |

**Exemple combiné filename + highlights :**

```markdown
```typescript [nuxt.config.ts]{4-5}
export default defineNuxtConfig({
  content: {
    build: {
      markdown: {} // Ces lignes seront highlight
    }
  }
})
```
```

### Breaking change : ProseCode → ProsePre

⚠️ **Migration Content 2.x → 3.x critique :**

| Composant v2 | Composant v3 | Usage |
|--------------|--------------|-------|
| `ProseCodeInline` | `ProseCode` | `` `code inline` `` |
| `ProseCode` | `ProsePre` | ` ``` code blocks ``` ` |

**Action requise :** Si vous avez un composant custom `ProseCode.vue` pour les blocs, **renommez-le en `ProsePre.vue`**.

### Props ProsePre

Le composant `ProsePre` reçoit ces props parsées depuis la syntaxe MDC :

```typescript
interface ProsePreProps {
  code: string           // Contenu du code
  language: string       // 'typescript', 'vue', etc.
  filename?: string      // 'nuxt.config.ts'
  highlights?: number[]  // [4, 5] pour {4-5}
  meta?: string          // Attributs additionnels
}
```

### Code groups avec persistance utilisateur

Les code groups créent des onglets pour alternatives (package managers, etc.) :

```markdown
::code-group

```bash [pnpm]
pnpm add @nuxt/content
```

```bash [npm]
npm install @nuxt/content
```

```bash [yarn]
yarn add @nuxt/content
```

::
```

**Composant custom avec persistance localStorage :**

```vue
<!-- app/components/content/ProseCodeGroup.vue -->
<script setup lang="ts">
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs'

const props = defineProps<{
  files: Array<{ filename: string; code: string; language: string }>
}>()

const STORAGE_KEY = 'preferred-package-manager'
const activeTab = ref(props.files[0]?.filename || '')

onMounted(() => {
  const saved = localStorage.getItem(STORAGE_KEY)
  if (saved && props.files.some(f => f.filename === saved)) {
    activeTab.value = saved
  }
})

watch(activeTab, (val) => {
  // Persist uniquement pour package managers connus
  if (['pnpm', 'npm', 'yarn', 'bun'].includes(val)) {
    localStorage.setItem(STORAGE_KEY, val)
  }
})
</script>

<template>
  <Tabs v-model="activeTab" class="code-group">
    <TabsList>
      <TabsTrigger
        v-for="file in files"
        :key="file.filename"
        :value="file.filename"
      >
        {{ file.filename }}
      </TabsTrigger>
    </TabsList>
    <TabsContent
      v-for="file in files"
      :key="file.filename"
      :value="file.filename"
    >
      <slot :name="file.filename" />
    </TabsContent>
  </Tabs>
</template>
```

**Avantage :** Le choix du package manager est mémorisé entre les pages et sessions.
