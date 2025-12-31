# Critères WCAG 2.2 Essentiels

## 2.4.11 Focus Not Obscured (Minimum) - Niveau AA

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

## 2.4.13 Focus Appearance (Niveau AAA - Recommandé)

La surface minimale d'un focus indicator doit être **au moins égale au périmètre de l'élément × 2px**. Pour un bouton de 150×75px : `(150 + 75) × 2 × 2 = 900px²` minimum.

Un **outline solide de 2px** satisfait automatiquement cette exigence.

## Double exigence de contraste focus

Un focus indicator doit satisfaire **deux critères** :

| Critère | Exigence | Mesure |
|---------|----------|--------|
| **1.4.11 Non-text Contrast** | 3:1 contre les couleurs adjacentes | Focus ring vs background |
| **Changement d'état** | 3:1 entre état focusé et non-focusé | Différence visuelle perceptible |

## 2.5.8 Target Size (Minimum) - Niveau AA

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

## 2.5.7 Dragging Movements - Niveau AA

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

## 3.3.8 Authentification Accessible - Niveau AA

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

## 3.2.6 Aide Cohérente - Niveau A

Les mécanismes d'aide (contact, FAQ, chat) doivent apparaître **au même endroit** dans le DOM sur toutes les pages d'un ensemble.

| Bon pattern | Mauvais pattern |
|-------------|-----------------|
| Lien "Aide" toujours dans le footer | Aide parfois dans le header, parfois absente |
| Chat widget toujours en bas à droite | Position variable selon les pages |
| FAQ accessible depuis chaque page | FAQ uniquement sur certaines pages |

## 3.3.7 Saisie Redondante - Niveau A

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
