# Plugin Patterns

**Plugin de formatage (universal):**

```typescript
// app/plugins/formatting.ts
export default defineNuxtPlugin(() => {
  return {
    provide: {
      formatDate: (date: string | Date, locale = 'fr-FR') => {
        return new Intl.DateTimeFormat(locale, {
          year: 'numeric',
          month: 'long',
          day: 'numeric'
        }).format(new Date(date))
      },

      readingTime: (content: string, wordsPerMinute = 200) => {
        const text = content.replace(/<[^>]*>/g, '')
        const words = text.trim().split(/\s+/).length
        return Math.ceil(words / wordsPerMinute)
      },

      excerpt: (content: string, length = 150) => {
        const text = content.replace(/<[^>]*>/g, '')
        return text.length > length
          ? text.substring(0, length) + '...'
          : text
      },

      slugify: (text: string) => {
        return text
          .toLowerCase()
          .normalize('NFD')
          .replace(/[\u0300-\u036f]/g, '')
          .replace(/[^a-z0-9]+/g, '-')
          .replace(/(^-|-$)/g, '')
      }
    }
  }
})
```

**Utilisation dans les templates:**

```vue
<template>
  <article>
    <time>{{ $formatDate(post.publishedAt) }}</time>
    <span>{{ $readingTime(post.content) }} min de lecture</span>
    <p>{{ $excerpt(post.content, 200) }}</p>
  </article>
</template>
```

**Plugin analytics client-only:**

```typescript
// app/plugins/analytics.client.ts
export default defineNuxtPlugin({
  name: 'analytics',
  parallel: true, // Ne bloque pas les autres plugins

  setup(nuxtApp) {
    const config = useRuntimeConfig()
    const router = useRouter()

    // Track navigation
    router.afterEach((to) => {
      if (typeof gtag !== 'undefined') {
        gtag('event', 'page_view', {
          page_path: to.fullPath,
          page_title: document.title
        })
      }
    })
  }
})
```

**Plugin ssr-width (prévention erreurs hydratation mobile):**

Ce plugin évite les erreurs d'hydratation sur mobile en fournissant une largeur SSR cohérente. Sans lui, les composants utilisant `useWindowSize` ou `useMediaQuery` de VueUse peuvent générer des mismatches serveur/client.

```typescript
// app/plugins/ssr-width.ts
import { provideSSRWidth } from '@vueuse/core'

export default defineNuxtPlugin((nuxtApp) => {
  // 1024px = largeur desktop par défaut pour le SSR
  // Évite les erreurs d'hydratation sur mobile où window.innerWidth diffère
  provideSSRWidth(1024, nuxtApp.vueApp)
})
```

**Quand l'utiliser :**

| Cas d'usage | Nécessite ssr-width |
|-------------|---------------------|
| `useWindowSize()` dans composant | ✅ Oui |
| `useMediaQuery()` pour responsive | ✅ Oui |
| Breakpoints dynamiques (menu mobile) | ✅ Oui |
| Composants purement statiques | ❌ Non |

**Alternative sans VueUse :**

Si VueUse n'est pas utilisé, créer un composable équivalent :

```typescript
// app/composables/useSSRSafeWidth.ts
export function useSSRSafeWidth(defaultWidth = 1024) {
  const width = ref(defaultWidth)

  if (import.meta.client) {
    width.value = window.innerWidth
    useEventListener(window, 'resize', () => {
      width.value = window.innerWidth
    })
  }

  return width
}
```

**Composable useSystemThemeSync (synchronisation thème système temps réel) :**

Ce composable réagit aux changements de thème système en temps réel (passage mode clair/sombre macOS, Windows). Utile quand `colorMode.preference === 'system'`.

```typescript
// app/composables/useSystemThemeSync.ts
export function useSystemThemeSync() {
  const colorMode = useColorMode()

  onMounted(() => {
    if (!window.matchMedia) return

    const query = window.matchMedia('(prefers-color-scheme: dark)')

    const handler = (e: MediaQueryListEvent) => {
      if (colorMode.preference === 'system') {
        // @nuxtjs/color-mode gère automatiquement cette synchro
        // Ajoutez ici une logique custom si nécessaire :
        // - Analytics tracking du changement
        // - Notification utilisateur
        // - Mise à jour d'autres états liés au thème
      }
    }

    query.addEventListener('change', handler)
    onUnmounted(() => query.removeEventListener('change', handler))
  })
}
```

**Usage :**

```vue
<script setup>
// Dans app.vue ou un layout
useSystemThemeSync()
</script>
```

**Note :** `@nuxtjs/color-mode` gère automatiquement la synchronisation basique. Ce composable est utile pour des actions supplémentaires (analytics, notifications, etc.).
