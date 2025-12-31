# shadcn-vue & Reka UI Accessibility Patterns

Reka UI 2.7.0 (base de shadcn-vue) implémente les **WAI-ARIA Authoring Practices** de façon exhaustive. Ce document décrit les comportements automatiques et les patterns à suivre.

## Hiérarchie du Nom Accessible (ARIA)

Le calcul du nom accessible par les navigateurs suit une **priorité stricte** que tout développeur Vue doit maîtriser :

| Priorité | Source | Exemple |
|----------|--------|---------|
| **1** | `aria-labelledby` | Référence un élément par ID |
| **2** | `aria-label` | Texte directement dans l'attribut |
| **3** | HTML natif | `<label>`, `alt`, `<caption>`, `<legend>` |
| **4** | Contenu enfant | Texte à l'intérieur de l'élément |
| **5 (éviter)** | `title` / `placeholder` | Dernier recours, support inconsistant |

### Choisir entre aria-label et aria-labelledby

| Situation | Attribut | Raison |
|-----------|----------|--------|
| Aucun texte visible (bouton icône) | `aria-label` | Fournit le label manquant |
| Texte visible peut servir de label | `aria-labelledby` | Garantit cohérence visuel/vocal |
| Combinaison de plusieurs sources | `aria-labelledby` | Peut référencer plusieurs IDs |

```vue
<template>
  <!-- aria-label : bouton icône sans texte visible -->
  <button aria-label="Fermer" class="p-2">
    <XIcon aria-hidden="true" />
  </button>

  <!-- aria-labelledby : référence un titre visible -->
  <h2 id="billing-heading">Adresse de facturation</h2>
  <div role="region" aria-labelledby="billing-heading">
    <!-- Formulaire -->
  </div>

  <!-- Combinaison de plusieurs sources textuelles -->
  <h2 id="article-title">7 façons de réduire son empreinte carbone</h2>
  <a id="read-more" aria-labelledby="read-more article-title">
    Lire la suite...
  </a>
  <!-- Annonce : "Lire la suite... 7 façons de réduire son empreinte carbone" -->
</template>
```

### aria-describedby pour informations supplémentaires

L'attribut `aria-describedby` fournit des informations **annoncées après** le nom et le rôle. **Ne jamais confondre description et label** : le label identifie, la description explique.

```vue
<script setup lang="ts">
const email = ref('')
const emailError = ref('')

const ariaDescribedBy = computed(() =>
  emailError.value ? 'email-error' : 'email-hint'
)
</script>

<template>
  <div class="form-group">
    <label for="email">Email</label>
    <input
      id="email"
      v-model="email"
      type="email"
      :aria-describedby="ariaDescribedBy"
      :aria-invalid="!!emailError"
    />
    <p id="email-hint" class="text-sm text-muted-foreground">
      Nous ne partagerons jamais votre email.
    </p>
    <p v-if="emailError" id="email-error" role="alert" class="text-destructive">
      {{ emailError }}
    </p>
  </div>
</template>
```

### Anti-patterns ARIA à éviter

| Anti-pattern | Problème | Solution |
|--------------|----------|----------|
| `aria-label` sur `<p>`, `<div>`, `<span>` sans rôle | Ignoré par les lecteurs d'écran | Ajouter `role` approprié ou utiliser élément sémantique |
| `"bouton soumettre"` dans le label | Double annonce ("bouton soumettre button") | Utiliser simplement `"Soumettre"` |
| `placeholder` comme seul label | Disparaît à la saisie, support inconsistant | Utiliser `<label>` visible |
| `title` pour le nommage | Support très inconsistant | Utiliser `aria-label` ou `aria-labelledby` |

---

## Zones Live (aria-live)

Les zones live permettent aux lecteurs d'écran d'annoncer les changements de contenu **sans déplacer le focus**.

### Règle cardinale

> **La zone live DOIT exister dans le DOM AVANT que son contenu ne change.**
> Une zone créée dynamiquement et immédiatement remplie ne sera PAS annoncée.

### polite vs assertive

| Valeur | Comportement | Cas d'usage |
|--------|--------------|-------------|
| `polite` (défaut) | Attend que l'utilisateur soit inactif | Messages succès, mises à jour panier, résultats recherche |
| `assertive` | Interrompt immédiatement | Erreurs critiques, alertes sécurité, messages urgents |

**Règle** : Toujours defaulter à `polite` pour éviter de submerger les utilisateurs.

### Attributs complémentaires

| Attribut | Usage |
|----------|-------|
| `aria-atomic="true"` | Annonce la région **entière** lors de tout changement (horloges, prix) |
| `aria-relevant="additions"` | Annonce uniquement les ajouts |
| `aria-relevant="removals"` | Annonce uniquement les suppressions |
| `aria-relevant="text"` | Annonce les modifications textuelles |

### Pattern Toast accessible

```vue
<script setup lang="ts">
const toastMessage = ref('')
const toastType = ref<'success' | 'error'>('success')

const showToast = (message: string, type: 'success' | 'error' = 'success') => {
  toastType.value = type
  toastMessage.value = message

  // Durée minimale de 5 secondes pour l'accessibilité
  setTimeout(() => {
    toastMessage.value = ''
  }, 5000)
}
</script>

<template>
  <!-- Zone live PRÉ-EXISTANTE dans le DOM -->
  <div
    :role="toastType === 'error' ? 'alert' : 'status'"
    :aria-live="toastType === 'error' ? 'assertive' : 'polite'"
    aria-atomic="true"
    class="toast-container"
  >
    <p v-if="toastMessage">{{ toastMessage }}</p>
  </div>
</template>
```

### Intégration Sonner (shadcn-vue Toast)

shadcn-vue utilise Sonner pour les toasts, qui gère automatiquement les zones live accessibles :

```vue
<script setup lang="ts">
import { toast } from 'sonner'

const handleSubmit = async () => {
  try {
    await submitForm()
    // Toast polite pour succès
    toast.success('Formulaire soumis avec succès')
  } catch {
    // Toast assertive pour erreur
    toast.error('Erreur lors de la soumission', {
      description: 'Veuillez réessayer ultérieurement.'
    })
  }
}
</script>
```

---

## Structure des Headings (SSG Multi-Pages)

La hiérarchie des titres `h1` à `h6` est le **principal mécanisme de navigation** pour les utilisateurs de lecteurs d'écran (touche H dans NVDA/JAWS).

### Règles fondamentales

| Règle | Explication |
|-------|-------------|
| **Un seul `<h1>` par page** | Décrit le contenu principal de la page |
| **Layout ne définit JAMAIS le `<h1>`** | Le `<h1>` appartient au contenu spécifique |
| **Pas de saut vers le bas** | `h2` suivi de `h4` = interdit |
| **Saut vers le haut autorisé** | `h4` suivi de `h2` = OK (ferme la sous-section) |

### Pattern Layout SSG

```vue
<!-- layouts/default.vue -->
<template>
  <div>
    <header>
      <nav aria-label="Navigation principale">
        <!-- Navigation sans h1 -->
      </nav>
    </header>

    <main id="main-content">
      <!-- Le slot contiendra le h1 de la page -->
      <slot />
    </main>

    <footer>
      <!-- h2 maximum dans le footer -->
    </footer>
  </div>
</template>
```

```vue
<!-- pages/about.vue -->
<script setup lang="ts">
useSeoMeta({ title: 'À propos - Mon Site' })
</script>

<template>
  <article>
    <h1>À propos de notre équipe</h1>

    <section aria-labelledby="history-heading">
      <h2 id="history-heading">Notre histoire</h2>
      <h3>Les débuts en 2015</h3>
      <h3>L'expansion internationale</h3>
    </section>

    <section aria-labelledby="values-heading">
      <h2 id="values-heading">Nos valeurs</h2>
    </section>
  </article>
</template>
```

---

## Composant Icon SVG Accessible

Pattern pour icônes décoratives vs informatives :

```vue
<!-- components/Icon.vue -->
<script setup lang="ts">
interface Props {
  name: string
  label?: string
  decorative?: boolean
  size?: 'sm' | 'md' | 'lg'
}

const props = withDefaults(defineProps<Props>(), {
  decorative: true,
  size: 'md'
})

const sizeClasses = {
  sm: 'w-4 h-4',
  md: 'w-5 h-5',
  lg: 'w-6 h-6'
}
</script>

<template>
  <!-- SVG décoratif (accompagne du texte) -->
  <svg
    v-if="decorative"
    aria-hidden="true"
    focusable="false"
    :class="sizeClasses[size]"
  >
    <use :href="`/icons.svg#${name}`" />
  </svg>

  <!-- SVG informatif (porteur de sens) -->
  <svg
    v-else
    role="img"
    :aria-label="label"
    focusable="false"
    :class="sizeClasses[size]"
  >
    <use :href="`/icons.svg#${name}`" />
  </svg>
</template>
```

| Type | Attributs | Quand utiliser |
|------|-----------|----------------|
| Décoratif | `aria-hidden="true"` `focusable="false"` | Icône accompagnant du texte visible |
| Informatif | `role="img"` `aria-label="..."` | Icône seule porteuse de sens |

**Note** : `focusable="false"` prévient les problèmes de navigation clavier dans IE/Edge legacy.

---

## Critères WCAG 2.2 Essentiels

### 2.4.11 Focus Not Obscured (Minimum) - Niveau AA

Un élément focusé ne doit **jamais être entièrement masqué** par du contenu créé par l'auteur (headers sticky, bannières cookies, chatbots flottants).

**Solution CSS avec scroll-padding :**

```css
html {
  scroll-padding-top: var(--header-height, 80px);
  scroll-padding-bottom: 50px; /* Pour footers sticky éventuels */
}
```

**Configuration nuxt.config.ts :**

```typescript
// Définir la variable CSS globalement
app: {
  head: {
    style: [
      { children: ':root { --header-height: 80px; }' }
    ]
  }
}
```

### 2.4.13 Focus Appearance (Niveau AAA - Recommandé)

La surface minimale d'un focus indicator doit être **au moins égale au périmètre de l'élément × 2px**. Pour un bouton de 150×75px : `(150 + 75) × 2 × 2 = 900px²` minimum.

Un **outline solide de 2px** satisfait automatiquement cette exigence.

### Double exigence de contraste focus

Un focus indicator doit satisfaire **deux critères** :

| Critère | Exigence | Mesure |
|---------|----------|--------|
| **1.4.11 Non-text Contrast** | 3:1 contre les couleurs adjacentes | Focus ring vs background |
| **Changement d'état** | 3:1 entre état focusé et non-focusé | Différence visuelle perceptible |

### 2.5.8 Target Size (Minimum) - Niveau AA

Les cibles de pointeur doivent mesurer au moins **24 × 24 CSS pixels**, sauf exceptions (espacement suffisant, alternative équivalente, lien inline).

```vue
<template>
  <!-- ✅ Bouton avec taille minimale -->
  <button class="min-w-6 min-h-6 px-4 py-2">Action</button>

  <!-- ✅ Icône 16x16 avec padding pour atteindre 24x24 -->
  <button class="p-1"> <!-- 16px + 8px padding = 24px -->
    <Icon class="w-4 h-4" />
    <span class="sr-only">Paramètres</span>
  </button>

  <!-- ✅ Liens inline - exception autorisée -->
  <p>Consultez notre <a href="/privacy">politique</a>.</p>
</template>
```

**Règle CSS globale recommandée :**

```css
button:not(.inline-btn),
a:not(.inline-link),
[role="button"] {
  min-width: 24px;
  min-height: 24px;
}
```

### 2.5.7 Dragging Movements - Niveau AA

Toute fonctionnalité utilisant le glissement (drag) doit avoir une **alternative par pointeur simple**.

```vue
<template>
  <!-- Carrousel : boutons + indicateurs en alternative au swipe -->
  <div class="carousel" role="region" aria-label="Carrousel d'images">
    <button @click="prevSlide" aria-label="Image précédente">
      <ChevronLeft />
    </button>

    <div class="carousel-slides"><!-- Slides avec drag optionnel --></div>

    <button @click="nextSlide" aria-label="Image suivante">
      <ChevronRight />
    </button>

    <!-- Indicateurs cliquables -->
    <div role="tablist">
      <button
        v-for="(_, index) in slides"
        :key="index"
        role="tab"
        :aria-selected="currentSlide === index"
        @click="currentSlide = index"
      />
    </div>
  </div>
</template>
```

### 3.3.8 Authentification Accessible - Niveau AA

Les formulaires d'authentification ne doivent pas exiger de **tests cognitifs** sans alternative. Les attributs `autocomplete` permettent aux gestionnaires de mots de passe de fonctionner.

**Règles critiques :**

| Exigence | Raison |
|----------|--------|
| `autocomplete="username"` sur email/login | Permet le remplissage auto |
| `autocomplete="current-password"` sur mot de passe | Permet le remplissage auto |
| **JAMAIS** `@paste.prevent` sur les champs password | Empêche les gestionnaires de mots de passe |
| Alternatives obligatoires | OAuth, magic link, passkeys |

```vue
<template>
  <form @submit.prevent="handleSubmit" autocomplete="on">
    <div class="space-y-4">
      <div>
        <label for="email">Email</label>
        <input
          id="email"
          type="email"
          autocomplete="username"
          inputmode="email"
        />
      </div>

      <div>
        <label for="password">Mot de passe</label>
        <input
          id="password"
          type="password"
          autocomplete="current-password"
          <!-- ❌ JAMAIS : @paste.prevent -->
        />
      </div>

      <button type="submit">Se connecter</button>

      <!-- Alternatives obligatoires WCAG 2.2 -->
      <div class="space-y-2">
        <button type="button" @click="signInWithGoogle">
          Se connecter avec Google
        </button>
        <button type="button" @click="sendMagicLink">
          Recevoir un lien de connexion par email
        </button>
      </div>
    </div>
  </form>
</template>
```

### 3.2.6 Aide Cohérente - Niveau A

Les mécanismes d'aide (contact, FAQ, chat) doivent apparaître **au même endroit** dans le DOM sur toutes les pages d'un ensemble.

| Bon pattern | Mauvais pattern |
|-------------|-----------------|
| Lien "Aide" toujours dans le footer | Aide parfois dans le header, parfois absente |
| Chat widget toujours en bas à droite | Position variable selon les pages |
| FAQ accessible depuis chaque page | FAQ uniquement sur certaines pages |

### 3.3.7 Saisie Redondante - Niveau A

Les formulaires multi-étapes doivent **pré-remplir ou proposer à la sélection** les informations précédemment saisies.

```vue
<script setup lang="ts">
const shippingAddress = ref({ /* ... */ })
const useSameForBilling = ref(true)

const billingAddress = computed(() =>
  useSameForBilling.value ? shippingAddress.value : separateBillingAddress.value
)
</script>

<template>
  <!-- Étape 2 : Facturation -->
  <div class="form-group">
    <label>
      <input type="checkbox" v-model="useSameForBilling" />
      Utiliser la même adresse pour la facturation
    </label>

    <!-- Affiche le formulaire uniquement si adresse différente -->
    <BillingAddressForm v-if="!useSameForBilling" />
  </div>
</template>
```

---

## Technique Double-Ring Focus

### Problème

Un seul outline coloré échoue sur certains fonds : le bleu disparaît sur fond sombre, le blanc sur fond clair.

### Solution mathématique

En superposant deux anneaux avec un contraste **9:1 entre eux**, au moins l'un des deux est **garanti** d'avoir un contraste 3:1 contre n'importe quel fond solide.

### Implémentation CSS

```css
*:focus-visible {
  outline: 2px solid #FFFFFF;  /* Anneau intérieur */
  outline-offset: 0;
  box-shadow: 0 0 0 4px #193146;  /* Anneau extérieur */
}

/* Le box-shadow de 4px : l'outline de 2px le chevauche,
   seuls 2px du shadow seront visibles */
```

### Implémentation TailwindCSS 4

```html
<button class="
  focus-visible:outline-2
  focus-visible:outline-white
  focus-visible:outline-offset-0
  focus-visible:ring-4
  focus-visible:ring-slate-900
  dark:focus-visible:outline-black
  dark:focus-visible:ring-white
">
  Bouton accessible
</button>
```

### Inversion dark mode

| Mode | Outline (intérieur) | Ring/Shadow (extérieur) |
|------|---------------------|-------------------------|
| Light | Blanc `#FFFFFF` | Sombre `#193146` |
| Dark | Noir `#0A0A0A` | Clair `#FAFAFA` |

### ⚠️ Règle critique : ne jamais supprimer outline

```css
/* ❌ INTERDIT - invisible en Windows High Contrast Mode */
*:focus-visible {
  outline: none;
  box-shadow: 0 0 0 4px blue;
}

/* ✅ CORRECT - maintient l'accessibilité WHCM */
*:focus-visible {
  outline: 2px solid transparent;  /* Transparent mais présent */
  box-shadow: 0 0 0 4px blue;
}
```

Les user agents suppriment souvent les `box-shadow` en mode couleurs forcées (High Contrast). L'`outline` est préservé.

---

## `focus-visible:` vs `focus:`

### Différence comportementale

| Pseudo-classe | Déclenchement | Usage |
|---------------|---------------|-------|
| `:focus` | Tout focus (clavier, souris, programmatique) | ❌ Éviter |
| `:focus-visible` | Focus clavier uniquement (heuristique navigateur) | ✅ Recommandé |

### Pourquoi utiliser `focus-visible:`

```html
<!-- ❌ Focus ring apparaît au clic souris (mauvaise UX) -->
<button class="focus:ring-2 focus:ring-blue-500">Clic</button>

<!-- ✅ Focus ring uniquement au Tab clavier -->
<button class="focus-visible:ring-2 focus-visible:ring-blue-500">Tab</button>
```

**Règle** : Toujours utiliser `focus-visible:` pour les indicateurs visuels de focus, sauf besoin spécifique d'indiquer le focus au clic (champs de formulaire par exemple).

---

## Outils de Vérification Contraste

### OddContrast pour oklch

Les outils classiques (WebAIM Contrast Checker) **ne supportent pas oklch**. Utiliser **OddContrast** qui gère nativement cet espace colorimétrique moderne.

### Outils recommandés

| Outil | Usage | Support oklch |
|-------|-------|---------------|
| **OddContrast** | Vérification contraste oklch | ✅ Natif |
| Chrome DevTools → Accessibility | Inspection en temps réel | ✅ |
| axe DevTools extension | Audit automatisé | ✅ |
| WAVE extension (v3.3.0+) | Audit visuel, supporte alpha/opacity | ✅ |
| WebAIM Contrast Checker | Vérification rapide | ❌ RGB uniquement |

### Simulateurs daltonisme

Chrome DevTools → Rendering → **Emulate vision deficiencies** :
- Protanopia (rouge)
- Deuteranopia (vert)
- Tritanopia (bleu)
- Achromatopsia (niveaux de gris)

---

## Tabindex : Règles Fondamentales

### Valeurs et usages

| Valeur | Comportement | Cas d'utilisation |
|--------|--------------|-------------------|
| **Absent** | Ordre naturel du DOM | Éléments HTML interactifs natifs (`<button>`, `<a>`, `<input>`) |
| `tabindex="0"` | Inclus dans l'ordre naturel | Éléments custom rendus interactifs |
| `tabindex="-1"` | Focusable par JS uniquement | Conteneurs de focus, navigation programmatique |
| `tabindex > 0` | **❌ INTERDIT** | Jamais - casse l'ordre prévisible |

### Règle : Utiliser les éléments HTML natifs en priorité

Les éléments `<button>`, `<a>`, `<input>` possèdent un comportement clavier natif (Enter, Space) et sont déjà dans l'ordre de tabulation.

```vue
<template>
  <!-- ✅ Éléments natifs - aucun tabindex nécessaire -->
  <button @click="handleAction">Action</button>
  <a href="/page">Navigation</a>

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
    <!-- Le focus est dirigé ici après navigation -->
  </main>
</template>
```

### Anti-patterns tabindex

```vue
<template>
  <!-- ❌ tabindex positif - détruit l'ordre prévisible -->
  <button tabindex="1">Premier</button>
  <button tabindex="2">Deuxième</button>

  <!-- ❌ Div cliquable sans accessibilité clavier -->
  <div @click="action">Cliquable mais pas accessible</div>

  <!-- ❌ tabindex="0" sur élément non-interactif -->
  <p tabindex="0">Ce paragraphe n'a pas besoin d'être focusable</p>

  <!-- ❌ Retirer un élément interactif de la tabulation -->
  <button tabindex="-1">Bouton inaccessible au clavier</button>
</template>
```

---

## Skip Links (Liens d'évitement)

WCAG 2.4.1 (Bypass Blocks) exige un moyen de sauter directement au contenu principal, évitant de traverser la navigation à chaque page.

### Implémentation complète

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
    if (!target.hasAttribute('tabindex')) {
      target.setAttribute('tabindex', '-1')
    }
    target.focus()
    target.scrollIntoView({ behavior: 'smooth' })
  }
}
</script>

<template>
  <nav aria-label="Liens d'évitement">
    <a
      v-for="link in links"
      :key="link.targetId"
      :href="`#${link.targetId}`"
      class="
        sr-only focus:not-sr-only
        focus:fixed focus:top-4 focus:left-4 focus:z-50
        focus:px-4 focus:py-3 focus:rounded-lg
        focus:bg-primary focus:text-primary-foreground
        focus-visible:outline-2 focus-visible:outline-offset-2
        motion-safe:transition-all
      "
      @click.prevent="skipTo(link.targetId)"
    >
      {{ link.label }}
    </a>
  </nav>
</template>
```

### Positionnement dans le layout

```vue
<!-- layouts/default.vue -->
<template>
  <!-- Skip links EN PREMIER dans le DOM -->
  <SkipLinks />

  <header>
    <nav id="main-nav" aria-label="Navigation principale">
      <MainNavigation />
    </nav>
  </header>

  <main id="main-content" tabindex="-1">
    <slot />
  </main>
</template>
```

---

## Focus Reset après Navigation Client-Side

En SSG avec hydratation, le changement de route ne déclenche pas d'annonce aux lecteurs d'écran. Le focus doit être explicitement déplacé.

```vue
<!-- layouts/default.vue -->
<script setup lang="ts">
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
  <TheHeader />

  <main id="main-content" tabindex="-1">
    <h1 ref="pageTitle" tabindex="-1" class="outline-hidden">
      {{ route.meta.title || 'Page' }}
    </h1>
    <slot />
  </main>
</template>
```

**Note** : Le `tabindex="-1"` sur le `<h1>` permet le focus programmatique sans l'inclure dans l'ordre de tabulation normal. `outline-hidden` évite l'outline au focus programmatique.

---

## Navigation Clavier par Composant

### Tableau récapitulatif

| Composant | Navigation | Actions | Notes |
|-----------|------------|---------|-------|
| **Dialog** | `Tab` entre éléments | `Escape` ferme | Focus trap automatique |
| **Tabs** | `←` `→` (horizontal) / `↑` `↓` (vertical) | `Home`/`End` premier/dernier | `Enter`/`Space` si `activation-mode="manual"` |
| **Accordion** | `↑` `↓` entre triggers | `Space`/`Enter` toggle | Roving tabindex auto |
| **DropdownMenu** | `↑` `↓` entre items | Typeahead par lettre | `→` `←` sous-menus |
| **Select** | `↑` `↓` dans la liste | `Enter` sélectionne | Typeahead supporté |
| **Tooltip** | N/A (hover/focus) | `Escape` ferme | Délai configurable |

### Attributs ARIA automatiques

Reka UI injecte automatiquement les attributs appropriés :

| Composant | Attributs |
|-----------|-----------|
| **Dialog** | `role="dialog"`, `aria-modal="true"`, `aria-labelledby`, `aria-describedby` |
| **Tabs** | `role="tablist"`, `role="tab"`, `role="tabpanel"`, `aria-selected` |
| **Accordion** | `role="region"`, `aria-expanded`, `aria-controls` |
| **DropdownMenu** | `role="menu"`, `role="menuitem"`, `aria-haspopup` |

## Focus Management

### Focus Trapping (Dialog, AlertDialog, Sheet)

Le focus est automatiquement piégé quand `modal=true` (défaut). À la fermeture, le focus retourne au trigger.

```vue
<Dialog>
  <DialogTrigger as-child>
    <Button>Ouvrir</Button>  <!-- Focus retourne ici à la fermeture -->
  </DialogTrigger>
  <DialogContent>
    <!-- Focus piégé dans ce conteneur -->
    <input autofocus />  <!-- Premier élément focusable -->
  </DialogContent>
</Dialog>
```

### Roving Tabindex (Tabs, RadioGroup, Toolbar)

Un seul élément du groupe est dans le tab order. Les flèches naviguent entre éléments.

```vue
<!-- Tab order: [autres éléments] → [tab actif] → [autres éléments] -->
<Tabs default-value="tab1">
  <TabsList>
    <TabsTrigger value="tab1">Un</TabsTrigger>   <!-- tabindex="0" si actif -->
    <TabsTrigger value="tab2">Deux</TabsTrigger> <!-- tabindex="-1" si inactif -->
  </TabsList>
</Tabs>
```

### RovingFocusGroup pour widgets custom

Reka UI expose `RovingFocusGroup` pour créer des toolbars accessibles :

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
        class="p-2 hover:bg-accent focus-visible:outline-2"
      >
        <component :is="tool.icon" class="w-4 h-4" />
      </button>
    </RovingFocusItem>
  </RovingFocusGroup>
</template>
```

| Prop | Valeur | Description |
|------|--------|-------------|
| `orientation` | `"horizontal"` / `"vertical"` | Direction des flèches |
| `loop` | `true` / `false` | Boucle au dernier/premier élément |

---

## Gestion de la Touche Escape

### Comportement par défaut Reka UI

Tous les composants overlay ferment automatiquement avec Escape et émettent `@escape-key-down` pour personnalisation.

```vue
<template>
  <!-- Comportement par défaut : Escape ferme -->
  <DialogContent>
    <!-- Rien à faire -->
  </DialogContent>

  <!-- Fermeture conditionnelle (formulaire non sauvegardé) -->
  <DialogContent @escape-key-down="handleEscape">
    <!-- Escape ferme sauf si données non sauvegardées -->
  </DialogContent>

  <!-- Escape désactivé (fournir bouton de fermeture visible!) -->
  <DialogContent @escape-key-down.prevent>
    <DialogClose as-child>
      <Button>Fermer</Button> <!-- OBLIGATOIRE si Escape désactivé -->
    </DialogClose>
  </DialogContent>
</template>

<script setup lang="ts">
const hasUnsavedChanges = ref(false)

function handleEscape(event: KeyboardEvent) {
  if (hasUnsavedChanges.value) {
    event.preventDefault()
    // Afficher confirmation avant fermeture
  }
}
</script>
```

### ⚠️ Anti-pattern : Escape désactivé sans alternative

```vue
<!-- ❌ Utilisateur piégé dans la modale -->
<DialogContent @escape-key-down.prevent>
  <!-- Pas de bouton de fermeture visible -->
</DialogContent>
```

---

## VisuallyHidden - Titres Accessibles

Quand le design ne permet pas de titre visible, utiliser `VisuallyHidden` :

```vue
<script setup>
import { VisuallyHidden } from 'reka-ui'
</script>

<template>
  <Dialog>
    <DialogContent>
      <!-- Titre masqué visuellement mais lu par lecteurs d'écran -->
      <VisuallyHidden as-child>
        <DialogTitle>Confirmation de suppression</DialogTitle>
      </VisuallyHidden>

      <!-- Contenu visible -->
      <p>Voulez-vous vraiment supprimer cet élément ?</p>
      <DialogFooter>
        <Button variant="destructive">Supprimer</Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>
```

**Cas d'usage VisuallyHidden :**

| Situation | Solution |
|-----------|----------|
| Dialog sans titre visible | `<VisuallyHidden><DialogTitle>` |
| Icône seule comme bouton | `<VisuallyHidden>Label accessible</VisuallyHidden>` |
| Description technique | `<VisuallyHidden><DialogDescription>` |

## Patterns d'Activation

### Tabs : automatic vs manual

```vue
<!-- AUTOMATIC (défaut) : active au focus -->
<Tabs activation-mode="automatic">
  <!-- Navigation rapide : flèches changent directement l'onglet -->
</Tabs>

<!-- MANUAL : requiert Enter/Space après focus -->
<Tabs activation-mode="manual">
  <!-- Meilleur si le contenu est lourd à charger -->
</Tabs>
```

### Accordion : single vs multiple

```vue
<!-- SINGLE avec collapsible : un seul ouvert, peut tout fermer -->
<Accordion type="single" collapsible>
  <AccordionItem value="faq-1">...</AccordionItem>
</Accordion>

<!-- MULTIPLE : plusieurs sections ouvertes simultanément -->
<Accordion type="multiple">
  <AccordionItem value="faq-1">...</AccordionItem>
</Accordion>
```

### Props pour contenu persistant

```vue
<!-- Garde le contenu dans le DOM même fermé (utile pour Ctrl+F) -->
<AccordionContent unmount-on-hide="false">
  Contenu searchable par le navigateur
</AccordionContent>
```

## Accessibilité des Animations (prefers-reduced-motion)

### Variants Tailwind motion-safe / motion-reduce

TailwindCSS 4 active par défaut les variants `motion-safe:` et `motion-reduce:`, mappant sur la media query `prefers-color-scheme`. L'approche recommandée est l'**opt-in** : les animations ne s'activent que pour les utilisateurs n'ayant pas demandé leur réduction.

```html
<!-- Pattern opt-in : animation uniquement si autorisée -->
<button class="
  bg-indigo-600
  motion-safe:transition
  motion-safe:duration-200
  motion-safe:hover:scale-105
  hover:bg-indigo-700
">
  Action
</button>

<!-- Spinner avec fallback statique -->
<svg class="motion-safe:animate-spin motion-reduce:hidden h-5 w-5">
  <!-- Spinner animé -->
</svg>
<span class="hidden motion-reduce:inline">⏳</span>
```

### Détection JavaScript avec VueUse

Pour les animations JavaScript (GSAP, Motion Vue), utiliser `usePreferredReducedMotion` de VueUse :

```typescript
import { usePreferredReducedMotion } from '@vueuse/core'

const motionPreference = usePreferredReducedMotion()
const shouldAnimate = computed(() => motionPreference.value === 'no-preference')
const animationDuration = computed(() => shouldAnimate.value ? 300 : 0)
```

### Critères WCAG 2.2

| Critère | Exigence | Impact |
|---------|----------|--------|
| **2.2.2** | Pause/stop/hide pour animations >5s | Carousels, backgrounds animés |
| **2.3.1** | Pas plus de 3 flashs/seconde | Risque épilepsie |
| **2.3.3** | Animations déclenchées par interaction désactivables | Respect `prefers-reduced-motion` |

**Recommandation** : Respecter `prefers-reduced-motion` satisfait généralement ces trois exigences.

---

## Animations shadcn-vue et Reka UI

### Attributs data-state pour animations

Les composants shadcn-vue utilisent les attributs `data-state` de Reka UI pour déclencher les animations CSS. Combinés avec `tw-animate-css`, cela donne :

```html
<DialogContent class="
  data-[state=open]:animate-in
  data-[state=open]:fade-in-0
  data-[state=open]:zoom-in-95
  data-[state=closed]:animate-out
  data-[state=closed]:fade-out-0
  data-[state=closed]:zoom-out-95
  data-[state=closed]:duration-200
  data-[state=open]:duration-300
">
  Contenu du dialog
</DialogContent>
```

### Utilitaires tw-animate-css

| Utilitaire | Animation |
|------------|-----------|
| `animate-in` | Active l'animation d'entrée |
| `animate-out` | Active l'animation de sortie |
| `fade-in-0` / `fade-out-0` | Fondu depuis/vers opacité 0 |
| `zoom-in-95` / `zoom-out-95` | Scale depuis/vers 95% |
| `slide-in-from-right` | Glisse depuis la droite |
| `slide-out-to-left` | Glisse vers la gauche |

### Variables CSS Reka UI pour animations dynamiques

Reka UI 2.7.0 expose des **variables CSS pour les dimensions**, permettant des animations de hauteur fluides sans JavaScript :

```css
/* Accordion avec animation hauteur fluide */
.accordion-content {
  overflow: hidden;
  /* Variable exposée par Reka UI */
  height: var(--reka-accordion-content-height);
  transition: height 300ms ease-out;
}

/* Collapsible content */
.collapsible-content {
  height: var(--reka-collapsible-content-height);
  width: var(--reka-collapsible-content-width);
}
```

### Prop forceMount pour Vue Transition

La prop `forceMount` maintient l'élément dans le DOM pendant les animations de sortie, nécessaire pour Vue `<Transition>` ou Motion Vue :

```vue
<DialogContent force-mount>
  <Transition
    enter-active-class="transition-all duration-300"
    leave-active-class="transition-all duration-200"
  >
    <div v-if="open">
      Contenu avec transition Vue
    </div>
  </Transition>
</DialogContent>
```

---

## Contraste et Couleurs

Les thèmes shadcn-vue doivent respecter WCAG AA :

| Élément | Ratio minimum | Vérification |
|---------|---------------|--------------|
| Texte normal | **4.5:1** | `--foreground` vs `--background` |
| Texte large (18px+ bold) | **3:1** | Titres |
| Composants UI | **3:1** | Bordures, icônes |
| Focus ring | **3:1** | `--ring` vs background |

**Outils de vérification :**
- Chrome DevTools → Accessibility panel
- Extension axe DevTools
- WebAIM Contrast Checker

## useId() pour Associations ARIA Stables (SSG)

Vue 3.5+ introduit `useId()` qui génère des IDs **stables entre serveur et client**, essentiel pour éviter les erreurs d'hydration avec les associations ARIA.

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
    <p :id="descriptionId" class="text-sm text-muted-foreground">
      Votre adresse email professionnelle
    </p>
    <p v-if="hasError" :id="errorId" role="alert" class="text-sm text-destructive">
      Email invalide
    </p>
  </div>
</template>
```

| Cas d'usage | Attribut ARIA | Pourquoi useId() |
|-------------|---------------|------------------|
| Label → Input | `for` / `id` | ID stable SSR/client |
| Input → Description | `aria-describedby` | Référence stable |
| Input → Erreur | `aria-errormessage` | Référence stable |
| Tab → Panel | `aria-controls` | Association stable |

**Règle** : Toujours utiliser `useId()` au lieu de `Math.random()` ou compteurs manuels pour les associations ARIA en SSG.

---

## Configuration Teleport/Portal en SSG

Reka UI utilise `<Teleport>` via `DialogPortal`, `DropdownMenuPortal`, etc. En SSG, la cible doit exister au moment du rendu.

### Prop `defer` (Vue 3.5+)

```vue
<!-- Attend que la cible soit montée avant de téléporter -->
<DialogPortal defer>
  <DialogContent>...</DialogContent>
</DialogPortal>
```

### Désactiver le Teleport si problématique

```vue
<!-- Rend le contenu in-place sans téléportation -->
<DialogPortal disabled>
  <DialogContent>...</DialogContent>
</DialogPortal>
```

| Prop | Comportement | Quand utiliser |
|------|--------------|----------------|
| `defer` | Attend que la cible existe | SSG avec timing complexe |
| `disabled` | Pas de téléportation | Debugging, cas spéciaux |
| (défaut) | Téléporte immédiatement | Cas standard CSR |

---

## Props Critiques Reka UI pour l'Accessibilité

### Dialog

| Prop / Event | Valeur | Usage accessibilité |
|--------------|--------|---------------------|
| `modal` | `true` (défaut) | Active focus trap, `aria-hidden` sur contenu externe |
| `@open-auto-focus` | `(e) => e.preventDefault()` | Personnaliser focus initial |
| `@close-auto-focus` | `(e) => e.preventDefault()` | Personnaliser focus au retour |

```vue
<DialogContent
  @open-auto-focus="(e) => {
    e.preventDefault()
    emailInput?.focus()
  }"
>
```

### Tabs

| Prop | Valeur | Usage accessibilité |
|------|--------|---------------------|
| `activationMode="automatic"` | Défaut | Active tab au focus (flèches) |
| `activationMode="manual"` | | Requiert Enter/Space - **préférer pour formulaires longs** |
| `orientation` | `horizontal` / `vertical` | Change navigation ←→ vs ↑↓ |
| `unmountOnHide` | `true` (défaut) | Démonte contenu caché pour performances |

### Accordion

| Prop | Valeur | Usage accessibilité |
|------|--------|---------------------|
| `type="single"` | | Un seul item ouvert |
| `type="multiple"` | | Plusieurs items ouverts |
| `collapsible` | `true` | Permet de tout fermer (avec `type="single"`) |

---

## Pattern Dialog + Form Accessible (VeeValidate)

Pattern complet pour formulaires dans les dialogs avec validation accessible :

```vue
<script setup lang="ts">
import { useId } from 'vue'
import { toTypedSchema } from '@vee-validate/zod'
import { useForm } from 'vee-validate'
import { z } from 'zod'
import {
  Dialog, DialogContent, DialogHeader,
  DialogTitle, DialogFooter
} from '@/components/ui/dialog'

const schema = toTypedSchema(z.object({
  email: z.string().email('Email invalide'),
  name: z.string().min(2, 'Minimum 2 caractères')
}))

const { handleSubmit, errors, defineField } = useForm({
  validationSchema: schema
})
const [email, emailAttrs] = defineField('email')
const [name, nameAttrs] = defineField('name')

const isOpen = ref(false)
const emailId = useId()
const emailErrorId = useId()

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
          <Label :for="emailId">Email</Label>
          <Input
            :id="emailId"
            v-model="email"
            v-bind="emailAttrs"
            type="email"
            :aria-invalid="!!errors.email"
            :aria-describedby="errors.email ? emailErrorId : undefined"
          />
          <p
            v-if="errors.email"
            :id="emailErrorId"
            class="text-sm text-destructive mt-1"
            role="alert"
          >
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

| Attribut | Rôle accessibilité |
|----------|-------------------|
| `aria-invalid` | Indique le champ en erreur |
| `aria-describedby` | Associe le message d'erreur au champ |
| `role="alert"` | Annonce l'erreur aux lecteurs d'écran |

---

## Erreurs Courantes Cassant l'Accessibilité Reka UI

| Erreur | Problème | Solution |
|--------|----------|----------|
| Omission de `DialogTitle` | Association ARIA supprimée, lecteurs d'écran perdus | Toujours inclure `DialogTitle` (visible ou `VisuallyHidden`) |
| Omission de `DialogDescription` | Manque de contexte pour les utilisateurs | Ajouter description ou `VisuallyHidden` si non nécessaire visuellement |
| `<div>` au lieu de `as-child` sur triggers | Élément non-interactif créé | Utiliser `as-child` avec `<Button>` |
| `@keydown.prevent` sans alternative | Navigation clavier cassée | Implémenter alternative ou ne pas bloquer |
| Modification DOM via CSS (`display: none`) | Éléments masqués aux lecteurs d'écran | Utiliser les props Reka UI (`open`, `hidden`) |
| IDs générés avec `Math.random()` | Erreurs hydration SSG | Utiliser `useId()` de Vue 3.5+ |
| Focus perdu après fermeture modale | Utilisateur désorienté | Laisser Reka UI gérer ou restaurer manuellement |

### Exemple Dialog correctement accessible

```vue
<template>
  <Dialog>
    <DialogTrigger as-child>
      <Button>Ouvrir</Button> <!-- ✅ as-child avec Button -->
    </DialogTrigger>

    <DialogContent>
      <DialogHeader>
        <DialogTitle>Confirmation</DialogTitle> <!-- ✅ Obligatoire -->
        <DialogDescription>
          Cette action est irréversible.
        </DialogDescription> <!-- ✅ Obligatoire -->
      </DialogHeader>

      <DialogFooter>
        <DialogClose as-child>
          <Button variant="outline">Annuler</Button>
        </DialogClose>
        <Button variant="destructive">Confirmer</Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>
```

---

## Tests Manuels avec Lecteurs d'Écran

Les tests automatisés détectent **~30% des problèmes** d'accessibilité. Les 70% restants nécessitent des tests manuels.

### NVDA (Windows, gratuit)

Lecteur d'écran le plus utilisé. Raccourcis essentiels :

| Raccourci | Action |
|-----------|--------|
| **H** | Naviguer entre les titres (headings) |
| **K** | Naviguer entre les liens |
| **B** | Naviguer entre les boutons |
| **F** | Naviguer entre les champs de formulaire |
| **D** | Naviguer entre les landmarks |
| **Insert + F7** | Liste de tous les éléments navigables |
| **Insert + F5** | Liste des liens |
| **Tab** | Élément interactif suivant |
| **Shift + Tab** | Élément interactif précédent |

### VoiceOver (macOS/iOS)

Utilise le modificateur **VO** (Ctrl + Option) pour toutes les commandes.

| Raccourci | Action |
|-----------|--------|
| **VO + U** | Rotor (navigation par type d'élément) |
| **VO + →** / **VO + ←** | Élément suivant/précédent |
| **VO + Cmd + H** | Titre suivant |
| **VO + Cmd + J** | Élément de formulaire suivant |
| **VO + Cmd + L** | Lien suivant |
| **Tab** | Élément interactif suivant |

**Différence clé** : VoiceOver n'a pas de modes browse/focus séparés comme NVDA.

### Checklist Tests Manuels

```markdown
## Tests obligatoires avant release
- [ ] Flux de navigation Tab logique et prévisible
- [ ] Focus trap fonctionnel dans les modales
- [ ] Zones live annoncent les mises à jour dynamiques
- [ ] Messages d'erreur formulaire associés aux champs
- [ ] Texte des liens compréhensible hors contexte visuel
- [ ] Hiérarchie des headings cohérente (H pour naviguer)
- [ ] Images informatives ont un alt descriptif
- [ ] Boutons icône ont un label accessible
- [ ] Skip link fonctionne et mène au contenu principal
```

---

## Checklist Accessibilité Composants

```markdown
## Avant merge d'un nouveau composant
- [ ] Navigation clavier fonctionne (Tab, Escape, flèches)
- [ ] Focus visible sur tous les éléments interactifs
- [ ] Labels accessibles (pas d'icônes seules sans texte)
- [ ] Contraste suffisant (4.5:1 texte, 3:1 UI)
- [ ] Annonces screen reader appropriées
- [ ] Test avec axe DevTools (0 violations critical/serious)
```

---

## Layout Accessible Complet (Référence)

Exemple de layout intégrant tous les patterns d'accessibilité :

```vue
<!-- layouts/default.vue -->
<script setup lang="ts">
const route = useRoute()
const pageTitle = ref<HTMLHeadingElement | null>(null)

// Focus reset après navigation client-side
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

  <header class="sticky top-0 z-40 bg-background border-b">
    <nav
      id="main-nav"
      aria-label="Navigation principale"
      class="container mx-auto px-4"
    >
      <MainNavigation />
    </nav>
  </header>

  <main
    id="main-content"
    tabindex="-1"
    class="container mx-auto px-4 py-8 min-h-screen"
  >
    <h1
      ref="pageTitle"
      tabindex="-1"
      class="text-3xl font-bold mb-6 outline-hidden"
    >
      {{ route.meta.title || 'Page' }}
    </h1>

    <slot />
  </main>

  <footer class="bg-muted py-8">
    <div class="container mx-auto px-4">
      <FooterContent />
    </div>
  </footer>
</template>

<style>
/* WCAG 2.4.11 - Focus Not Obscured */
html {
  scroll-padding-top: 80px;
  scroll-padding-bottom: 40px;
}

/* WCAG 2.5.8 - Target Size Minimum */
button:not(.inline-btn),
a:not(.inline-link),
[role="button"] {
  min-width: 24px;
  min-height: 24px;
}
</style>
```

### Checklist du layout

| Critère | Implémentation |
|---------|----------------|
| Skip links premier élément | `<SkipLinks />` en tête du template |
| Landmarks sémantiques | `<header>`, `<main>`, `<footer>` avec roles |
| Focus reset navigation | `watch(route.path)` + `nextTick` + `focus()` |
| Cible skip link focusable | `<main tabindex="-1">` |
| Focus Not Obscured | `scroll-padding-top` |
| Target Size | `min-width/height: 24px` global |

---

## Checklist Minimale WCAG 2.2 (Résumé Actionnable)

Critères **indispensables** avant toute mise en production :

| Critère | Exigence | Vérification |
|---------|----------|--------------|
| **Taille cibles** | ≥ 24×24 CSS pixels | `min-width: 24px; min-height: 24px` sur interactifs |
| **Focus visible** | Contraste 3:1 entre états | Double-ring ou outline 2px visible |
| **DialogTitle** | Toujours présent | Visible ou `<VisuallyHidden>` |
| **prefers-reduced-motion** | Respecté | CSS filet de sécurité global |
| **IDs ARIA** | Stables SSR/client | `useId()` de Vue 3.5+ |
| **Score Lighthouse** | ≥ 95% accessibilité | CI avec assertions |

### Commandes de vérification rapide

```bash
# Tests automatisés (détectent ~30% des problèmes)
pnpm test:a11y              # Vitest + axe-core
pnpm test:e2e:a11y          # Playwright + axe

# Lighthouse CI
pnpm lhci autorun

# ESLint accessibilité
pnpm lint --filter vuejs-accessibility
```

### Outils de tests manuels (70% restants)

| Outil | Usage |
|-------|-------|
| **NVDA** (Windows) | Test principal lecteur d'écran |
| **VoiceOver** (macOS) | Test secondaire |
| **Chrome DevTools > Accessibility** | Inspection ARIA temps réel |
| **axe DevTools extension** | Audit page complète |
| **WAVE extension** | Audit visuel (overlay sur la page) |

**Règle** : Les tests automatisés sont nécessaires mais insuffisants. Prévoir des tests manuels avec lecteurs d'écran avant chaque release majeure.
