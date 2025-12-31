# Accessibilité clavier WCAG 2.2 pour blogs SSG Nuxt 4

L'accessibilité clavier représente le fondement d'une expérience inclusive sur le web, et le stack Nuxt 4 + Reka UI + TailwindCSS 4 offre des outils natifs particulièrement puissants pour l'implémenter correctement. Ce rapport couvre les pratiques conformes WCAG 2.2 avec des exemples de code prêts à l'emploi pour un blog statique.

Reka UI, successeur de Radix Vue, implémente automatiquement les patterns WAI-ARIA pour le focus trap, la gestion d'Escape et les attributs ARIA, réduisant considérablement le travail manuel. TailwindCSS 4 introduit une configuration CSS-native avec support oklch natif pour des indicateurs de focus à contraste optimal en modes clair et sombre.

---

## 1. Tabindex logique et ordre de focus

Le respect de l'ordre naturel du DOM constitue la règle fondamentale de l'accessibilité clavier. WCAG 2.4.3 (Focus Order) exige que l'ordre de navigation préserve le sens et l'opérabilité du contenu.

### Valeurs tabindex et leurs usages

| Valeur | Comportement | Cas d'utilisation |
|--------|--------------|-------------------|
| **Absent** | Ordre naturel du DOM | Éléments HTML interactifs natifs |
| `tabindex="0"` | Inclus dans l'ordre naturel | Éléments custom rendus interactifs |
| `tabindex="-1"` | Focusable par JS uniquement | Conteneurs de focus, éléments pour navigation programmatique |
| `tabindex > 0` | **❌ INTERDIT** | Jamais - casse l'ordre prévisible |

### Règle : Utiliser les éléments HTML natifs en priorité

**Justification** : Les éléments `<button>`, `<a>`, `<input>` possèdent un comportement clavier natif (Enter, Space) et sont déjà dans l'ordre de tabulation. Les éléments custom nécessitent une implémentation manuelle complète.

```vue
<script setup lang="ts">
// ✅ Aucun tabindex nécessaire pour les éléments natifs
</script>

<template>
  <!-- ✅ Éléments natifs - ordre naturel automatique -->
  <button @click="handleAction">Action</button>
  <a href="/page">Navigation</a>
  <input type="text" v-model="search" />
  
  <!-- ✅ tabindex="0" pour élément custom interactif -->
  <div 
    role="button" 
    tabindex="0" 
    @click="handleClick"
    @keydown.enter="handleClick"
    @keydown.space.prevent="handleClick"
  >
    Bouton personnalisé
  </div>
  
  <!-- ✅ tabindex="-1" pour focus programmatique -->
  <main id="main-content" tabindex="-1">
    <h1 ref="pageTitle" tabindex="-1">{{ title }}</h1>
    <!-- Le focus est dirigé ici après navigation -->
  </main>
</template>
```

### Règle : Gérer le focus après navigation côté client

**Justification** : En SSG avec hydratation, le changement de route ne déclenche pas d'annonce aux lecteurs d'écran. Le focus doit être explicitement déplacé vers le nouveau contenu.

```vue
<!-- layouts/default.vue -->
<script setup lang="ts">
import { ref, watch, nextTick } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const pageTitle = ref<HTMLHeadingElement | null>(null)

// Déplacer le focus après chaque navigation
watch(
  () => route.path,
  async () => {
    await nextTick()
    pageTitle.value?.focus()
  }
)
</script>

<template>
  <SkipLinks />
  <header>
    <MainNavigation />
  </header>
  
  <main id="main-content">
    <h1 ref="pageTitle" tabindex="-1">
      {{ route.meta.title }}
    </h1>
    <slot />
  </main>
</template>
```

### Anti-patterns à éviter

```vue
<template>
  <!-- ❌ tabindex positif - détruit l'ordre prévisible -->
  <button tabindex="1">Premier</button>
  <button tabindex="2">Deuxième</button>
  
  <!-- ❌ Div cliquable sans accessibilité clavier -->
  <div @click="action">Cliquable mais pas accessible</div>
  
  <!-- ❌ tabindex="0" sur élément non-interactif -->
  <p tabindex="0">Ce paragraphe n'a pas besoin d'être focusable</p>
  
  <!-- ❌ Retirer de la tabulation un élément interactif -->
  <button tabindex="-1">Bouton inaccessible au clavier</button>
</template>
```

---

## 2. Skip links (liens d'évitement)

Le skip link permet aux utilisateurs clavier de sauter directement au contenu principal, évitant de traverser la navigation à chaque page. WCAG 2.4.1 (Bypass Blocks) l'exige au niveau A.

### Règle : Premier élément focusable du DOM

**Justification** : Le skip link doit être le tout premier élément dans l'ordre de tabulation pour être immédiatement accessible. Il apparaît visuellement uniquement au focus pour ne pas encombrer l'interface.

```vue
<!-- components/SkipLinks.vue -->
<script setup lang="ts">
interface SkipLink {
  label: string
  targetId: string
}

const links: SkipLink[] = [
  { label: 'Aller au contenu principal', targetId: 'main-content' },
  { label: 'Aller à la navigation', targetId: 'main-nav' },
]

function skipTo(targetId: string) {
  const target = document.getElementById(targetId)
  if (target) {
    // Rendre focusable si nécessaire
    if (!target.hasAttribute('tabindex')) {
      target.setAttribute('tabindex', '-1')
    }
    target.focus()
    target.scrollIntoView({ behavior: 'smooth' })
  }
}
</script>

<template>
  <nav aria-label="Liens d'évitement" class="skip-links">
    <a
      v-for="link in links"
      :key="link.targetId"
      :href="`#${link.targetId}`"
      class="
        sr-only focus:not-sr-only 
        focus:fixed focus:top-4 focus:left-4 focus:z-50
        focus:px-4 focus:py-3 focus:rounded-lg
        focus:bg-indigo-600 focus:text-white
        focus:outline-2 focus:outline-offset-2 focus:outline-indigo-500
        dark:focus:bg-indigo-500 dark:focus:outline-indigo-400
        focus:shadow-lg
        transition-all duration-200
      "
      @click.prevent="skipTo(link.targetId)"
    >
      {{ link.label }}
    </a>
  </nav>
</template>
```

### Classes TailwindCSS 4 pour le masquage accessible

```css
/* sr-only : masqué visuellement, visible aux lecteurs d'écran */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

/* not-sr-only : annule sr-only au focus */
.not-sr-only {
  position: static;
  width: auto;
  height: auto;
  padding: 0;
  margin: 0;
  overflow: visible;
  clip: auto;
  white-space: normal;
}
```

### Positionnement dans la structure Nuxt 4

```vue
<!-- layouts/default.vue -->
<template>
  <!-- Skip links EN PREMIER dans le DOM -->
  <SkipLinks />
  
  <header>
    <nav id="main-nav" role="navigation" aria-label="Navigation principale">
      <MainNavigation />
    </nav>
  </header>
  
  <main id="main-content" role="main" tabindex="-1">
    <slot />
  </main>
  
  <footer role="contentinfo">
    <FooterContent />
  </footer>
</template>
```

### Anti-patterns à éviter

```vue
<template>
  <!-- ❌ Skip link après la navigation - inutile -->
  <header>
    <Navigation />
  </header>
  <a href="#main">Skip to content</a>
  
  <!-- ❌ Skip link toujours visible - encombre l'interface -->
  <a href="#main" class="block p-4 bg-blue-500">Skip</a>
  
  <!-- ❌ Cible sans tabindex="-1" - focus ne fonctionne pas sur tous les navigateurs -->
  <main id="main-content">
    <!-- tabindex="-1" manquant -->
  </main>
</template>
```

---

## 3. Focus trap dans les modales et dialogues

Un focus trap confine la navigation clavier à l'intérieur d'un composant modal, empêchant l'utilisateur de tabber vers le contenu derrière l'overlay. WCAG 2.4.3 et les patterns WAI-ARIA Dialog l'exigent.

### Comportement automatique de Reka UI

**Règle** : Utiliser les composants Reka UI/shadcn-vue plutôt qu'implémenter manuellement le focus trap.

**Justification** : Reka UI gère automatiquement le focus trap, la restauration du focus, les attributs ARIA, et la fermeture Escape. L'implémentation manuelle est source d'erreurs.

| Composant | Focus Trap | Prop Modal | Escape | Restauration Focus |
|-----------|------------|------------|--------|-------------------|
| Dialog | Auto si modal | `modal: true` (défaut) | Auto | Auto vers trigger |
| AlertDialog | Toujours | N/A (toujours modal) | Auto | Auto vers trigger |
| Popover | Si modal=true | `modal: false` (défaut) | Auto | Auto vers trigger |
| DropdownMenu | Auto si modal | `modal: true` (défaut) | Auto | Auto vers trigger |

### Dialog avec shadcn-vue/Reka UI

```vue
<script setup lang="ts">
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
  DialogClose,
} from '@/components/ui/dialog'
import { Button } from '@/components/ui/button'

const isOpen = ref(false)

// Personnaliser le focus initial si nécessaire
function handleOpenAutoFocus(event: Event) {
  event.preventDefault()
  document.getElementById('email-input')?.focus()
}

// Empêcher fermeture si formulaire non sauvegardé
const hasUnsavedChanges = ref(false)

function handleEscapeKeyDown(event: KeyboardEvent) {
  if (hasUnsavedChanges.value) {
    event.preventDefault()
  }
}
</script>

<template>
  <Dialog v-model:open="isOpen">
    <DialogTrigger as-child>
      <Button variant="outline">S'abonner à la newsletter</Button>
    </DialogTrigger>
    
    <DialogContent 
      class="sm:max-w-[425px]"
      @open-auto-focus="handleOpenAutoFocus"
      @escape-key-down="handleEscapeKeyDown"
    >
      <DialogHeader>
        <DialogTitle>Newsletter</DialogTitle>
        <DialogDescription>
          Recevez nos derniers articles directement dans votre boîte mail.
        </DialogDescription>
      </DialogHeader>
      
      <form @submit.prevent="subscribe">
        <input 
          id="email-input"
          type="email" 
          v-model="email"
          class="w-full px-3 py-2 border rounded-md focus:outline-2 focus:outline-indigo-500"
          placeholder="votre@email.com"
        />
      </form>
      
      <DialogFooter>
        <DialogClose as-child>
          <Button variant="outline">Annuler</Button>
        </DialogClose>
        <Button type="submit">S'abonner</Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>
```

### AlertDialog pour actions destructives

```vue
<script setup lang="ts">
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from '@/components/ui/alert-dialog'

// Focus par défaut sur Cancel (sécuritaire pour actions destructives)
// Pour focus sur Action: @open-auto-focus avec preventDefault
</script>

<template>
  <AlertDialog>
    <AlertDialogTrigger as-child>
      <Button variant="destructive">Supprimer le commentaire</Button>
    </AlertDialogTrigger>
    
    <AlertDialogContent>
      <AlertDialogHeader>
        <AlertDialogTitle>Êtes-vous sûr ?</AlertDialogTitle>
        <AlertDialogDescription>
          Cette action est irréversible. Le commentaire sera définitivement supprimé.
        </AlertDialogDescription>
      </AlertDialogHeader>
      
      <AlertDialogFooter>
        <!-- Focus initial ici par défaut (sécuritaire) -->
        <AlertDialogCancel>Annuler</AlertDialogCancel>
        <AlertDialogAction @click="deleteComment">
          Supprimer
        </AlertDialogAction>
      </AlertDialogFooter>
    </AlertDialogContent>
  </AlertDialog>
</template>
```

### FocusScope pour cas custom

Reka UI expose `FocusScope` pour un contrôle manuel si nécessaire :

```vue
<script setup lang="ts">
import { FocusScope } from 'reka-ui'

const showPanel = ref(false)
</script>

<template>
  <FocusScope 
    v-if="showPanel"
    :trapped="true"
    :loop="true"
    @mount-auto-focus="(e) => e.preventDefault()"
    @unmount-auto-focus="(e) => e.preventDefault()"
  >
    <div class="panel">
      <button>Action 1</button>
      <button>Action 2</button>
      <button @click="showPanel = false">Fermer</button>
    </div>
  </FocusScope>
</template>
```

### Anti-patterns à éviter

```vue
<template>
  <!-- ❌ Modal sans focus trap - l'utilisateur peut tabber derrière -->
  <div v-if="isOpen" class="modal-overlay">
    <div class="modal-content">
      <!-- Pas de focus trap -->
    </div>
  </div>
  
  <!-- ❌ Dialog sans aria-modal - lecteur d'écran annonce contenu derrière -->
  <div role="dialog">
    <!-- aria-modal="true" manquant -->
  </div>
  
  <!-- ❌ Pas de restauration du focus à la fermeture -->
  <button @click="closeModal">Fermer</button>
  <!-- L'utilisateur perd sa position dans la page -->
</template>
```

---

## 4. Gestion de la touche Escape

La touche Escape est le mécanisme standard pour fermer les éléments superposés (modales, popovers, menus). WCAG et les patterns WAI-ARIA l'exigent pour tous les composants de type dialog.

### Comportement par défaut de Reka UI

Tous les composants overlay de Reka UI ferment automatiquement avec Escape et émettent un événement `@escape-key-down` pour personnalisation.

```vue
<script setup lang="ts">
import { DialogContent } from 'reka-ui'

// Cas 1: Comportement par défaut (fermeture automatique)
// Rien à faire, Escape ferme le dialog

// Cas 2: Empêcher fermeture conditionnellement
function handleEscape(event: KeyboardEvent) {
  if (formIsDirty.value) {
    event.preventDefault()
    showConfirmation.value = true
  }
}

// Cas 3: Empêcher fermeture complètement
function preventEscape(event: KeyboardEvent) {
  event.preventDefault()
}
</script>

<template>
  <!-- Comportement par défaut -->
  <DialogContent>
    <!-- Escape ferme automatiquement -->
  </DialogContent>
  
  <!-- Fermeture conditionnelle -->
  <DialogContent @escape-key-down="handleEscape">
    <!-- Escape ferme sauf si formulaire modifié -->
  </DialogContent>
  
  <!-- Escape désactivé (utiliser avec parcimonie) -->
  <DialogContent @escape-key-down.prevent>
    <!-- Escape ne ferme pas - fournir un bouton de fermeture visible -->
  </DialogContent>
</template>
```

### Navigation clavier complète des composants Reka UI

**DropdownMenu** :

| Touche | Action |
|--------|--------|
| `Space/Enter` | Ouvre le menu, active l'item |
| `ArrowDown` | Focus item suivant |
| `ArrowUp` | Focus item précédent |
| `ArrowRight/Left` | Ouvre/ferme sous-menu |
| `Esc` | Ferme et retourne au trigger |
| `Home/End` | Premier/dernier item |
| `A-Z` | Typeahead - saute à l'item correspondant |

```vue
<script setup lang="ts">
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu'
</script>

<template>
  <DropdownMenu>
    <DropdownMenuTrigger as-child>
      <Button variant="ghost" size="icon">
        <MoreVertical class="h-4 w-4" />
        <span class="sr-only">Plus d'options</span>
      </Button>
    </DropdownMenuTrigger>
    
    <DropdownMenuContent :side-offset="5" :loop="true">
      <DropdownMenuItem @select="share">
        Partager
      </DropdownMenuItem>
      <DropdownMenuItem @select="bookmark">
        Ajouter aux favoris
      </DropdownMenuItem>
      <DropdownMenuSeparator />
      <DropdownMenuItem @select="report" class="text-red-600">
        Signaler
      </DropdownMenuItem>
    </DropdownMenuContent>
  </DropdownMenu>
</template>
```

### Anti-patterns à éviter

```vue
<script setup lang="ts">
// ❌ Gestion manuelle d'Escape fragile
onMounted(() => {
  document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape') closeModal()
  })
})
// Problèmes : pas de cleanup, pas de gestion des modales imbriquées
</script>

<template>
  <!-- ❌ Désactiver Escape sans alternative de fermeture visible -->
  <DialogContent @escape-key-down.prevent>
    <!-- Pas de bouton de fermeture visible = utilisateur piégé -->
  </DialogContent>
</template>
```

---

## 5. Focus visible (outline)

L'indicateur de focus permet aux utilisateurs clavier de savoir quel élément est actif. WCAG 2.4.7 (Focus Visible) exige un indicateur visible au niveau AA.

### Règle : Utiliser focus-visible plutôt que focus

**Justification** : `focus-visible` s'applique uniquement à la navigation clavier, pas au clic souris. Cela évite les outlines indésirables lors des interactions souris tout en maintenant l'accessibilité clavier.

```vue
<template>
  <!-- ✅ focus-visible pour navigation clavier uniquement -->
  <button 
    class="
      px-6 py-3 rounded-lg
      bg-indigo-600 text-white
      hover:bg-indigo-700
      focus-visible:outline-2 
      focus-visible:outline-offset-2 
      focus-visible:outline-indigo-500
      dark:focus-visible:outline-indigo-400
      active:bg-indigo-800
      transition-all duration-150
    "
  >
    Action principale
  </button>
  
  <!-- ✅ focus-within pour conteneurs -->
  <div 
    class="
      p-4 border-2 border-gray-200 rounded-lg
      focus-within:border-indigo-500 
      focus-within:ring-4 focus-within:ring-indigo-500/20
      dark:border-gray-700 
      dark:focus-within:border-indigo-400
    "
  >
    <input type="text" class="w-full outline-none" />
  </div>
</template>
```

### Outline vs Ring en TailwindCSS 4

| Propriété | CSS | Cas d'utilisation |
|-----------|-----|-------------------|
| `outline-*` | `outline` | **Accessibilité** - compatible forced-colors mode |
| `ring-*` | `box-shadow` | Effets décoratifs |

**Changements TailwindCSS 4** :
- `outline-none` → renommé `outline-hidden` (préserve focus en forced-colors)
- `ring` seul = **1px** (était 3px en v3)

```vue
<template>
  <!-- ✅ Outline pour accessibilité (meilleur support forced-colors) -->
  <button class="focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-500">
    Accessible
  </button>
  
  <!-- Ring pour effets décoratifs -->
  <button class="focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2">
    Décoratif
  </button>
</template>
```

### Configuration CSS-native TailwindCSS 4 avec oklch

```css
/* app.css */
@import "tailwindcss";

@theme {
  /* Tokens de focus en oklch pour contraste optimal */
  --color-focus-ring: oklch(0.55 0.25 260);
  --color-focus-ring-dark: oklch(0.75 0.20 260);
  
  /* Espacement focus */
  --spacing-focus-offset: 2px;
  
  /* Animation focus */
  --ease-focus: cubic-bezier(0.4, 0, 0.2, 1);
  --duration-focus: 150ms;
}
```

### Support dark mode pour indicateurs de focus

```vue
<template>
  <button 
    class="
      focus-visible:outline-2 
      focus-visible:outline-offset-2
      focus-visible:outline-indigo-600
      dark:focus-visible:outline-indigo-400
      transition-shadow duration-150
    "
  >
    Adaptatif clair/sombre
  </button>
  
  <!-- Input avec focus adaptatif -->
  <input 
    type="text"
    class="
      border-2 border-gray-300 rounded-lg px-4 py-2
      focus:border-indigo-500 
      focus:outline-2 focus:outline-indigo-500
      dark:bg-gray-800 dark:border-gray-600
      dark:focus:border-indigo-400 dark:focus:outline-indigo-400
    "
  />
</template>
```

### Anti-patterns à éviter

```css
/* ❌ Supprimer l'outline sans alternative */
button:focus {
  outline: none;
}

/* ❌ Outline avec contraste insuffisant */
button:focus {
  outline: 1px solid #cccccc; /* Contraste < 3:1 */
}
```

```vue
<template>
  <!-- ❌ focus au lieu de focus-visible (outline au clic souris) -->
  <button class="focus:outline-2 focus:outline-blue-500">
    Outline même au clic
  </button>
</template>
```

---

## 6. Critères WCAG 2.2 spécifiques

WCAG 2.2 (octobre 2023) introduit de nouveaux critères particulièrement pertinents pour l'accessibilité clavier et les interfaces interactives.

### 2.4.11 Focus Not Obscured (Minimum) - Niveau AA

**Critère** : L'élément recevant le focus clavier ne doit pas être **entièrement** masqué par du contenu créé par l'auteur.

**Technique CSS recommandée** :

```css
/* Empêcher les headers/footers sticky de masquer le focus */
html {
  scroll-padding-top: 80px;    /* hauteur du header sticky */
  scroll-padding-bottom: 60px; /* hauteur du footer sticky */
}
```

```vue
<template>
  <!-- ✅ Cookie banner modal qui prend le focus -->
  <Dialog v-model:open="showCookieBanner">
    <DialogContent>
      <!-- L'utilisateur doit interagir avant de continuer -->
    </DialogContent>
  </Dialog>
  
  <!-- ❌ Cookie banner non-modal qui masque le focus -->
  <div class="fixed bottom-0 w-full bg-gray-800 p-4">
    <!-- Peut masquer les éléments focusés en bas de page -->
  </div>
</template>
```

### 2.4.13 Focus Appearance - Niveau AAA

**Critère** : L'indicateur de focus doit avoir :
- Une surface au moins égale à un **périmètre de 2 CSS pixels** autour du composant
- Un **contraste de 3:1** entre les états focused/unfocused

```vue
<template>
  <!-- ✅ Outline 2px avec contraste suffisant -->
  <button 
    class="
      focus-visible:outline-2 
      focus-visible:outline-offset-2 
      focus-visible:outline-indigo-600
    "
  >
    Conforme 2.4.13
  </button>
  
  <!-- ✅ Indicateur deux couleurs (haute visibilité) -->
  <button 
    class="
      focus-visible:outline-[3px] 
      focus-visible:outline-black
      focus-visible:shadow-[0_0_0_6px_white]
    "
  >
    Haute visibilité
  </button>
</template>
```

### 2.5.8 Target Size (Minimum) - Niveau AA

**Critère** : Les cibles de pointeur doivent mesurer au moins **24 × 24 CSS pixels**, sauf si :
- L'espacement garantit qu'un cercle de 24px centré n'intersecte pas d'autre cible
- Une alternative équivalente existe sur la même page
- La cible est inline dans du texte

```vue
<template>
  <!-- ✅ Bouton avec taille minimale -->
  <button class="min-w-6 min-h-6 px-4 py-2">
    Action
  </button>
  
  <!-- ✅ Icône 16x16 avec padding pour atteindre 24x24 -->
  <button class="p-1"> <!-- 16px + 8px padding = 24px -->
    <Icon class="w-4 h-4" />
    <span class="sr-only">Paramètres</span>
  </button>
  
  <!-- ✅ Liens texte inline - exception autorisée -->
  <p>
    Consultez notre <a href="/privacy">politique de confidentialité</a> 
    pour plus d'informations.
  </p>
</template>

<style>
/* Règle globale pour les cibles interactives */
button, 
a:not([class*="inline"]), 
input[type="checkbox"], 
input[type="radio"] {
  min-width: 24px;
  min-height: 24px;
}
</style>
```

### 2.5.7 Dragging Movements - Niveau AA

**Critère** : Toute fonctionnalité utilisant le glissement (drag) doit avoir une alternative par pointeur simple.

```vue
<script setup lang="ts">
// Pour un carrousel d'images
const currentSlide = ref(0)

function nextSlide() {
  currentSlide.value = (currentSlide.value + 1) % slides.length
}

function prevSlide() {
  currentSlide.value = (currentSlide.value - 1 + slides.length) % slides.length
}
</script>

<template>
  <div class="carousel" role="region" aria-label="Carrousel d'images">
    <!-- ✅ Boutons de navigation en alternative au swipe -->
    <button 
      @click="prevSlide"
      class="carousel-btn prev"
      aria-label="Image précédente"
    >
      <ChevronLeft />
    </button>
    
    <div class="carousel-slides">
      <!-- Slides avec drag optionnel -->
    </div>
    
    <button 
      @click="nextSlide"
      class="carousel-btn next"
      aria-label="Image suivante"
    >
      <ChevronRight />
    </button>
    
    <!-- ✅ Indicateurs cliquables -->
    <div class="carousel-dots" role="tablist">
      <button
        v-for="(_, index) in slides"
        :key="index"
        role="tab"
        :aria-selected="currentSlide === index"
        :aria-label="`Aller à l'image ${index + 1}`"
        @click="currentSlide = index"
        class="w-3 h-3 rounded-full"
        :class="currentSlide === index ? 'bg-indigo-600' : 'bg-gray-300'"
      />
    </div>
  </div>
</template>
```

---

## 7. Roving tabindex pour widgets complexes

Le pattern roving tabindex permet la navigation par flèches dans un groupe d'éléments, avec un seul élément tabbable à la fois. Utilisé pour toolbars, tabs, menus.

### Utilisation de RovingFocusGroup (Reka UI)

```vue
<script setup lang="ts">
import { RovingFocusGroup, RovingFocusItem } from 'reka-ui'

const tools = [
  { id: 'bold', label: 'Gras', icon: Bold },
  { id: 'italic', label: 'Italique', icon: Italic },
  { id: 'underline', label: 'Souligné', icon: Underline },
]
</script>

<template>
  <RovingFocusGroup 
    orientation="horizontal" 
    :loop="true"
    as="div"
    role="toolbar"
    aria-label="Outils de formatage"
  >
    <RovingFocusItem 
      v-for="tool in tools" 
      :key="tool.id"
      as-child
    >
      <button
        :aria-label="tool.label"
        class="p-2 hover:bg-gray-100 focus-visible:outline-2 focus-visible:outline-indigo-500"
      >
        <component :is="tool.icon" class="w-4 h-4" />
      </button>
    </RovingFocusItem>
  </RovingFocusGroup>
</template>
```

---

## Checklist pour Claude Code

```markdown
## Checklist Accessibilité Clavier - Nuxt 4 + Reka UI + TailwindCSS 4

### Structure et navigation
- [ ] Skip link comme premier élément focusable du DOM
- [ ] Skip link utilise `sr-only focus:not-sr-only` avec positionnement au focus
- [ ] Landmarks HTML sémantiques : `<main>`, `<nav>`, `<header>`, `<footer>`
- [ ] `<main id="main-content" tabindex="-1">` pour cible du skip link
- [ ] Focus reset après navigation client-side (`watch` sur `route.path`)

### Tabindex
- [ ] Jamais de `tabindex` positif (> 0)
- [ ] `tabindex="0"` uniquement pour éléments custom interactifs
- [ ] `tabindex="-1"` pour éléments focusables par JS uniquement
- [ ] Éléments natifs (`<button>`, `<a>`, `<input>`) sans tabindex

### Focus visible
- [ ] `focus-visible:` (pas `focus:`) pour indicateurs visuels
- [ ] Outline de **2px minimum** pour WCAG 2.4.13
- [ ] `outline-offset-2` pour séparation visuelle
- [ ] Contraste **3:1 minimum** entre états focused/unfocused
- [ ] Support dark mode : `dark:focus-visible:outline-{color}`
- [ ] Jamais `outline: none` sans alternative visible

### Composants modaux (Dialog, AlertDialog, Popover)
- [ ] Utiliser composants Reka UI/shadcn-vue (focus trap automatique)
- [ ] `aria-modal="true"` (automatique avec Reka UI)
- [ ] `aria-labelledby` vers DialogTitle
- [ ] `aria-describedby` vers DialogDescription
- [ ] Focus initial sur premier élément interactif ou personnalisé via `@open-auto-focus`
- [ ] Restauration du focus au trigger à la fermeture (automatique)

### Touche Escape
- [ ] Fermeture automatique (comportement Reka UI par défaut)
- [ ] Si Escape désactivé (`@escape-key-down.prevent`), fournir bouton de fermeture visible
- [ ] AlertDialog : fermeture Escape active par défaut

### Taille des cibles (WCAG 2.5.8)
- [ ] Cibles interactives **24×24px minimum**
- [ ] Icônes < 24px : ajouter padding pour atteindre 24px
- [ ] Espacement suffisant si cibles plus petites

### Focus non masqué (WCAG 2.4.11)
- [ ] `scroll-padding-top/bottom` pour headers/footers sticky
- [ ] Cookie banners modaux (ou qui ne masquent pas le focus)

### Alternatives au drag (WCAG 2.5.7)
- [ ] Carrousels : boutons prev/next + indicateurs cliquables
- [ ] Listes réordonnables : boutons monter/descendre
- [ ] Upload fichiers : bouton "Parcourir" en plus du drag-and-drop

### Widgets complexes
- [ ] Roving tabindex pour toolbars, tabs, menus
- [ ] Navigation par flèches (ArrowUp/Down/Left/Right)
- [ ] Home/End pour premier/dernier élément
- [ ] Typeahead pour menus (A-Z saute à l'item)

### TailwindCSS 4 spécifique
- [ ] `outline-hidden` au lieu de `outline-none` (v4)
- [ ] `ring-3` au lieu de `ring` seul pour équivalent v3
- [ ] Configuration @theme pour tokens de focus oklch
- [ ] `forced-color-adjust-auto` préservé pour High Contrast Mode
```

---

## Exemple complet de layout accessible

```vue
<!-- layouts/default.vue -->
<script setup lang="ts">
import { ref, watch, nextTick } from 'vue'
import { useRoute } from 'vue-router'
import SkipLinks from '~/components/SkipLinks.vue'
import MainNavigation from '~/components/MainNavigation.vue'

const route = useRoute()
const pageTitle = ref<HTMLHeadingElement | null>(null)

// Reset focus après navigation
watch(
  () => route.path,
  async () => {
    await nextTick()
    pageTitle.value?.focus()
  }
)
</script>

<template>
  <!-- Skip links - PREMIER dans le DOM -->
  <SkipLinks />
  
  <header class="sticky top-0 z-40 bg-white dark:bg-gray-900 border-b">
    <nav 
      id="main-nav" 
      role="navigation" 
      aria-label="Navigation principale"
      class="container mx-auto px-4"
    >
      <MainNavigation />
    </nav>
  </header>
  
  <main 
    id="main-content" 
    role="main"
    tabindex="-1"
    class="container mx-auto px-4 py-8 min-h-screen"
  >
    <h1 
      ref="pageTitle" 
      tabindex="-1"
      class="text-3xl font-bold mb-6 outline-none"
    >
      {{ route.meta.title || 'Page' }}
    </h1>
    
    <slot />
  </main>
  
  <footer 
    role="contentinfo"
    class="bg-gray-100 dark:bg-gray-800 py-8"
  >
    <div class="container mx-auto px-4">
      <FooterContent />
    </div>
  </footer>
</template>

<style>
/* Scroll padding pour Focus Not Obscured */
html {
  scroll-padding-top: 80px;
  scroll-padding-bottom: 40px;
}

/* Taille minimale des cibles */
button:not(.inline-btn),
a:not(.inline-link),
[role="button"] {
  min-width: 24px;
  min-height: 24px;
}
</style>
```

Ce rapport fournit une référence complète pour implémenter l'accessibilité clavier conforme WCAG 2.2 dans un blog SSG Nuxt 4. Les composants Reka UI/shadcn-vue automatisent la majorité des patterns complexes (focus trap, restauration, Escape), permettant de se concentrer sur la structure sémantique et les indicateurs de focus.