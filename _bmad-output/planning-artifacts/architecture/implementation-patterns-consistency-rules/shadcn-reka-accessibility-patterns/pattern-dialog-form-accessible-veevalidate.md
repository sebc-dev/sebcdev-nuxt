# Pattern Dialog + Form Accessible (VeeValidate)

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
