# Accessibilité WCAG 2.2 avec Reka UI et shadcn-vue pour Nuxt 4

Reka UI 2.7.0 fournit une couche d'accessibilité WAI-ARIA complète qui gère automatiquement **90% des exigences d'accessibilité** pour les composants Dialog, Menu, Tabs et Accordion. Le travail principal consiste à respecter les nouveaux critères WCAG 2.2 — notamment la **taille minimale des cibles (24×24px)**, la **visibilité du focus non obscurci**, et la gestion de `prefers-reduced-motion` — tout en évitant les anti-patterns qui cassent l'accessibilité héritée. En mode SSG avec Nuxt 4, l'utilisation de `useId()` (Vue 3.5+) pour les associations ARIA et la configuration correcte des Teleport/Portal sont essentielles pour éviter les erreurs d'hydration.

---

## Nouveautés WCAG 2.2 impactant vos composants

WCAG 2.2, publié le **5 octobre 2023**, introduit 9 nouveaux critères de succès. Quatre d'entre eux affectent directement les composants UI interactifs de votre projet.

**2.5.8 Target Size (Minimum) - Niveau AA** impose une taille minimale de **24×24 CSS pixels** pour tous les éléments interactifs. Cela concerne les boutons de fermeture des dialogs, les items de menu, les onglets tabs et les headers d'accordion. L'exception "Spacing" permet des cibles plus petites si un cercle de 24px centré sur chaque cible ne chevauche pas d'autres cibles. En pratique, appliquez `min-width: 24px; min-height: 24px` ou augmentez la zone cliquable via padding.

**2.4.11 Focus Not Obscured (AA)** exige que l'élément focusé ne soit jamais **entièrement masqué** par du contenu — sticky headers, bannières cookies, ou dialogs non-modaux. La technique CSS `scroll-padding` évite ce problème. Pour les dialogs modaux avec Reka UI, le focus trap intégré garantit que le focus reste dans le dialog, évitant ainsi ce critère. Les dialogs non-modaux (`modal={false}`) nécessitent une attention particulière.

**2.4.13 Focus Appearance (AAA)** définit des critères précis pour l'indicateur de focus : périmètre d'au moins **2 CSS pixels** et contraste de **3:1** entre états focusé et non-focusé. shadcn-vue utilise par défaut `focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2`, ce qui satisfait généralement ce critère.

**2.5.7 Dragging Movements (AA)** impose une alternative au drag-and-drop pour toute fonctionnalité de glissement. Si vos menus ou tabs sont réorganisables par drag, ajoutez des boutons haut/bas comme alternative.

---

## API Reka UI 2.7.0 : accessibilité intégrée

Reka UI (anciennement Radix Vue) implémente automatiquement les patterns WAI-ARIA. Les variables CSS utilisent désormais le préfixe `--reka-*` au lieu de `--radix-*`.

### Dialog : focus trap et associations ARIA automatiques

Le composant Dialog gère nativement le **focus trap en mode modal**, la fermeture via Escape, et les associations `aria-labelledby`/`aria-describedby`. Le retour du focus au trigger à la fermeture est automatique.

```vue
<script setup lang="ts">
import {
  DialogRoot, DialogTrigger, DialogPortal, DialogOverlay,
  DialogContent, DialogTitle, DialogDescription, DialogClose
} from 'reka-ui'

const isOpen = ref(false)
</script>

<template>
  <DialogRoot v-model:open="isOpen">
    <DialogTrigger as-child>
      <Button>Ouvrir le profil</Button>
    </DialogTrigger>
    <DialogPortal>
      <DialogOverlay class="fixed inset-0 bg-black/50" />
      <DialogContent 
        class="fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 
               bg-white p-6 rounded-lg shadow-xl w-[425px] max-w-[90vw]"
        @escape-key-down="isOpen = false"
      >
        <!-- aria-labelledby généré automatiquement -->
        <DialogTitle class="text-lg font-semibold">
          Modifier le profil
        </DialogTitle>
        <!-- aria-describedby généré automatiquement -->
        <DialogDescription class="text-sm text-gray-600 mt-2">
          Apportez des modifications à votre profil ici.
        </DialogDescription>
        
        <form class="mt-4 space-y-4">
          <!-- Contenu du formulaire -->
        </form>
        
        <!-- Bouton fermeture : respecter WCAG 2.5.8 (≥24×24px) -->
        <DialogClose as-child>
          <button 
            class="absolute top-4 right-4 w-8 h-8 flex items-center justify-center
                   rounded-full hover:bg-gray-100 focus-visible:ring-2"
            aria-label="Fermer"
          >
            <span aria-hidden="true">×</span>
          </button>
        </DialogClose>
      </DialogContent>
    </DialogPortal>
  </DialogRoot>
</template>
```

**Props critiques pour l'accessibilité** : `modal` (défaut `true`) active le focus trap et rend le contenu extérieur `aria-hidden`. Les events `@openAutoFocus` et `@closeAutoFocus` permettent de personnaliser la gestion du focus — par exemple pour focus un champ spécifique à l'ouverture :

```vue
<DialogContent 
  @open-auto-focus="(e) => { 
    e.preventDefault(); 
    emailInput?.focus() 
  }"
>
```

### Menus : navigation clavier et typeahead

Les DropdownMenu et ContextMenu implémentent le pattern WAI-ARIA Menu Button avec **typeahead** (recherche par frappe), **roving tabindex**, et navigation complète via flèches.

```vue
<script setup lang="ts">
import {
  DropdownMenuRoot, DropdownMenuTrigger, DropdownMenuPortal,
  DropdownMenuContent, DropdownMenuItem, DropdownMenuSeparator,
  DropdownMenuSub, DropdownMenuSubTrigger, DropdownMenuSubContent,
  DropdownMenuCheckboxItem, DropdownMenuRadioGroup, DropdownMenuRadioItem
} from 'reka-ui'

const showBookmarks = ref(true)
const theme = ref('light')
</script>

<template>
  <DropdownMenuRoot>
    <DropdownMenuTrigger as-child>
      <Button variant="outline" class="min-h-[44px]">
        Options
        <ChevronDown class="ml-2 h-4 w-4" />
      </Button>
    </DropdownMenuTrigger>
    
    <DropdownMenuPortal>
      <DropdownMenuContent 
        class="min-w-[220px] bg-white rounded-md shadow-lg p-1"
        :side-offset="5"
        align="start"
      >
        <!-- MenuItem standard - WCAG 2.5.8 : min-height 24px -->
        <DropdownMenuItem 
          class="px-2 py-2 min-h-[32px] rounded cursor-pointer
                 data-[highlighted]:bg-gray-100 outline-none"
          @select="handleProfile"
        >
          Profil
        </DropdownMenuItem>
        
        <DropdownMenuSeparator class="h-px bg-gray-200 my-1" />
        
        <!-- CheckboxItem avec v-model -->
        <DropdownMenuCheckboxItem 
          v-model="showBookmarks"
          class="px-2 py-2 min-h-[32px] rounded cursor-pointer
                 data-[highlighted]:bg-gray-100 outline-none flex items-center"
        >
          <span class="w-4 mr-2">
            <Check v-if="showBookmarks" class="h-4 w-4" />
          </span>
          Afficher favoris
        </DropdownMenuCheckboxItem>
        
        <!-- Sous-menu -->
        <DropdownMenuSub>
          <DropdownMenuSubTrigger 
            class="px-2 py-2 min-h-[32px] rounded cursor-pointer
                   data-[highlighted]:bg-gray-100 outline-none flex justify-between"
          >
            Thème
            <ChevronRight class="h-4 w-4" />
          </DropdownMenuSubTrigger>
          <DropdownMenuPortal>
            <DropdownMenuSubContent class="min-w-[160px] bg-white rounded-md shadow-lg p-1">
              <DropdownMenuRadioGroup v-model="theme">
                <DropdownMenuRadioItem value="light" class="px-2 py-2 min-h-[32px]">
                  Clair
                </DropdownMenuRadioItem>
                <DropdownMenuRadioItem value="dark" class="px-2 py-2 min-h-[32px]">
                  Sombre
                </DropdownMenuRadioItem>
              </DropdownMenuRadioGroup>
            </DropdownMenuSubContent>
          </DropdownMenuPortal>
        </DropdownMenuSub>
      </DropdownMenuContent>
    </DropdownMenuPortal>
  </DropdownMenuRoot>
</template>
```

**Navigation clavier automatique** : Arrow Up/Down navigue entre items, Arrow Right/Left ouvre/ferme les sous-menus, Home/End saute au premier/dernier item, Escape ferme le menu, et la **frappe rapide** (typeahead) permet de naviguer vers un item en tapant son début.

### Tabs : orientation et activation automatique

Le composant Tabs gère automatiquement les associations `aria-controls`/`aria-selected` et adapte la navigation clavier selon l'orientation.

```vue
<script setup lang="ts">
import { TabsRoot, TabsList, TabsTrigger, TabsContent, TabsIndicator } from 'reka-ui'

const activeTab = ref('account')
</script>

<template>
  <!-- orientation="vertical" change la navigation : ↑↓ au lieu de ←→ -->
  <TabsRoot v-model="activeTab" orientation="horizontal">
    <TabsList 
      class="flex border-b border-gray-200" 
      aria-label="Paramètres du compte"
    >
      <!-- TabsTrigger : min 24×24px pour WCAG 2.5.8 -->
      <TabsTrigger 
        value="account"
        class="px-4 py-3 min-h-[44px] border-b-2 border-transparent
               data-[state=active]:border-blue-500 data-[state=active]:text-blue-600
               focus-visible:ring-2 focus-visible:ring-offset-2 outline-none"
      >
        Compte
      </TabsTrigger>
      <TabsTrigger 
        value="security"
        class="px-4 py-3 min-h-[44px] border-b-2 border-transparent
               data-[state=active]:border-blue-500 data-[state=active]:text-blue-600
               focus-visible:ring-2 focus-visible:ring-offset-2 outline-none"
      >
        Sécurité
      </TabsTrigger>
      <!-- Indicateur animé optionnel -->
      <TabsIndicator class="absolute bottom-0 left-0 h-0.5 bg-blue-500 transition-all" />
    </TabsList>
    
    <!-- TabsContent avec aria-labelledby automatique -->
    <TabsContent value="account" class="p-4">
      <h2 class="text-lg font-medium">Paramètres du compte</h2>
      <!-- Contenu -->
    </TabsContent>
    <TabsContent value="security" class="p-4">
      <h2 class="text-lg font-medium">Paramètres de sécurité</h2>
    </TabsContent>
  </TabsRoot>
</template>
```

**Props clés** : `activationMode` contrôle l'activation — `automatic` (défaut) active le tab au focus, `manual` requiert Enter/Space. Pour les formulaires longs dans les tabs, préférez `manual` pour éviter la perte de données lors de la navigation clavier. `unmountOnHide` (défaut `true`) démonte le contenu caché pour les performances.

### Accordion : single vs multiple et animations

L'Accordion gère `aria-expanded` et `aria-controls` automatiquement, avec support des modes single/multiple.

```vue
<script setup lang="ts">
import {
  AccordionRoot, AccordionItem, AccordionHeader, 
  AccordionTrigger, AccordionContent
} from 'reka-ui'
</script>

<template>
  <!-- type="multiple" permet plusieurs items ouverts simultanément -->
  <AccordionRoot 
    type="single" 
    collapsible 
    default-value="item-1"
    class="w-full"
  >
    <AccordionItem value="item-1" class="border-b">
      <!-- AccordionHeader wrapper le trigger dans un heading -->
      <AccordionHeader as="h3">
        <AccordionTrigger 
          class="flex w-full justify-between items-center py-4 px-2 min-h-[48px]
                 text-left font-medium hover:bg-gray-50
                 focus-visible:ring-2 focus-visible:ring-inset outline-none"
        >
          Informations produit
          <ChevronDown 
            class="h-4 w-4 transition-transform duration-200
                   [[data-state=open]_&]:rotate-180" 
            aria-hidden="true"
          />
        </AccordionTrigger>
      </AccordionHeader>
      <AccordionContent 
        class="overflow-hidden data-[state=open]:animate-accordion-down
               data-[state=closed]:animate-accordion-up"
      >
        <div class="pb-4 px-2">
          <p>Notre produit phare combine technologie de pointe...</p>
        </div>
      </AccordionContent>
    </AccordionItem>
    
    <AccordionItem value="item-2" class="border-b">
      <AccordionHeader as="h3">
        <AccordionTrigger class="...">Livraison</AccordionTrigger>
      </AccordionHeader>
      <AccordionContent class="...">
        <div class="pb-4 px-2">Nous livrons dans le monde entier...</div>
      </AccordionContent>
    </AccordionItem>
  </AccordionRoot>
</template>

<style>
/* Animation accessible avec CSS variables Reka UI */
@keyframes accordion-down {
  from { height: 0; }
  to { height: var(--reka-accordion-content-height); }
}
@keyframes accordion-up {
  from { height: var(--reka-accordion-content-height); }
  to { height: 0; }
}

/* Respecter prefers-reduced-motion */
@media (prefers-reduced-motion: reduce) {
  .animate-accordion-down,
  .animate-accordion-up {
    animation: none;
  }
}
</style>
```

---

## Patterns shadcn-vue et préservation de l'accessibilité

shadcn-vue encapsule Reka UI avec le pattern `useForwardPropsEmits` qui transmet intégralement les props et events, préservant l'accessibilité. Le point critique est l'utilisation correcte de `as-child`.

**Pattern as-child obligatoire pour les triggers** évite le double wrapping qui casse la sémantique :

```vue
<!-- ✅ CORRECT : as-child fusionne les props sur le Button -->
<DialogTrigger as-child>
  <Button variant="outline">Ouvrir</Button>
</DialogTrigger>

<!-- ❌ INCORRECT : crée un button dans un button -->
<DialogTrigger>
  <Button>Ouvrir</Button>
</DialogTrigger>
```

**Pattern Dialog + Form avec VeeValidate** :

```vue
<script setup lang="ts">
import { toTypedSchema } from '@vee-validate/zod'
import { useForm } from 'vee-validate'
import * as z from 'zod'
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from '@/components/ui/dialog'

const schema = toTypedSchema(z.object({
  email: z.string().email('Email invalide'),
  name: z.string().min(2, 'Minimum 2 caractères')
}))

const { handleSubmit, errors, defineField } = useForm({ validationSchema: schema })
const [email, emailAttrs] = defineField('email')
const [name, nameAttrs] = defineField('name')

const isOpen = ref(false)
const onSubmit = handleSubmit((values) => {
  console.log(values)
  isOpen.value = false
})
</script>

<template>
  <Dialog v-model:open="isOpen">
    <DialogTrigger as-child>
      <Button>Créer un compte</Button>
    </DialogTrigger>
    <DialogContent>
      <DialogHeader>
        <DialogTitle>Nouveau compte</DialogTitle>
      </DialogHeader>
      
      <form @submit="onSubmit" class="space-y-4">
        <div>
          <Label for="email">Email</Label>
          <Input 
            id="email" 
            v-model="email" 
            v-bind="emailAttrs"
            type="email"
            :aria-invalid="!!errors.email"
            :aria-describedby="errors.email ? 'email-error' : undefined"
          />
          <p v-if="errors.email" id="email-error" class="text-sm text-red-500 mt-1" role="alert">
            {{ errors.email }}
          </p>
        </div>
        
        <DialogFooter>
          <Button type="submit">Créer</Button>
        </DialogFooter>
      </form>
    </DialogContent>
  </Dialog>
</template>
```

---

## SSR/SSG avec Nuxt 4 : points critiques

### useId() pour les associations ARIA stables

Vue 3.5 introduit `useId()` qui génère des IDs stables entre serveur et client, **essentiel** pour les associations `aria-labelledby`, `aria-describedby`, `aria-controls` :

```vue
<script setup lang="ts">
import { useId } from 'vue'

const inputId = useId()
const descriptionId = useId()
const errorId = useId()

const hasError = ref(false)
</script>

<template>
  <div class="form-field">
    <label :for="inputId">Email</label>
    <input 
      :id="inputId"
      type="email"
      :aria-describedby="descriptionId"
      :aria-errormessage="hasError ? errorId : undefined"
      :aria-invalid="hasError"
    />
    <p :id="descriptionId" class="text-sm text-gray-500">
      Votre adresse email professionnelle
    </p>
    <p v-if="hasError" :id="errorId" role="alert" class="text-sm text-red-500">
      Email invalide
    </p>
  </div>
</template>
```

### Configuration Teleport en SSG

Reka UI utilise `<Teleport>` via `DialogPortal`, `DropdownMenuPortal`, etc. En SSG, la cible doit exister. La prop `defer` (Vue 3.5+) résout les problèmes de timing :

```vue
<!-- Avec defer, le teleport attend que la cible soit montée -->
<DialogPortal defer>
  <DialogContent>...</DialogContent>
</DialogPortal>

<!-- Ou désactiver le teleport si problématique -->
<DialogPortal disabled>
  <DialogContent>...</DialogContent>
</DialogPortal>
```

### Configuration Nuxt 4 recommandée

```typescript
// nuxt.config.ts
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  compatibilityDate: '2024-11-01',
  
  modules: [
    'shadcn-nuxt',
    '@nuxtjs/html-validator', // Détecte les erreurs d'accessibilité HTML
    '@vueuse/nuxt'
  ],
  
  shadcn: {
    prefix: '',
    componentDir: './components/ui'
  },
  
  htmlValidator: {
    logLevel: 'verbose',
    options: {
      extends: ['html-validate:recommended'],
      rules: {
        'prefer-native-element': 'error', // Favorise les éléments natifs
        'no-redundant-role': 'error'      // Évite les rôles redondants
      }
    }
  },
  
  css: ['~/assets/css/main.css'],
  
  vite: {
    plugins: [tailwindcss()]
  },
  
  app: {
    head: {
      htmlAttrs: { lang: 'fr' }
    }
  },
  
  // SSG
  routeRules: {
    '/**': { prerender: true }
  }
})
```

### Plugin SSR Width pour éviter les erreurs d'hydration responsive

```typescript
// plugins/ssr-width.ts
import { provideSSRWidth } from '@vueuse/core'

export default defineNuxtPlugin((nuxtApp) => {
  // Assume une largeur desktop pour le SSR
  provideSSRWidth(1024, nuxtApp.vueApp)
})
```

---

## Gestion de prefers-reduced-motion

Reka UI n'inclut pas de styles d'animation — c'est votre responsabilité de respecter `prefers-reduced-motion`.

**Avec Tailwind CSS** :

```vue
<template>
  <!-- motion-safe: animation seulement si non-reduce -->
  <DialogOverlay 
    class="fixed inset-0 bg-black/50 
           motion-safe:animate-fade-in 
           motion-reduce:animate-none"
  />
  
  <AccordionContent 
    class="overflow-hidden 
           motion-safe:data-[state=open]:animate-accordion-down
           motion-safe:data-[state=closed]:animate-accordion-up
           motion-reduce:data-[state=closed]:hidden"
  >
    <!-- Contenu -->
  </AccordionContent>
</template>
```

**Composable pour animation conditionnelle** :

```typescript
// composables/useReducedMotion.ts
export function useReducedMotion() {
  const prefersReducedMotion = useMediaQuery('(prefers-reduced-motion: reduce)')
  
  const transitionDuration = computed(() => 
    prefersReducedMotion.value ? 0 : 300
  )
  
  return { prefersReducedMotion, transitionDuration }
}
```

**CSS global de sécurité** :

```css
/* assets/css/accessibility.css */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

---

## Tests d'accessibilité automatisés

### Configuration Vitest + axe-core

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    environment: 'jsdom', // IMPORTANT: happy-dom ne fonctionne pas avec axe
    setupFiles: ['./test/setup.ts'],
    globals: true
  }
})
```

```typescript
// test/setup.ts
import { expect } from 'vitest'
import * as matchers from 'vitest-axe/matchers'

expect.extend(matchers)
```

**Test de composant avec axe** :

```typescript
// components/__tests__/Dialog.a11y.test.ts
import { render, fireEvent } from '@testing-library/vue'
import { axe } from 'vitest-axe'
import { describe, it, expect } from 'vitest'
import ProfileDialog from '../ProfileDialog.vue'

describe('ProfileDialog Accessibility', () => {
  it('should have no violations when closed', async () => {
    const { container } = render(ProfileDialog)
    expect(await axe(container)).toHaveNoViolations()
  })

  it('should have no violations when open', async () => {
    const { container, getByRole } = render(ProfileDialog)
    
    await fireEvent.click(getByRole('button', { name: /ouvrir/i }))
    
    const results = await axe(container, {
      runOnly: ['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa']
    })
    expect(results).toHaveNoViolations()
  })

  it('should trap focus within dialog', async () => {
    const { getByRole } = render(ProfileDialog)
    
    await fireEvent.click(getByRole('button', { name: /ouvrir/i }))
    
    const dialog = getByRole('dialog')
    expect(dialog).toHaveAttribute('aria-modal', 'true')
    expect(document.activeElement).toBe(dialog.querySelector('button, input'))
  })
})
```

### Lighthouse CI pour SSG

```json
// lighthouserc.json
{
  "ci": {
    "collect": {
      "startServerCommand": "npm run build && npm run preview",
      "url": ["http://localhost:3000/"],
      "numberOfRuns": 3
    },
    "assert": {
      "assertions": {
        "categories:accessibility": ["error", { "minScore": 0.95 }]
      }
    }
  }
}
```

---

## Anti-patterns critiques à éviter

| Anti-pattern | Conséquence | Solution |
|--------------|-------------|----------|
| `outline: none` sans remplacement | Focus invisible (WCAG 2.4.7, 2.4.13) | Utiliser `focus-visible:ring-2` |
| `<div @click>` sans role/tabindex | Non focusable au clavier | Utiliser `<button>` |
| `aria-hidden="true"` sur élément focusable | Conflit lecteur d'écran | Séparer icône et texte sr-only |
| Live region ajoutée dynamiquement | Annonce manquée | Live region présente dès le départ |
| `role="menu"` pour navigation | Sémantique incorrecte | `<nav>` avec liens standards |
| Dialog sans DialogTitle | `aria-labelledby` manquant | Toujours inclure DialogTitle (peut être VisuallyHidden) |
| Targets < 24×24px | Échec WCAG 2.5.8 | `min-width/height: 24px` ou padding |

**Focus perdu après fermeture modale** — toujours restaurer :

```typescript
// Reka UI le fait automatiquement, mais si vous gérez manuellement :
const closeModal = () => {
  isOpen.value = false
  nextTick(() => {
    triggerRef.value?.focus()
  })
}
```

---

## Conclusion

L'implémentation de l'accessibilité WCAG 2.2 avec Reka UI 2.7.0 et shadcn-vue repose sur trois piliers : **faire confiance aux primitives** pour les attributs ARIA et la navigation clavier, **respecter les nouveaux critères WCAG 2.2** (taille cibles, focus visible, reduced motion), et **éviter les anti-patterns** qui cassent l'accessibilité héritée.

Pour Nuxt 4 en SSG, `useId()` est **indispensable** pour des associations ARIA stables entre serveur et client. Les tests automatisés avec axe-core détectent **environ 30-40%** des problèmes d'accessibilité — les tests manuels avec VoiceOver (macOS) et NVDA (Windows) restent nécessaires pour valider l'expérience réelle des utilisateurs de technologies d'assistance.

La checklist minimale : tous les éléments interactifs ≥24×24px, focus visible avec contraste 3:1, `prefers-reduced-motion` respecté, DialogTitle toujours présent, et score Lighthouse accessibilité ≥95.