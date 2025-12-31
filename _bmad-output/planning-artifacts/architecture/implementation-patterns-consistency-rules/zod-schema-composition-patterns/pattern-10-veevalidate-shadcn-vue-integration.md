# Pattern 10 : VeeValidate + shadcn-vue integration

Pattern complet pour intégrer Zod avec les formulaires shadcn-vue via VeeValidate.

## Installation

```bash
pnpm add vee-validate @vee-validate/zod
```

## Composant formulaire typé

```vue
<!-- components/ContactForm.vue -->
<script setup lang="ts">
import { useForm } from 'vee-validate'
import { toTypedSchema } from '@vee-validate/zod'
import { z } from 'zod/v4'
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage
} from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
import { Button } from '@/components/ui/button'

// Schema Zod 4 avec messages FR
const contactSchema = z.object({
  email: z.email({ error: 'Adresse email invalide' }),
  name: z.string().min(2, { error: 'Le nom doit contenir au moins 2 caractères' }),
  message: z.string().min(10, { error: 'Message trop court (min. 10 caractères)' })
})

type ContactForm = z.infer<typeof contactSchema>

// Conversion pour VeeValidate
const validationSchema = toTypedSchema(contactSchema)

const { handleSubmit, errors, isSubmitting } = useForm<ContactForm>({
  validationSchema,
  initialValues: {
    email: '',
    name: '',
    message: ''
  }
})

const onSubmit = handleSubmit(async (values) => {
  // values est typé : { email: string; name: string; message: string }
  await $fetch('/api/contact', { method: 'POST', body: values })
})
</script>

<template>
  <Form @submit="onSubmit" class="space-y-4">
    <FormField v-slot="{ componentField }" name="email">
      <FormItem>
        <FormLabel>Email</FormLabel>
        <FormControl>
          <Input type="email" placeholder="vous@exemple.com" v-bind="componentField" />
        </FormControl>
        <FormMessage />
      </FormItem>
    </FormField>

    <FormField v-slot="{ componentField }" name="name">
      <FormItem>
        <FormLabel>Nom</FormLabel>
        <FormControl>
          <Input type="text" placeholder="Votre nom" v-bind="componentField" />
        </FormControl>
        <FormMessage />
      </FormItem>
    </FormField>

    <FormField v-slot="{ componentField }" name="message">
      <FormItem>
        <FormLabel>Message</FormLabel>
        <FormControl>
          <Textarea placeholder="Votre message..." v-bind="componentField" />
        </FormControl>
        <FormMessage />
      </FormItem>
    </FormField>

    <Button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Envoi...' : 'Envoyer' }}
    </Button>
  </Form>
</template>
```

## ⚠️ Caveat VeeValidate + Zod

Les méthodes `.refine()` et `.superRefine()` **ne s'exécutent pas** quand des clés d'objet sont manquantes. Solution :

```typescript
// ❌ refine() ignoré si passwordConfirm est undefined
const schema = z.object({
  password: z.string(),
  passwordConfirm: z.string()
}).refine(d => d.password === d.passwordConfirm)

// ✅ Toujours fournir des valeurs initiales
const { handleSubmit } = useForm({
  validationSchema: toTypedSchema(schema),
  initialValues: {
    password: '',
    passwordConfirm: ''  // REQUIS pour que refine() s'exécute
  }
})
```

---
