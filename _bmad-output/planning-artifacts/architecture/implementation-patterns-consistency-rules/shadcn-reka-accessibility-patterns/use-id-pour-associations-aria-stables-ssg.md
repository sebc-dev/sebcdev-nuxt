# useId() pour Associations ARIA Stables (SSG)

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
