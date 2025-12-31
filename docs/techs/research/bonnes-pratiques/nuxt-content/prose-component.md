# Prose Components Nuxt Content v3 : guide complet 2025

La personnalisation des Prose Components dans Nuxt Content v3.10+ nécessite une approche fondamentalement différente de la v2 : les composants doivent être enregistrés globalement, placés dans `components/content/` (ou `components/mdc/` avec le suffixe `.global.vue`), et la consolidation ProseCode/ProsePre requiert une migration explicite. Ce guide couvre l'ensemble des bonnes pratiques actuelles pour votre stack Nuxt 4.2.x, TailwindCSS 4.1.x et shadcn-vue.

---

## Structure des fichiers et conventions de nommage

Les Prose Components font partie du module `@nuxtjs/mdc`, dépendance de `@nuxt/content`. Pour personnaliser un composant, créez un fichier portant le même nom dans le répertoire `components/content/`. Cette convention remplace automatiquement le composant par défaut.

**Arborescence recommandée pour Nuxt 4.2.x avec app/ directory :**

```
app/
├── components/
│   ├── content/           # Prose components personnalisés
│   │   ├── ProseA.vue
│   │   ├── ProsePre.vue
│   │   ├── ProseCode.vue
│   │   └── ProseImg.vue
│   └── ui/                # Composants shadcn-vue
│       ├── button/
│       └── badge/
```

**Alternative avec MDC direct :** Pour une meilleure compatibilité SSG, utilisez `components/mdc/` avec le suffixe `.global.vue` :

```
components/mdc/ProsePre.global.vue
```

**Liste complète des Prose Components disponibles :**

| Tag HTML | Composant | Tag HTML | Composant |
|----------|-----------|----------|-----------|
| `p` | `ProseP` | `table` | `ProseTable` |
| `h1`-`h6` | `ProseH1`-`ProseH6` | `thead/tbody` | `ProseThead/ProseTbody` |
| `ul/ol/li` | `ProseUl/ProseOl/ProseLi` | `tr/th/td` | `ProseTr/ProseTh/ProseTd` |
| `blockquote` | `ProseBlockquote` | `a` | `ProseA` |
| `pre` | `ProsePre` | `img` | `ProseImg` |
| `code` | `ProseCode` | `em/strong` | `ProseEm/ProseStrong` |

---

## Configuration nuxt.config.ts pour Nuxt Content v3

La configuration correcte est critique pour éviter les problèmes de résolution de composants et les erreurs SSG. Voici la configuration complète validée pour votre stack :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@nuxt/content',
    'shadcn-nuxt',
    '@nuxtjs/tailwindcss'
  ],

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
          theme: {
            default: 'github-light',
            dark: 'github-dark'
          },
          langs: ['javascript', 'typescript', 'vue', 'css', 'html', 'bash', 'json']
        },
        toc: { depth: 3 }
      }
    },
    // Pour Cloudflare Pages SSG
    database: {
      type: 'd1',
      binding: 'DB'
    }
  },

  // Configuration shadcn-vue
  shadcn: {
    prefix: '',
    componentDir: './components/ui'
  }
})
```

**Configuration `content.renderer.alias` :** Cette option permet de mapper des tags HTML vers des composants personnalisés sans créer de fichiers dans `components/content/`. Cependant, pour la plupart des cas d'usage, le pattern de surcharge par fichier est recommandé car plus explicite et maintenable.

---

## Implémentation ProsePre.vue avec bouton de copie

Le composant `ProsePre` gère les blocs de code (triple backticks). Voici l'implémentation complète avec **bouton de copie accessible**, **fallback navigateur**, et **intégration shadcn-vue** :

```vue
<!-- components/content/ProsePre.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue'
import { Button } from '@/components/ui/button'
import { Check, Copy, X } from 'lucide-vue-next'

// Props fournis automatiquement par Nuxt Content/MDC
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

**Props reçus de Nuxt Content :** Le prop `code` contient le texte brut du code (sans coloration syntaxique), essentiel pour la fonctionnalité de copie. Les props `language`, `filename`, `highlights` et `meta` sont extraits de la syntaxe Markdown étendue comme ` ```ts [file.ts]{1,3-5} meta-info `.

---

## Implémentation ProseA.vue pour liens externes

La détection automatique des liens externes et l'ajout des attributs de sécurité appropriés est une exigence fondamentale. Voici l'implémentation complète avec **indicateur visuel**, **accessibilité** et **intégration NuxtLink** :

```vue
<!-- components/content/ProseA.vue -->
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

**Points de sécurité critiques :** L'attribut `rel="noopener noreferrer"` est obligatoire avec `target="_blank"` pour prévenir les attaques de tabnabbing. NuxtLink gère automatiquement ces attributs quand `external={true}`, mais l'expliciter garantit la compatibilité. Pour du contenu généré par les utilisateurs, ajoutez également `rel="nofollow"`.

---

## Différences majeures entre Nuxt Content v2 et v3

La migration v2 → v3 comporte plusieurs **breaking changes critiques** concernant les Prose Components :

**Consolidation ProseCode/ProsePre :** C'est le changement le plus impactant. En v2, trois composants existaient : `ProsePre`, `ProseCode`, et `ProseCodeInline`. En v3, seuls deux composants subsistent. Les backticks simples (`` ` ``) mappent vers `ProseCode` (anciennement `ProseCodeInline`), et les triples backticks (` ``` `) mappent vers `ProsePre`.

**Action de migration :**
1. Déplacer la logique de votre `ProseCode` v2 vers `ProsePre`
2. Renommer `ProseCodeInline` en `ProseCode`

**Enregistrement global supprimé :** Les composants dans `components/content/` ne sont **plus automatiquement enregistrés globalement**. Configuration explicite requise :

```typescript
components: {
  global: true,
  dirs: [{ path: '~/components/content', global: true }]
}
```

**Remplacement de ContentSlot :** Les composants `<ContentSlot>` et `<MDCSlot>` n'existent plus. Utilisez le slot Vue natif avec l'attribut `mdc-unwrap` :

```vue
<!-- v2 (obsolète) -->
<ContentSlot :use="$slots.default" unwrap="p" />

<!-- v3 (correct) -->
<slot mdc-unwrap="p" />
```

**Changements API :** Les fonctions `queryContent()` deviennent `queryCollection()`, `fetchContentNavigation()` devient `queryCollectionNavigation()`, et le type `NavItem` est remplacé par `ContentNavigationItem`.

---

## Pièges courants et solutions validées

**Erreur d'hydration avec wrappers :** Ajouter des éléments wrapper aux composants inline cause des mismatches d'hydration SSR :

```vue
<!-- ❌ PROVOQUE UNE ERREUR D'HYDRATION -->
<template>
  <figure>
    <img :src="src" :alt="alt" />
    <figcaption>{{ alt }}</figcaption>
  </figure>
</template>

<!-- ✅ SOLUTION: Garder la structure minimale ou utiliser ClientOnly -->
<template>
  <ClientOnly>
    <figure>
      <img :src="src" :alt="alt" />
      <figcaption>{{ alt }}</figcaption>
    </figure>
  </ClientOnly>
</template>
```

**Composants non rendus après `nuxi generate` :** Si vos composants fonctionnent en développement mais pas après génération statique, utilisez le suffixe `.global.vue` et placez-les dans `components/mdc/`.

**Warning "Two component files resolving to same name" :** Ce warning est **attendu** lors de la surcharge de composants prose. Il indique que votre composant personnalisé remplace bien celui par défaut.

**Ordre des modules avec Nuxt UI :** Si vous utilisez `@nuxt/ui`, enregistrez `@nuxt/content` **après** `@nuxt/ui` dans le tableau des modules.

**Cloudflare Pages et base de données :** Nuxt Content v3 utilise SQLite. Pour Cloudflare Pages, configurez D1 :

```typescript
content: {
  database: {
    type: 'd1',
    binding: 'DB' // Nom du binding dans wrangler.toml
  }
}
```

---

## Checklist actionnable pour Claude Code

```markdown
## Setup initial
- [ ] Créer `components/content/` dans le répertoire app/
- [ ] Configurer `components.global: true` dans nuxt.config.ts
- [ ] Vérifier l'ordre des modules (content après ui si applicable)

## ProsePre.vue (blocs de code)
- [ ] Définir les props: code, language, filename, highlights, meta, class
- [ ] Implémenter navigator.clipboard.writeText() avec try/catch
- [ ] Ajouter fallback execCommand pour navigateurs anciens
- [ ] États visuels: idle → copying → copied/error → idle (2.5s timeout)
- [ ] aria-label dynamique sur le bouton
- [ ] Zone aria-live="polite" pour annonces screen reader
- [ ] aria-hidden="true" sur les icônes SVG

## ProseA.vue (liens)
- [ ] Détecter externe: href.startsWith('http') || href.startsWith('//')
- [ ] Exclure mailto: et tel: de la détection externe
- [ ] Ajouter target="_blank" pour liens externes
- [ ] Ajouter rel="noopener noreferrer" pour liens externes
- [ ] Icône indicateur avec aria-hidden="true"
- [ ] Texte sr-only "(ouvre dans un nouvel onglet)"

## SSG Cloudflare Pages
- [ ] Configurer database.type: 'd1' avec binding approprié
- [ ] Utiliser suffixe .global.vue si problèmes de résolution
- [ ] Tester `nuxi generate` avant déploiement

## Migration v2 → v3
- [ ] Déplacer logique ProseCode v2 → ProsePre v3
- [ ] Renommer ProseCodeInline → ProseCode
- [ ] Remplacer <ContentSlot> par <slot mdc-unwrap="p" />
- [ ] Mettre à jour imports de types (NavItem → ContentNavigationItem)
```

---

## Conclusion

La personnalisation des Prose Components dans Nuxt Content v3.10+ repose sur trois piliers : **l'enregistrement global explicite** des composants, la **compréhension de la consolidation ProseCode/ProsePre**, et une **attention particulière à la compatibilité SSG**. L'intégration de shadcn-vue s'effectue naturellement via l'import des composants UI dans vos prose components personnalisés, sans configuration particulière.

Les patterns présentés ici pour le bouton de copie et la gestion des liens externes suivent les standards d'accessibilité WCAG et les meilleures pratiques de sécurité web actuelles. La combinaison de VueUse (`useClipboard`) pour les cas complexes et de l'implémentation manuelle pour le contrôle total offre une flexibilité optimale selon vos besoins.