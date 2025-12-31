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

## Copy Button avec Feedback — UX Spec Required

Le bouton de copie est un élément UX critique pour le persona Lucas (Time-to-value < 60s).

### Requirements UX

| Exigence | Spécification |
|----------|---------------|
| **Visibilité** | Toujours visible (pas hover-only) |
| **Position** | Coin supérieur droit du code block |
| **Feedback** | Icône → checkmark + texte "Copié!" pendant 2s |
| **Accessibilité** | `aria-label`, focus visible, keyboard accessible |

### Implémentation ProsePre avec Copy Button

```vue
<!-- app/components/content/ProsePre.vue -->
<script setup lang="ts">
import { Check, Copy } from 'lucide-vue-next'
import { Button } from '@/components/ui/button'

const props = defineProps<{
  code: string
  language?: string
  filename?: string
  highlights?: number[]
}>()

const copied = ref(false)
let timeoutId: ReturnType<typeof setTimeout> | null = null

async function copyToClipboard() {
  try {
    await navigator.clipboard.writeText(props.code)
    copied.value = true

    // Reset après 2 secondes (UX Spec)
    if (timeoutId) clearTimeout(timeoutId)
    timeoutId = setTimeout(() => {
      copied.value = false
    }, 2000)
  } catch (err) {
    console.error('Failed to copy:', err)
  }
}

onUnmounted(() => {
  if (timeoutId) clearTimeout(timeoutId)
})
</script>

<template>
  <div class="group relative">
    <!-- Language badge (coin supérieur gauche) -->
    <span
      v-if="language"
      class="absolute left-3 top-2 text-xs text-foreground-muted font-mono"
    >
      {{ language }}
    </span>

    <!-- Filename si présent -->
    <div
      v-if="filename"
      class="flex items-center gap-2 border-b border-border px-4 py-2 text-sm text-foreground-muted"
    >
      {{ filename }}
    </div>

    <!-- Copy button (toujours visible - UX Spec) -->
    <Button
      variant="ghost"
      size="icon"
      class="absolute right-2 top-2 h-8 w-8"
      :aria-label="copied ? 'Copié!' : 'Copier le code'"
      @click="copyToClipboard"
    >
      <Transition name="fade" mode="out-in">
        <Check
          v-if="copied"
          class="h-4 w-4 text-success"
          aria-hidden="true"
        />
        <Copy
          v-else
          class="h-4 w-4"
          aria-hidden="true"
        />
      </Transition>

      <!-- Feedback texte "Copié!" -->
      <Transition name="slide-fade">
        <span
          v-if="copied"
          class="absolute -left-12 text-xs text-success font-medium"
        >
          Copié!
        </span>
      </Transition>
    </Button>

    <!-- Code content -->
    <pre
      class="overflow-x-auto rounded-lg bg-background-secondary p-4 pt-8"
      :class="{ 'pt-12': filename }"
    ><slot /></pre>
  </div>
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 150ms ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-fade-enter-active {
  transition: all 200ms ease-out;
}
.slide-fade-leave-active {
  transition: all 150ms ease-in;
}
.slide-fade-enter-from {
  opacity: 0;
  transform: translateX(10px);
}
.slide-fade-leave-to {
  opacity: 0;
}
</style>
```

### Points critiques

1. **Toujours visible** : Pas de `opacity-0 group-hover:opacity-100` — accessibilité + mobile
2. **Feedback 2 secondes** : Durée spécifiée dans l'UX Spec
3. **Icône Check** : Confirmation visuelle immédiate avec couleur `--success`
4. **Texte "Copié!"** : Feedback textuel pour renforcer l'action
5. **Cleanup timeout** : Évite les memory leaks avec `onUnmounted`

### Composable réutilisable (optionnel)

```typescript
// composables/useCopyToClipboard.ts
export function useCopyToClipboard(duration = 2000) {
  const copied = ref(false)
  let timeoutId: ReturnType<typeof setTimeout> | null = null

  async function copy(text: string) {
    try {
      await navigator.clipboard.writeText(text)
      copied.value = true

      if (timeoutId) clearTimeout(timeoutId)
      timeoutId = setTimeout(() => {
        copied.value = false
      }, duration)

      return true
    } catch (err) {
      console.error('Copy failed:', err)
      return false
    }
  }

  onUnmounted(() => {
    if (timeoutId) clearTimeout(timeoutId)
  })

  return { copied, copy }
}
```

**Usage :**

```vue
<script setup>
const { copied, copy } = useCopyToClipboard()
</script>
```
