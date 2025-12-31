# Hiérarchie du Nom Accessible (ARIA)

Le calcul du nom accessible par les navigateurs suit une **priorité stricte** que tout développeur Vue doit maîtriser :

| Priorité | Source | Exemple |
|----------|--------|---------|
| **1** | `aria-labelledby` | Référence un élément par ID |
| **2** | `aria-label` | Texte directement dans l'attribut |
| **3** | HTML natif | `<label>`, `alt`, `<caption>`, `<legend>` |
| **4** | Contenu enfant | Texte à l'intérieur de l'élément |
| **5 (éviter)** | `title` / `placeholder` | Dernier recours, support inconsistant |

## Choisir entre aria-label et aria-labelledby

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

## aria-describedby pour informations supplémentaires

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

## Anti-patterns ARIA à éviter

| Anti-pattern | Problème | Solution |
|--------------|----------|----------|
| `aria-label` sur `<p>`, `<div>`, `<span>` sans rôle | Ignoré par les lecteurs d'écran | Ajouter `role` approprié ou utiliser élément sémantique |
| `"bouton soumettre"` dans le label | Double annonce ("bouton soumettre button") | Utiliser simplement `"Soumettre"` |
| `placeholder` comme seul label | Disparaît à la saisie, support inconsistant | Utiliser `<label>` visible |
| `title` pour le nommage | Support très inconsistant | Utiliser `aria-label` ou `aria-labelledby` |

---
