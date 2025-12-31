# Les 3 Helpers Essentiels

## 1. mountSuspended() - Composants async

Wrapper qui initialise le runtime Nuxt complet (auto-imports, plugins, composables) :

```typescript
// test/nuxt/ArticleCard.test.ts
import { mountSuspended } from '@nuxt/test-utils/runtime'
import ArticleCard from '~/components/content/ArticleCard.vue'

it('affiche les données article', async () => {
  const wrapper = await mountSuspended(ArticleCard, {
    props: {
      title: 'Mon Article',
      description: 'Description test',
      publishedAt: '2025-01-15',
    },
  })

  expect(wrapper.text()).toContain('Mon Article')
  expect(wrapper.text()).toContain('Description test')
})

// Test d'une page complète à une route spécifique
import App from '~/app.vue'

it('rend la page about', async () => {
  const wrapper = await mountSuspended(App, { route: '/about' })
  expect(wrapper.text()).toContain('À propos')
})
```

| mount() (vue-test-utils) | mountSuspended() (nuxt) |
|--------------------------|-------------------------|
| Sync | **Async (toujours await)** |
| Pas d'auto-imports | Auto-imports disponibles |
| Pas de composables Nuxt | Composables fonctionnels |
| Setup manuel Suspense | Suspense intégré |

## 2. mockNuxtImport() - Mock auto-imports

Mock des composables auto-importés. **Doit être au niveau module** (hoisted) :

```typescript
// test/nuxt/UserPage.test.ts
import { vi } from 'vitest'
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'

// vi.hoisted pour mocks dynamiques entre tests
const { mockRoute } = vi.hoisted(() => ({
  mockRoute: vi.fn(),
}))

mockNuxtImport('useRoute', () => mockRoute)

afterEach(() => {
  mockRoute.mockReset()
})

test('affiche l\'ID utilisateur depuis la route', async () => {
  mockRoute.mockReturnValue({ params: { id: '42' } })

  const wrapper = await mountSuspended(UserPage)
  expect(wrapper.text()).toContain('Utilisateur #42')
})

test('gère l\'absence d\'ID', async () => {
  mockRoute.mockReturnValue({ params: {} })

  const wrapper = await mountSuspended(UserPage)
  expect(wrapper.text()).toContain('Aucun utilisateur')
})
```

**Pièges courants :**

| Erreur | Cause | Solution |
|--------|-------|----------|
| Mock ignoré | Appelé 2x pour le même import | Un seul `mockNuxtImport` par import par fichier |
| Valeur undefined | Factory retourne valeur au lieu de fonction | `() => () => value` pour composables |
| Variables non définies | Référence variable locale | Utiliser `vi.hoisted()` |

## 3. registerEndpoint() - Mock API Nitro

Crée des endpoints mock pour tester les composants qui fetch des données :

```typescript
// test/nuxt/UserList.test.ts
import { registerEndpoint, mountSuspended } from '@nuxt/test-utils/runtime'
import UserList from '~/components/UserList.vue'

registerEndpoint('/api/users', () => ([
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
]))

registerEndpoint('/api/users', {
  method: 'POST',
  handler: () => ({ id: 3, name: 'Charlie', created: true }),
})

it('affiche les utilisateurs', async () => {
  const wrapper = await mountSuspended(UserList)
  expect(wrapper.text()).toContain('Alice')
  expect(wrapper.text()).toContain('Bob')
})
```

**Limitation :** Les endpoints ne peuvent pas changer entre tests. Pour du mock dynamique, utiliser MSW.

---
