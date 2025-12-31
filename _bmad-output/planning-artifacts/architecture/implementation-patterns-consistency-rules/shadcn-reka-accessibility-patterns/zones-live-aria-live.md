# Zones Live (aria-live)

Les zones live permettent aux lecteurs d'écran d'annoncer les changements de contenu **sans déplacer le focus**.

## Règle cardinale

> **La zone live DOIT exister dans le DOM AVANT que son contenu ne change.**
> Une zone créée dynamiquement et immédiatement remplie ne sera PAS annoncée.

## polite vs assertive

| Valeur | Comportement | Cas d'usage |
|--------|--------------|-------------|
| `polite` (défaut) | Attend que l'utilisateur soit inactif | Messages succès, mises à jour panier, résultats recherche |
| `assertive` | Interrompt immédiatement | Erreurs critiques, alertes sécurité, messages urgents |

**Règle** : Toujours defaulter à `polite` pour éviter de submerger les utilisateurs.

## Attributs complémentaires

| Attribut | Usage |
|----------|-------|
| `aria-atomic="true"` | Annonce la région **entière** lors de tout changement (horloges, prix) |
| `aria-relevant="additions"` | Annonce uniquement les ajouts |
| `aria-relevant="removals"` | Annonce uniquement les suppressions |
| `aria-relevant="text"` | Annonce les modifications textuelles |

## Pattern Toast accessible

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

## Intégration Sonner (shadcn-vue Toast)

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
