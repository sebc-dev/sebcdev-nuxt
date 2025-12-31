# Middleware Patterns

**Organisation avec préfixe numérique pour ordre d'exécution :**

```
app/middleware/
├── 01.analytics.global.ts    # Exécuté en premier
├── 02.redirects.global.ts    # Exécuté en deuxième
├── 03.trailing-slash.global.ts
└── auth.ts                   # Named middleware (à la demande)
```

L'ordre est alphabétique pour les globaux (triés comme string : `01` < `02` < `10`).

**Règle critique : NE JAMAIS utiliser `next()` (pattern Vue Router 3)**

```typescript
// app/middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
  const user = useState('user')

  // ✅ Correct : navigateTo pour redirection
  if (!user.value?.isAuthenticated) {
    return navigateTo('/login')
  }

  // ✅ Correct : navigateTo avec code redirect
  if (to.path === '/ancien-chemin') {
    return navigateTo('/nouveau-chemin', { redirectCode: 301 })
  }

  // ✅ Correct : abortNavigation pour bloquer
  if (user.value?.isBanned) {
    return abortNavigation()
  }

  // ✅ Correct : erreur personnalisée
  if (!hasPermission(to)) {
    return abortNavigation(
      createError({
        statusCode: 403,
        statusMessage: 'Accès interdit'
      })
    )
  }

  // ✅ Correct : return void pour continuer
  return
})
```

**Middleware global de redirections:**

```typescript
// app/middleware/02.redirects.global.ts
const redirects: Record<string, string> = {
  '/articles': '/blog',
  '/posts': '/blog',
  '/tag': '/tags',
}

export default defineNuxtRouteMiddleware((to) => {
  const redirect = redirects[to.path]
  if (redirect) {
    return navigateTo(redirect, { redirectCode: 301 })
  }
})
```

**Trailing slash redirect (SEO) :**

```typescript
// app/middleware/03.trailing-slash.global.ts
export default defineNuxtRouteMiddleware(({ path, query, hash }) => {
  // Ne pas traiter la racine ou les paths sans trailing slash
  if (path === '/' || !path.endsWith('/')) return

  const nextPath = path.replace(/\/+$/, '') || '/'
  return navigateTo({ path: nextPath, query, hash }, { redirectCode: 301 })
})
```

**Skip hydration pour middleware analytics (client-side) :**

```typescript
// app/middleware/01.analytics.global.ts
export default defineNuxtRouteMiddleware((to, from) => {
  // Skip côté serveur
  if (import.meta.server) return

  // Skip pendant l'hydratation initiale
  const nuxtApp = useNuxtApp()
  if (nuxtApp.isHydrating && nuxtApp.payload.serverRendered) return

  // Analytics tracking (client-side uniquement, après hydratation)
  if (typeof gtag !== 'undefined') {
    gtag('config', 'GA_MEASUREMENT_ID', { page_path: to.fullPath })
  }
})
```

**Validation de routes avec definePageMeta:**

```vue
<!-- pages/blog/[slug].vue -->
<script setup lang="ts">
definePageMeta({
  validate(route) {
    const slug = route.params.slug as string
    // Valide format slug (lettres, chiffres, tirets)
    if (!/^[a-z0-9-]+$/.test(slug)) {
      return {
        statusCode: 400,
        statusMessage: 'Format de slug invalide'
      }
    }
    return true
  },
  middleware: ['draft-protection']
})
</script>
```
