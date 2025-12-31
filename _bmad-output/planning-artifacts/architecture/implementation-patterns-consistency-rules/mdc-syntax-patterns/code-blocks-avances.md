# Code Blocks avancés

## Syntaxe filename

Afficher le nom de fichier dans le header du code block :

```markdown
```typescript [nuxt.config.ts]
export default defineNuxtConfig({
  // ...
})
```
```

Le composant `ProsePre` reçoit la prop `filename` parsée automatiquement.

## Line highlighting

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

## Breaking change : ProseCode → ProsePre

⚠️ **Migration Content 2.x → 3.x critique :**

| Composant v2 | Composant v3 | Usage |
|--------------|--------------|-------|
| `ProseCodeInline` | `ProseCode` | `` `code inline` `` |
| `ProseCode` | `ProsePre` | ` ``` code blocks ``` ` |

**Action requise :** Si vous avez un composant custom `ProseCode.vue` pour les blocs, **renommez-le en `ProsePre.vue`**.

## Props ProsePre

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

## Code groups avec persistance utilisateur

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
