# Sécurité Cloudflare (SSG)

## Rate Limiting WAF (gratuit)

Le rate limiting applicatif est **impossible en SSG pur** sans serveur. Utiliser le WAF Cloudflare (1 règle gratuite) :

**Configuration via Dashboard** (Security → WAF → Rate limiting rules) :

| Paramètre | Valeur |
|-----------|--------|
| Expression | `(http.request.uri.path contains "/api/")` |
| Caractéristique | IP Address |
| Seuil | 100 requêtes / minute |
| Action | Managed Challenge |

**Alternatives selon le cas :**

| Protection | Solution | Coût |
|------------|----------|------|
| Protection globale bots | Bot Fight Mode | Gratuit |
| Rate limiting API | WAF Rate Limiting | Gratuit (1 règle) |
| Formulaires | Cloudflare Turnstile | Gratuit |
| API endpoints | Pages Functions + Workers | Inclus |

## Cloudflare Turnstile (Formulaires)

Alternative CAPTCHA gratuite et respectueuse de la vie privée. Utiliser pour les formulaires de contact, commentaires, newsletter.

**1. Configuration Dashboard Cloudflare :**
- Naviguer vers Turnstile → Add Site
- Récupérer `SITE_KEY` (public) et `SECRET_KEY` (privé)

**2. Composant Vue côté client :**

```vue
<!-- app/components/TurnstileWidget.vue -->
<script setup lang="ts">
const siteKey = useRuntimeConfig().public.turnstileSiteKey

const emit = defineEmits<{
  verified: [token: string]
  error: []
}>()

onMounted(() => {
  if (window.turnstile) {
    window.turnstile.render('#turnstile-container', {
      sitekey: siteKey,
      callback: (token: string) => emit('verified', token),
      'error-callback': () => emit('error')
    })
  }
})
</script>

<template>
  <div id="turnstile-container" />
</template>
```

**3. Validation côté Pages Function :**

```typescript
// functions/api/contact.ts
interface TurnstileResponse {
  success: boolean
  'error-codes'?: string[]
}

export async function onRequestPost(context: EventContext<Env, string, unknown>) {
  const formData = await context.request.formData()
  const token = formData.get('cf-turnstile-response') as string

  // Validation Turnstile
  const verification = await fetch(
    'https://challenges.cloudflare.com/turnstile/v0/siteverify',
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        secret: context.env.TURNSTILE_SECRET_KEY,
        response: token,
        remoteip: context.request.headers.get('cf-connecting-ip')
      })
    }
  ).then(r => r.json() as Promise<TurnstileResponse>)

  if (!verification.success) {
    return new Response(JSON.stringify({ error: 'Vérification échouée' }), {
      status: 403,
      headers: { 'Content-Type': 'application/json' }
    })
  }

  // Traitement du formulaire...
  return new Response(JSON.stringify({ success: true }), {
    headers: { 'Content-Type': 'application/json' }
  })
}
```

**4. Configuration nuxt.config.ts :**

```typescript
runtimeConfig: {
  turnstileSecretKey: '',  // TURNSTILE_SECRET_KEY env var
  public: {
    turnstileSiteKey: ''   // NUXT_PUBLIC_TURNSTILE_SITE_KEY env var
  }
}
```

**5. Script Turnstile dans app.vue :**

```typescript
useHead({
  script: [{
    src: 'https://challenges.cloudflare.com/turnstile/v0/api.js',
    async: true,
    defer: true
  }]
})
```

**Note :** Turnstile est prévu pour la phase post-MVP (formulaire de contact, commentaires).
