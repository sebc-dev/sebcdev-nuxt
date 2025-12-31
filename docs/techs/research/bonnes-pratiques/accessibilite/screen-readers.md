# Accessibilité WCAG 2.2 pour screen readers avec Vue/Nuxt et shadcn-vue

Les composants Reka UI et shadcn-vue fournissent une accessibilité ARIA complète par défaut, mais leur personnalisation incorrecte représente la cause principale des régressions d'accessibilité dans les projets Vue modernes. Ce guide détaille les bonnes pratiques WCAG 2.2 spécifiques à votre stack technique : Nuxt 4.2.x SSG, Vue 3 Composition API, shadcn-vue 2.4.3+, Reka UI 2.7.0, et TailwindCSS 4.1.x.

La publication de WCAG 2.2 en octobre 2023 introduit six nouveaux critères de succès ciblant particulièrement les utilisateurs de technologies d'assistance et les personnes avec des déficiences cognitives. Ces critères complètent les exigences existantes en matière d'attributs ARIA, de zones live, de structure des titres et de textes alternatifs. L'intégration de ces pratiques dans un projet Nuxt SSG nécessite une compréhension approfondie des interactions entre le rendu statique, l'hydratation Vue, et les bibliothèques de composants accessibles.

## Les attributs ARIA : hiérarchie et cas d'usage précis

Le calcul du nom accessible par les navigateurs suit une priorité stricte que tout développeur Vue doit maîtriser. L'attribut `aria-labelledby` prime sur tout, suivi de `aria-label`, puis des techniques HTML natives (`<label>`, `alt`, `<caption>`, `<legend>`), et enfin du contenu enfant. Les attributs `title` et `placeholder` arrivent en dernier recours et doivent être évités pour le nommage accessible.

### Choisir entre aria-label et aria-labelledby

L'attribut `aria-label` convient uniquement lorsqu'aucun texte visible n'existe dans le DOM. Son usage principal concerne les boutons icône sans texte visible et les régions de navigation nécessitant identification. En revanche, `aria-labelledby` doit être privilégié dès qu'un texte visible peut servir de label, car il référence directement le contenu existant et garantit la cohérence entre l'affichage visuel et l'annonce vocale.

```vue
<script setup lang="ts">
// Bouton icône - aria-label nécessaire car pas de texte visible
</script>

<template>
  <!-- Correct : aria-label pour bouton icône -->
  <button aria-label="Fermer" class="p-2">
    <XIcon aria-hidden="true" />
  </button>

  <!-- Correct : aria-labelledby référençant un titre visible -->
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

L'attribut `aria-describedby` fournit des informations supplémentaires annoncées après le nom et le rôle de l'élément. Son usage principal concerne les instructions de formulaire, les messages d'erreur et le contexte additionnel pour les boutons critiques. **Ne jamais confondre description et label** : le label identifie, la description explique.

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

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
    <p id="email-hint" class="text-sm text-gray-500">
      Nous ne partagerons jamais votre email.
    </p>
    <p v-if="emailError" id="email-error" role="alert" class="text-red-600">
      {{ emailError }}
    </p>
  </div>
</template>
```

### Anti-patterns courants à éviter absolument

Les erreurs les plus fréquentes incluent l'utilisation de `aria-label` sur des éléments non interactifs comme `<p>`, `<div>` ou `<span>` sans rôle ARIA approprié. L'inclusion du nom de rôle dans le label ("bouton soumettre" au lieu de "soumettre") provoque une double annonce par les lecteurs d'écran. L'utilisation de `placeholder` comme substitut au `<label>` pose des problèmes de persistance et de support.

## Les zones live pour contenus dynamiques

Les zones live (`aria-live`) permettent aux lecteurs d'écran d'annoncer les changements de contenu sans déplacer le focus. La règle cardinale : **la zone live doit exister dans le DOM avant que son contenu ne change**. Une zone créée dynamiquement et immédiatement remplie ne sera pas annoncée.

### Polite versus assertive et combinaisons d'attributs

La valeur `polite` attend que l'utilisateur soit inactif avant d'annoncer le changement, convenant aux messages de succès, mises à jour de panier et résultats de recherche. La valeur `assertive` interrompt immédiatement l'utilisateur et doit être réservée aux erreurs critiques, alertes de sécurité et messages urgents. **Defaulter systématiquement à `polite`** évite de submerger les utilisateurs.

L'attribut `aria-atomic="true"` force l'annonce de la région entière lors de tout changement, essentiel pour les horloges, prix mis à jour ou tout contexte nécessitant l'ensemble de l'information. L'attribut `aria-relevant` spécifie quels types de changements déclenchent les annonces : `additions` pour les nouveaux éléments, `removals` pour les suppressions, `text` pour les modifications textuelles.

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

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

// Exposer pour usage externe
defineExpose({ showToast })
</script>

<template>
  <!-- Zone live pré-existante dans le DOM -->
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

### Intégration avec shadcn-vue Toast et Sonner

shadcn-vue utilise Sonner pour les toasts, qui gère automatiquement les zones live accessibles. Le composant `ToastRoot` de Reka UI propose la prop `type` avec deux valeurs : `foreground` pour les actions initiées par l'utilisateur (annonce immédiate, peut vider la file d'attente) et `background` pour les tâches en arrière-plan (moins disruptif).

```vue
<script setup lang="ts">
import { toast } from 'sonner'

const handleSubmit = async () => {
  try {
    await submitForm()
    // Toast polite pour succès
    toast.success('Formulaire soumis avec succès')
  } catch (error) {
    // Toast assertive pour erreur
    toast.error('Erreur lors de la soumission', {
      description: 'Veuillez réessayer ultérieurement.'
    })
  }
}
</script>
```

## Structure des headings et gestion SSG multi-pages

La hiérarchie des titres `h1` à `h6` constitue le principal mécanisme de navigation pour les utilisateurs de lecteurs d'écran. La touche H dans NVDA et JAWS permet de sauter d'un titre à l'autre, rendant une structure logique absolument critique.

### Règles fondamentales pour Nuxt SSG

Chaque page doit contenir **un seul `<h1>`** décrivant son contenu principal. Les layouts Nuxt ne doivent jamais définir le `<h1>` car celui-ci appartient au contenu spécifique de chaque page. Les sauts de niveau vers le bas (h2 suivi de h4) sont interdits, mais remonter les niveaux après une section (h4 suivi de h2) est autorisé car il indique la fermeture d'une sous-section.

```vue
<!-- layouts/default.vue -->
<script setup lang="ts">
// Le layout fournit la structure mais PAS le h1
</script>

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
useSeoMeta({
  title: 'À propos - Mon Site'
})
</script>

<template>
  <article>
    <h1>À propos de notre équipe</h1>
    
    <section aria-labelledby="history-heading">
      <h2 id="history-heading">Notre histoire</h2>
      <h3>Les débuts en 2015</h3>
      <!-- Contenu -->
      <h3>L'expansion internationale</h3>
      <!-- Contenu -->
    </section>
    
    <section aria-labelledby="values-heading">
      <h2 id="values-heading">Nos valeurs</h2>
      <!-- Contenu -->
    </section>
  </article>
</template>
```

### Composant heading dynamique pour composants réutilisables

Les composants réutilisables comme les cartes ou sections génériques ne peuvent pas connaître à l'avance leur niveau de titre approprié. Un composant de titre dynamique résout ce problème en acceptant le niveau comme prop.

```vue
<!-- components/DynamicHeading.vue -->
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

```vue
<!-- components/Card.vue -->
<script setup lang="ts">
interface Props {
  title: string
  headingLevel?: 2 | 3 | 4 | 5 | 6
}

const props = withDefaults(defineProps<Props>(), {
  headingLevel: 2
})
</script>

<template>
  <article class="card">
    <DynamicHeading :level="headingLevel">
      {{ title }}
    </DynamicHeading>
    <slot />
  </article>
</template>
```

## Alt text et images accessibles dans Nuxt Content

Le texte alternatif doit décrire la **fonction et le contenu** de l'image, pas simplement son apparence. Les images décoratives ou redondantes avec le texte adjacent utilisent `alt=""` pour être ignorées par les lecteurs d'écran. Les phrases "image de" ou "photo de" sont à éviter car les lecteurs d'écran annoncent déjà le rôle "image".

### Images dans Markdown/MDC avec Nuxt Content

La syntaxe standard Markdown supporte le texte alternatif, et la syntaxe MDC permet d'ajouter des attributs additionnels. Un composant ProseImg personnalisé peut renforcer l'accessibilité et gérer automatiquement les figures avec légendes.

```markdown
<!-- Syntaxe Markdown standard -->
![Vue d'ensemble du tableau de bord avec graphiques de ventes](/images/dashboard.png)

<!-- Syntaxe MDC avec attributs -->
![Graphique montrant une croissance de 25% au Q4](/images/chart.png){width="800" height="400"}
```

```vue
<!-- components/content/ProseImg.vue -->
<script setup lang="ts">
interface Props {
  src: string
  alt: string
  width?: string | number
  height?: string | number
}

defineProps<Props>()
</script>

<template>
  <figure v-if="$slots.default" class="my-4">
    <NuxtImg 
      :src="src" 
      :alt="alt" 
      :width="width" 
      :height="height"
      loading="lazy"
      class="rounded-lg"
    />
    <figcaption class="text-sm text-gray-600 mt-2">
      <slot />
    </figcaption>
  </figure>
  <NuxtImg 
    v-else 
    :src="src" 
    :alt="alt" 
    :width="width" 
    :height="height"
    loading="lazy"
  />
</template>
```

### SVG et icônes accessibles

Les SVG décoratifs accompagnant du texte doivent être masqués avec `aria-hidden="true"`. Les SVG porteurs de sens utilisent `role="img"` avec un `<title>` référencé par `aria-labelledby`. L'attribut `focusable="false"` prévient les problèmes de navigation clavier dans Internet Explorer et Edge legacy.

```vue
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
  <!-- SVG décoratif -->
  <svg 
    v-if="decorative"
    aria-hidden="true"
    focusable="false"
    :class="sizeClasses[size]"
  >
    <use :href="`/icons.svg#${name}`" />
  </svg>
  
  <!-- SVG informatif -->
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

## Les six nouveaux critères WCAG 2.2

WCAG 2.2 introduit des exigences spécifiques impactant directement l'implémentation frontend. Ces critères ciblent les utilisateurs de claviers, les personnes avec des déficiences motrices et cognitives.

### Focus non obscurci (2.4.11 - Niveau AA)

Les éléments focusés ne doivent jamais être entièrement masqués par des headers sticky, bandeaux cookies ou widgets de chat. La solution CSS recommandée utilise `scroll-padding` pour créer un espace tampon correspondant à la hauteur des éléments fixes plus une marge de sécurité.

```css
/* globals.css ou tailwind.config.ts */
html {
  scroll-padding-top: 5rem; /* Hauteur header sticky + buffer */
  scroll-padding-bottom: 4rem; /* Hauteur footer sticky si présent */
}

@media (max-width: 768px) {
  html {
    scroll-padding-top: 8rem; /* Header mobile plus grand */
  }
}
```

### Mouvements de glissement (2.5.7 - Niveau AA)

Toute fonctionnalité utilisant le glisser-déposer doit offrir une alternative au clic simple. Les listes réordonnables nécessitent des boutons haut/bas, les sliders doivent permettre le clic sur la piste ou offrir un champ numérique, et les zones de téléchargement par glissement doivent inclure un bouton de sélection de fichier.

```vue
<script setup lang="ts">
import { ref } from 'vue'

interface Item {
  id: string
  name: string
}

const items = ref<Item[]>([
  { id: '1', name: 'Premier élément' },
  { id: '2', name: 'Deuxième élément' },
  { id: '3', name: 'Troisième élément' }
])

const moveItem = (currentIndex: number, direction: -1 | 1) => {
  const newIndex = currentIndex + direction
  if (newIndex >= 0 && newIndex < items.value.length) {
    const [removed] = items.value.splice(currentIndex, 1)
    items.value.splice(newIndex, 0, removed)
  }
}
</script>

<template>
  <ul role="list" class="space-y-2">
    <li 
      v-for="(item, index) in items" 
      :key="item.id"
      class="flex items-center justify-between p-3 bg-white rounded border"
    >
      <span>{{ item.name }}</span>
      
      <!-- Alternatives au glisser-déposer -->
      <div class="flex gap-1">
        <button 
          @click="moveItem(index, -1)"
          :disabled="index === 0"
          :aria-label="`Déplacer ${item.name} vers le haut`"
          class="p-2 disabled:opacity-50"
        >
          <ChevronUpIcon aria-hidden="true" class="w-4 h-4" />
        </button>
        <button 
          @click="moveItem(index, 1)"
          :disabled="index === items.length - 1"
          :aria-label="`Déplacer ${item.name} vers le bas`"
          class="p-2 disabled:opacity-50"
        >
          <ChevronDownIcon aria-hidden="true" class="w-4 h-4" />
        </button>
      </div>
    </li>
  </ul>
</template>
```

### Taille des cibles (2.5.8 - Niveau AA)

Les cibles d'interaction doivent mesurer au minimum **24×24 pixels CSS**, ou disposer d'un espacement de 24px vers les cibles adjacentes si elles sont plus petites. Les checkboxes et radios natifs du navigateur (13-16px) nécessitent une personnalisation.

```css
/* TailwindCSS 4.x avec @apply ou classes utilitaires */
button,
[role="button"],
input[type="submit"] {
  min-width: 24px;
  min-height: 24px;
}

/* Checkboxes accessibles */
input[type="checkbox"],
input[type="radio"] {
  width: 24px;
  height: 24px;
}

/* Boutons icône avec padding pour atteindre 24px */
.icon-button {
  min-width: 24px;
  min-height: 24px;
  padding: 4px; /* 16px icône + 4px padding = 24px */
}
```

### Authentification accessible (3.3.8 - Niveau AA)

Les formulaires d'authentification ne doivent pas exiger de tests cognitifs (mémorisation de mot de passe, transcription de codes, CAPTCHAs) sans fournir d'alternative. Les attributs `autocomplete` corrects permettent aux gestionnaires de mots de passe de fonctionner, et **le collage dans les champs mot de passe ne doit jamais être bloqué**.

```vue
<script setup lang="ts">
const handleSubmit = async () => {
  // Soumission du formulaire
}
</script>

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
          <!-- JAMAIS : @paste.prevent -->
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

### Aide cohérente (3.2.6) et saisie redondante (3.3.7)

Les mécanismes d'aide (contact, FAQ, chat) doivent apparaître au même endroit dans le DOM sur toutes les pages d'un ensemble. Les formulaires multi-étapes doivent pré-remplir ou proposer à la sélection les informations précédemment saisies, comme le pattern "identique à l'adresse de livraison" pour la facturation.

## Intégration Reka UI et shadcn-vue sans régression

Reka UI fournit une accessibilité WAI-ARIA complète pour tous ses composants primitifs. Les attributs ARIA, la gestion du focus et la navigation clavier sont automatiquement configurés. **Le risque principal réside dans la personnalisation** qui peut involontairement casser ces comportements.

### Dialog accessible avec gestion correcte du focus

Le composant Dialog de Reka UI implémente automatiquement le piège de focus (focus trap), la fermeture par Échap, et les attributs `aria-labelledby` et `aria-describedby` via les sous-composants `DialogTitle` et `DialogDescription`. Ces éléments sont **obligatoires** pour l'accessibilité.

```vue
<script setup lang="ts">
import { ref } from 'vue'
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
  DialogClose
} from '@/components/ui/dialog'
import { VisuallyHidden } from 'reka-ui'

const isOpen = ref(false)
</script>

<template>
  <Dialog v-model:open="isOpen">
    <DialogTrigger as-child>
      <Button variant="outline">Ouvrir le dialogue</Button>
    </DialogTrigger>
    
    <DialogContent
      @escape-key-down="isOpen = false"
      @open-auto-focus="(event) => {
        // Personnaliser le focus initial si nécessaire
        // event.preventDefault() pour annuler le focus auto
      }"
    >
      <DialogHeader>
        <!-- Titre visible -->
        <DialogTitle>Modifier le profil</DialogTitle>
        <DialogDescription>
          Effectuez vos modifications ici. Cliquez sur sauvegarder quand vous avez terminé.
        </DialogDescription>
      </DialogHeader>
      
      <!-- Contenu du dialog -->
      
      <DialogClose aria-label="Fermer">
        <XIcon aria-hidden="true" />
      </DialogClose>
    </DialogContent>
  </Dialog>
</template>
```

Pour un titre invisible mais accessible, utilisez `VisuallyHidden` :

```vue
<DialogHeader>
  <VisuallyHidden as-child>
    <DialogTitle>Titre pour lecteurs d'écran uniquement</DialogTitle>
  </VisuallyHidden>
  <DialogDescription>Description visible</DialogDescription>
</DialogHeader>
```

### Tabs avec orientation et activation correctes

Le composant Tabs gère automatiquement les rôles `tablist`, `tab` et `tabpanel`, ainsi que les attributs `aria-selected` et `aria-controls`. La prop `orientation` affecte la navigation clavier (flèches horizontales ou verticales).

```vue
<script setup lang="ts">
import {
  Tabs,
  TabsContent,
  TabsList,
  TabsTrigger
} from '@/components/ui/tabs'
</script>

<template>
  <Tabs default-value="account" orientation="horizontal">
    <TabsList aria-label="Paramètres du compte">
      <TabsTrigger value="account">Compte</TabsTrigger>
      <TabsTrigger value="password">Mot de passe</TabsTrigger>
      <TabsTrigger value="notifications">Notifications</TabsTrigger>
    </TabsList>
    
    <TabsContent value="account">
      <!-- Contenu onglet compte -->
    </TabsContent>
    <TabsContent value="password">
      <!-- Contenu onglet mot de passe -->
    </TabsContent>
    <TabsContent value="notifications">
      <!-- Contenu onglet notifications -->
    </TabsContent>
  </Tabs>
</template>
```

### Erreurs courantes cassant l'accessibilité Reka UI

L'omission de `DialogTitle` ou `DialogDescription` supprime les associations ARIA critiques. L'utilisation de `<div>` au lieu de `as-child` avec un `<Button>` sur les triggers crée des éléments non interactifs. Le blocage des événements clavier (` @keydown.prevent`) sans implémentation alternative casse la navigation. La modification du DOM interne des composants via CSS peut masquer des éléments essentiels aux lecteurs d'écran.

## Outils de test et validation

L'automatisation ne détecte que **30 à 50% des problèmes d'accessibilité**. Les tests manuels avec lecteurs d'écran et navigation clavier restent indispensables pour une couverture complète.

### Tests automatisés dans le pipeline CI/CD

```typescript
// tests/a11y.spec.ts - Playwright avec axe-core
import { test, expect } from '@playwright/test'
import AxeBuilder from '@axe-core/playwright'

test.describe('Accessibilité WCAG 2.2 AA', () => {
  test('Page d\'accueil sans violations', async ({ page }) => {
    await page.goto('/')
    
    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa'])
      .exclude('[data-testid="third-party-widget"]')
      .analyze()
    
    expect(results.violations).toEqual([])
  })
  
  test('Formulaire de contact accessible', async ({ page }) => {
    await page.goto('/contact')
    
    const results = await new AxeBuilder({ page })
      .include('form')
      .analyze()
    
    // Tolérer zéro violation critique ou sérieuse
    const critical = results.violations.filter(
      v => v.impact === 'critical' || v.impact === 'serious'
    )
    expect(critical).toHaveLength(0)
  })
})
```

```yaml
# .github/workflows/a11y.yml
name: Accessibility Tests
on: [push, pull_request]

jobs:
  accessibility:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run build
      - run: npm run test:a11y
      
      - name: Lighthouse CI
        run: |
          npm install -g @lhci/cli@0.15.x
          lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_TOKEN }}
```

### Configuration ESLint pour Vue

```javascript
// eslint.config.js (ESLint 9+)
import pluginVueA11y from 'eslint-plugin-vuejs-accessibility'

export default [
  ...pluginVueA11y.configs['flat/recommended'],
  {
    rules: {
      'vuejs-accessibility/alt-text': 'error',
      'vuejs-accessibility/anchor-has-content': 'error',
      'vuejs-accessibility/click-events-have-key-events': 'error',
      'vuejs-accessibility/form-control-has-label': 'error',
      'vuejs-accessibility/heading-has-content': 'error',
      'vuejs-accessibility/interactive-supports-focus': 'error',
      'vuejs-accessibility/label-has-for': 'error',
      'vuejs-accessibility/tabindex-no-positive': 'error'
    }
  }
]
```

### Tests manuels avec lecteurs d'écran

**NVDA (Windows, gratuit)** constitue le lecteur d'écran le plus utilisé. Les raccourcis essentiels incluent H pour naviguer entre les titres, K pour les liens, B pour les boutons, F pour les champs de formulaire, et D pour les landmarks. La touche Insert + F7 affiche la liste de tous les éléments navigables.

**VoiceOver (macOS/iOS)** utilise le modificateur VO (Ctrl + Option) pour toutes les commandes. Le Rotor (VO + U) permet de naviguer par type d'élément. Contrairement à NVDA, VoiceOver n'a pas de modes browse/focus séparés.

Les tests manuels doivent couvrir : le flux de navigation Tab logique, le piège de focus dans les modales, l'annonce correcte des zones live lors des mises à jour dynamiques, les messages d'erreur de formulaire associés aux champs, et le texte des liens en dehors de leur contexte visuel.

## Conclusion

L'accessibilité WCAG 2.2 dans un stack Vue moderne repose sur trois piliers : exploiter l'accessibilité native de Reka UI sans la casser par des personnalisations hasardeuses, respecter la sémantique HTML et ARIA avec une attention particulière aux zones live et à la structure des titres, et maintenir un pipeline de tests combinant automatisation (axe-core, Lighthouse) et validation manuelle (NVDA, VoiceOver).

Les nouveaux critères WCAG 2.2 introduisent des contraintes concrètes mesurables : **24×24 pixels minimum** pour les cibles d'interaction, **scroll-padding** pour protéger la visibilité du focus, **autocomplete** correctement configuré sur les formulaires d'authentification, et **alternatives au glisser-déposer** systématiques. Ces exigences techniques s'intègrent naturellement dans un workflow Vue/Nuxt utilisant des composants Tailwind stylés et des primitives Reka UI, à condition de ne jamais omettre les sous-composants requis (DialogTitle, TabsList avec aria-label) et de tester régulièrement avec de vrais utilisateurs de technologies d'assistance.