# Prose Components Patterns

Patterns d'implémentation des Prose Components personnalisés pour Nuxt Content v3.10+ avec shadcn-vue.

## ProsePre.vue - Blocs de code avec bouton copie

Implémentation complète du composant `ProsePre` avec **bouton de copie accessible**, **états visuels**, et **fallback navigateur**.

### Props reçus automatiquement

```typescript
interface ProsePreProps {
  code: string           // Texte brut du code (pour copie)
  language: string | null // 'typescript', 'vue', etc.
  filename: string | null // 'nuxt.config.ts' (syntaxe [filename])
  highlights: number[]    // [4, 5] pour {4-5}
  meta: string | null     // Attributs additionnels
  class: string | null    // Classes CSS Shiki
}
```

### Implémentation complète

```vue
<!-- app/components/content/ProsePre.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue'
import { Button } from '@/components/ui/button'
import { Check, Copy, X } from 'lucide-vue-next'

interface Props {
  code: string
  language: string | null
  filename: string | null
  highlights: number[]
  meta: string | null
  class: string | null
}

const props = withDefaults(defineProps<Props>(), {
  code: '',
  language: null,
  filename: null,
  highlights: () => [],
  meta: null,
  class: null
})

type CopyStatus = 'idle' | 'copying' | 'copied' | 'error'
const status = ref<CopyStatus>('idle')
const announcement = ref('')

// Vérification SSR-safe de la disponibilité du clipboard
const isSupported = computed(() =>
  typeof navigator !== 'undefined' && !!navigator.clipboard
)

const buttonLabel = computed(() => ({
  idle: 'Copier le code',
  copying: 'Copie en cours...',
  copied: 'Copié !',
  error: 'Échec de la copie'
}[status.value]))

// Fallback pour navigateurs anciens (IE11, Safari < 13.1)
function fallbackCopy(text: string): boolean {
  const textarea = document.createElement('textarea')
  textarea.value = text
  textarea.setAttribute('readonly', '')
  textarea.style.cssText = 'position:fixed;left:-9999px'
  document.body.appendChild(textarea)
  textarea.select()
  const success = document.execCommand('copy')
  document.body.removeChild(textarea)
  return success
}

async function copyCode(): Promise<void> {
  if (!props.code) return
  status.value = 'copying'

  try {
    if (isSupported.value) {
      await navigator.clipboard.writeText(props.code)
    } else if (!fallbackCopy(props.code)) {
      throw new Error('execCommand fallback failed')
    }

    status.value = 'copied'
    announcement.value = 'Code copié dans le presse-papiers'

    setTimeout(() => {
      status.value = 'idle'
      announcement.value = ''
    }, 2500)
  } catch (error) {
    console.error('Copy failed:', error)
    status.value = 'error'
    announcement.value = 'Impossible de copier le code'

    setTimeout(() => {
      status.value = 'idle'
      announcement.value = ''
    }, 2500)
  }
}
</script>

<template>
  <div class="code-block group relative my-6 rounded-lg overflow-hidden
              border border-border bg-muted/30">
    <!-- En-tête avec nom de fichier et bouton de copie -->
    <div class="flex items-center justify-between px-4 py-2
                bg-muted/50 border-b border-border">
      <span class="text-sm text-muted-foreground font-mono">
        {{ filename || language || 'code' }}
      </span>

      <Button
        variant="ghost"
        size="sm"
        :aria-label="buttonLabel"
        :disabled="!code"
        class="h-8 gap-1.5 opacity-0 group-hover:opacity-100
               focus:opacity-100 transition-opacity"
        :class="{
          'text-green-500': status === 'copied',
          'text-destructive': status === 'error'
        }"
        @click="copyCode"
      >
        <Copy v-if="status === 'idle'" class="h-4 w-4" aria-hidden="true" />
        <Check v-else-if="status === 'copied'" class="h-4 w-4" aria-hidden="true" />
        <X v-else-if="status === 'error'" class="h-4 w-4" aria-hidden="true" />
        <span class="hidden sm:inline">{{ buttonLabel }}</span>
      </Button>
    </div>

    <!-- Corps du bloc de code -->
    <pre class="overflow-x-auto p-4 text-sm leading-relaxed" :class="$props.class"><slot /></pre>

    <!-- Zone live pour lecteurs d'écran -->
    <div
      role="status"
      aria-live="polite"
      aria-atomic="true"
      class="sr-only"
    >
      {{ announcement }}
    </div>
  </div>
</template>

<style>
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
</style>
```

### États du bouton copie

| État | Icône | Couleur | Durée |
|------|-------|---------|-------|
| `idle` | Copy | Default | — |
| `copying` | Copy (animé) | Default | Instantané |
| `copied` | Check | `text-green-500` | 2.5s → idle |
| `error` | X | `text-destructive` | 2.5s → idle |

### Accessibilité

- `aria-label` dynamique selon l'état
- Zone `aria-live="polite"` pour annoncer le résultat aux lecteurs d'écran
- `aria-hidden="true"` sur les icônes décoratives
- Bouton focusable même quand masqué (opacity via `focus:opacity-100`)

---

## ProseA.vue - Liens avec détection externe

Implémentation du composant `ProseA` avec **détection automatique des liens externes**, **attributs de sécurité**, et **indicateur visuel accessible**.

### Règles de détection

| Type de lien | Comportement | Exemple |
|--------------|--------------|---------|
| HTTP/HTTPS | Externe (nouvel onglet) | `https://example.com` |
| Protocol-relative | Externe (nouvel onglet) | `//example.com` |
| mailto:/tel: | Même onglet (pas externe) | `mailto:test@example.com` |
| Chemin relatif | Interne (NuxtLink) | `/blog/article` |
| Ancre | Interne | `#section` |

### Implémentation complète

```vue
<!-- app/components/content/ProseA.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { ExternalLink } from 'lucide-vue-next'

interface Props {
  href: string
  target?: string
  rel?: string
}

const props = withDefaults(defineProps<Props>(), {
  href: '',
  target: undefined,
  rel: undefined
})

/**
 * Détection intelligente des liens externes
 * - mailto: et tel: restent dans le même onglet
 * - http/https et URLs protocol-relative ouvrent dans un nouvel onglet
 * - Chemins relatifs et ancres sont internes
 */
const linkInfo = computed(() => {
  const href = props.href

  if (!href) return { isExternal: false, isSpecial: false }

  // Protocoles spéciaux - ne pas ouvrir dans un nouvel onglet
  if (/^(mailto:|tel:)/i.test(href)) {
    return { isExternal: false, isSpecial: true }
  }

  // Liens externes (http, https, protocol-relative)
  if (href.startsWith('http://') || href.startsWith('https://') || href.startsWith('//')) {
    return { isExternal: true, isSpecial: false }
  }

  // Tout le reste est interne
  return { isExternal: false, isSpecial: false }
})

const isExternal = computed(() => linkInfo.value.isExternal)

const computedTarget = computed(() => {
  if (props.target) return props.target
  return isExternal.value ? '_blank' : undefined
})

// Sécurité: noopener prévient le tabnabbing, noreferrer cache le referrer
const computedRel = computed(() => {
  if (props.rel) return props.rel
  return isExternal.value ? 'noopener noreferrer' : undefined
})
</script>

<template>
  <NuxtLink
    :href="href"
    :target="computedTarget"
    :rel="computedRel"
    :external="isExternal"
    class="inline-flex items-center gap-0.5 text-primary underline
           underline-offset-2 hover:text-primary/80 transition-colors"
  >
    <slot />
    <ExternalLink
      v-if="isExternal"
      class="ml-0.5 h-3.5 w-3.5 flex-shrink-0"
      aria-hidden="true"
    />
    <span v-if="isExternal" class="sr-only">(ouvre dans un nouvel onglet)</span>
  </NuxtLink>
</template>
```

### Attributs de sécurité

| Attribut | Raison |
|----------|--------|
| `rel="noopener"` | Empêche `window.opener` - prévient le tabnabbing |
| `rel="noreferrer"` | Cache le HTTP Referer header |
| `target="_blank"` | Ouvre dans un nouvel onglet |

**Pour contenu utilisateur (UGC)** : Ajouter également `rel="nofollow"` pour le SEO.

### Accessibilité

- Icône avec `aria-hidden="true"` (décorative)
- Texte `<span class="sr-only">` pour lecteurs d'écran
- Indication claire qu'un nouvel onglet s'ouvre

---

## ProseImg.vue - Images avec légendes

Implémentation optionnelle pour ajouter des fonctionnalités aux images Markdown.

```vue
<!-- app/components/content/ProseImg.vue -->
<script setup lang="ts">
interface Props {
  src: string
  alt: string
  width?: string | number
  height?: string | number
}

const props = defineProps<Props>()
</script>

<template>
  <figure class="my-6">
    <NuxtImg
      :src="src"
      :alt="alt"
      :width="width"
      :height="height"
      class="rounded-lg mx-auto"
      loading="lazy"
    />
    <figcaption v-if="alt" class="text-center text-sm text-muted-foreground mt-2">
      {{ alt }}
    </figcaption>
  </figure>
</template>
```

**⚠️ Attention** : Ce pattern peut causer des erreurs d'hydration SSR car il ajoute un wrapper `<figure>`. Utiliser `<ClientOnly>` si nécessaire :

```vue
<template>
  <ClientOnly>
    <figure>...</figure>
    <template #fallback>
      <img :src="src" :alt="alt" />
    </template>
  </ClientOnly>
</template>
```

### Règles Alt Text WCAG

| Règle | Explication | Exemple |
|-------|-------------|---------|
| **Décrire fonction ET contenu** | Pas juste l'apparence | ✅ "Graphique montrant 25% croissance Q4" |
| **Images décoratives** | `alt=""` pour ignorer | Icônes accompagnant du texte |
| **Éviter "image de" / "photo de"** | Double annonce par lecteurs d'écran | ❌ "Photo d'un graphique" → ✅ "Graphique des ventes" |
| **Liens avec image seule** | Alt décrit la destination | ✅ "Accueil" pour logo cliquable |
| **Images complexes** | Description longue via `aria-describedby` | Infographies, diagrammes |

```vue
<template>
  <!-- ✅ Image informative -->
  <NuxtImg
    src="/images/chart.png"
    alt="Graphique montrant une croissance de 25% au Q4 2024"
  />

  <!-- ✅ Image décorative (accompagne du texte) -->
  <NuxtImg src="/icons/checkmark.svg" alt="" aria-hidden="true" />

  <!-- ✅ Logo cliquable -->
  <NuxtLink to="/">
    <NuxtImg src="/logo.svg" alt="Accueil sebc.dev" />
  </NuxtLink>

  <!-- ✅ Image complexe avec description longue -->
  <figure>
    <NuxtImg
      src="/infographic.png"
      alt="Infographie du processus de déploiement"
      aria-describedby="infographic-desc"
    />
    <figcaption id="infographic-desc">
      Le processus comprend 5 étapes : développement local,
      tests automatisés, review de code, staging, et production.
    </figcaption>
  </figure>
</template>
```

---

## DynamicHeading.vue - Titres de niveau flexible

Les composants réutilisables (cartes, sections) ne connaissent pas leur niveau de titre approprié dans la hiérarchie de la page. Ce composant accepte le niveau comme prop.

```vue
<!-- app/components/DynamicHeading.vue -->
<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  level: 1 | 2 | 3 | 4 | 5 | 6
  id?: string
}

const props = withDefaults(defineProps<Props>(), {
  level: 2
})

const tag = computed(() => `h${props.level}` as keyof HTMLElementTagNameMap)
</script>

<template>
  <component :is="tag" :id="id">
    <slot />
  </component>
</template>
```

### Utilisation dans un composant Card

```vue
<!-- app/components/content/ArticleCard.vue -->
<script setup lang="ts">
interface Props {
  title: string
  description: string
  href: string
  headingLevel?: 2 | 3 | 4 | 5 | 6
}

const props = withDefaults(defineProps<Props>(), {
  headingLevel: 2
})
</script>

<template>
  <article class="card p-4 border rounded-lg">
    <DynamicHeading :level="headingLevel" class="text-lg font-semibold">
      <NuxtLink :href="href">{{ title }}</NuxtLink>
    </DynamicHeading>
    <p class="text-muted-foreground mt-2">{{ description }}</p>
  </article>
</template>
```

### Exemple d'utilisation contextuelle

```vue
<!-- pages/blog/index.vue -->
<template>
  <main>
    <h1>Tous les articles</h1> <!-- Niveau 1 : titre de page -->

    <section aria-labelledby="featured-heading">
      <h2 id="featured-heading">Articles à la une</h2> <!-- Niveau 2 -->

      <!-- Les cartes utilisent h3 car elles sont dans une section h2 -->
      <ArticleCard
        v-for="article in featured"
        :key="article.id"
        :title="article.title"
        :description="article.description"
        :href="article.path"
        :heading-level="3"
      />
    </section>
  </main>
</template>
```

| Contexte | Niveau carte | Raison |
|----------|--------------|--------|
| Sous un `<h1>` | `2` | Premier niveau de sous-section |
| Sous un `<h2>` | `3` | Sous-sous-section |
| Dans un aside | `2` ou `3` | Dépend de la structure du aside |

---

## ProseH2.vue - Headings avec ancres

Implémentation pour ajouter des liens d'ancrage cliquables aux titres.

```vue
<!-- app/components/content/ProseH2.vue -->
<script setup lang="ts">
interface Props {
  id?: string
}

const props = defineProps<Props>()
</script>

<template>
  <h2 :id="id" class="group flex items-center gap-2 scroll-mt-20">
    <slot />
    <a
      v-if="id"
      :href="`#${id}`"
      class="opacity-0 group-hover:opacity-100 transition-opacity text-muted-foreground hover:text-primary"
      aria-label="Lien vers cette section"
    >
      #
    </a>
  </h2>
</template>
```

**Pattern similaire** : Répliquer pour `ProseH3`, `ProseH4`, etc. avec les classes de taille appropriées.

---

## Résumé des composants

| Composant | Fonction | Accessibilité | Sécurité |
|-----------|----------|---------------|----------|
| `ProsePre` | Blocs de code avec copie | aria-live, aria-label | — |
| `ProseA` | Liens externes détectés | sr-only pour nouvel onglet | noopener noreferrer |
| `ProseImg` | Images avec légendes | alt obligatoire | — |
| `ProseH2+` | Headings avec ancres | aria-label sur lien | — |

---

## Checklist implémentation

```markdown
## ProsePre.vue
- [ ] Props: code, language, filename, highlights, meta, class
- [ ] État machine: idle → copying → copied/error → idle
- [ ] Fallback execCommand pour navigateurs anciens
- [ ] aria-label dynamique sur le bouton
- [ ] Zone aria-live="polite" pour screen readers
- [ ] Timeout 2.5s pour reset état

## ProseA.vue
- [ ] Détection: http://, https://, //
- [ ] Exclusion: mailto:, tel:
- [ ] target="_blank" pour externes
- [ ] rel="noopener noreferrer" pour externes
- [ ] Icône ExternalLink avec aria-hidden="true"
- [ ] <span class="sr-only">(ouvre dans un nouvel onglet)</span>

## ProseImg.vue (optionnel)
- [ ] <figure> wrapper avec légende
- [ ] ClientOnly si erreurs hydration
- [ ] loading="lazy" pour performance
- [ ] Alt text descriptif (fonction + contenu, pas apparence)
- [ ] alt="" pour images décoratives
- [ ] aria-describedby pour images complexes

## DynamicHeading.vue
- [ ] Props: level (1-6), id (optionnel)
- [ ] Computed tag basé sur level
- [ ] Slot pour contenu flexible
- [ ] Usage dans Card avec headingLevel prop
```
