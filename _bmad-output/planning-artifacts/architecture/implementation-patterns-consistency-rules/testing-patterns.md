# Testing Patterns

## Installation

```bash
pnpm add -D @nuxt/test-utils vitest @vue/test-utils happy-dom @vitest/coverage-v8 playwright-core
```

Ajouter le module dans `nuxt.config.ts` :

```typescript
export default defineNuxtConfig({
  modules: ['@nuxt/test-utils/module'],
})
```

---

## Configuration Vitest pour Nuxt 4

**Fichier `vitest.config.ts` à la racine du projet :**

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import { defineVitestProject } from '@nuxt/test-utils/config'

export default defineConfig({
  test: {
    projects: [
      // Tests unitaires rapides - environnement Node
      {
        test: {
          name: 'unit',
          include: ['test/unit/**/*.{test,spec}.ts'],
          environment: 'node',
          globals: true,
        },
      },
      // Tests composants Nuxt - environnement complet
      await defineVitestProject({
        test: {
          name: 'nuxt',
          include: ['test/nuxt/**/*.{test,spec}.ts'],
          environment: 'nuxt',
          environmentOptions: {
            nuxt: {
              domEnvironment: 'happy-dom',  // Plus rapide que jsdom
              mock: {
                intersectionObserver: true,
                indexedDb: true,
              },
            },
          },
        },
      }),
    ],
  },
})
```

**Prérequis :** `"type": "module"` dans `package.json` ou renommer en `vitest.config.mts`.

**Scripts npm recommandés :**

```json
{
  "scripts": {
    "test": "vitest",
    "test:unit": "vitest --project unit",
    "test:nuxt": "vitest --project nuxt",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
}
```

---

## Setup File Complet (test/setup.ts)

**Configuration essentielle pour happy-dom et shadcn-vue/Reka UI :**

```typescript
// test/setup.ts
import { vi, afterEach } from 'vitest'
import { enableAutoUnmount, config } from '@vue/test-utils'
import { PropertySymbol } from 'happy-dom'

// ═══════════════════════════════════════════════════════════════
// AUTO-CLEANUP - Démonte automatiquement les composants
// ═══════════════════════════════════════════════════════════════
enableAutoUnmount(afterEach)

afterEach(() => {
  vi.restoreAllMocks()
  vi.clearAllTimers()
  config.global.renderStubDefaultSlot = false
})

// ═══════════════════════════════════════════════════════════════
// TIMER WORKAROUND - Compatibilité happy-dom + Vitest
// ═══════════════════════════════════════════════════════════════
const browserWindow =
  global.document?.[PropertySymbol.ownerWindow] ||
  global.document?.[PropertySymbol.window]

if (browserWindow) {
  global.setTimeout = browserWindow.setTimeout
  global.clearTimeout = browserWindow.clearTimeout
  global.setInterval = browserWindow.setInterval
  global.clearInterval = browserWindow.clearInterval
  global.requestAnimationFrame = browserWindow.requestAnimationFrame
  global.cancelAnimationFrame = browserWindow.cancelAnimationFrame
}

// ═══════════════════════════════════════════════════════════════
// BROWSER API MOCKS - APIs manquantes dans happy-dom
// ═══════════════════════════════════════════════════════════════
Element.prototype.getBoundingClientRect = vi.fn(() => ({
  width: 120, height: 120,
  top: 0, left: 0, bottom: 120, right: 120,
  x: 0, y: 0,
  toJSON: () => {},
}))

global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}))

Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
})

// ═══════════════════════════════════════════════════════════════
// REKA UI SHIMS - Requis pour shadcn-vue composants
// ═══════════════════════════════════════════════════════════════
window.PointerEvent = MouseEvent as typeof PointerEvent

global.DOMRect = class DOMRect {
  x = 0; y = 0; width = 0; height = 0
  top = 0; right = 0; bottom = 0; left = 0
  toJSON() { return this }
}

Element.prototype.scrollIntoView = vi.fn()
Element.prototype.hasPointerCapture = vi.fn(() => false)
Element.prototype.releasePointerCapture = vi.fn()
Element.prototype.setPointerCapture = vi.fn()
```

**Référencer dans vitest.config.ts :**

```typescript
test: {
  setupFiles: ['./test/setup.ts'],
  // ...
}
```

---

## Structure de Dossiers

Les tests restent à la **racine du projet**, pas dans `app/` :

```
project/
├── app/                     # srcDir Nuxt 4
│   ├── components/
│   ├── composables/
│   └── pages/
├── test/                    # Tests à la racine
│   ├── unit/               # Tests purs (Node) - rapides
│   │   ├── utils.test.ts
│   │   └── formatters.test.ts
│   └── nuxt/               # Tests composants (Nuxt env)
│       ├── ArticleCard.test.ts
│       └── useReadingTime.test.ts
├── vitest.config.ts
└── nuxt.config.ts
```

| Dossier | Environnement | Usage | Vitesse |
|---------|---------------|-------|---------|
| `test/unit/` | Node | Fonctions pures, utils, validations | Rapide |
| `test/nuxt/` | Nuxt | Composants, composables, pages | Plus lent |

---

## Les 3 Helpers Essentiels

### 1. mountSuspended() - Composants async

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

### 2. mockNuxtImport() - Mock auto-imports

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

### 3. registerEndpoint() - Mock API Nitro

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

## Testing Composables Nuxt

### useState

```typescript
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'
import { ref } from 'vue'

mockNuxtImport('useState', () => {
  return (key: string, init?: () => any) => ref(init ? init() : undefined)
})

it('initialise le state', async () => {
  const wrapper = await mountSuspended(CounterComponent)
  expect(wrapper.text()).toContain('Count: 0')
})
```

### useFetch / useAsyncData

```typescript
import { vi } from 'vitest'
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'
import { ref } from 'vue'

const { mockUseFetch } = vi.hoisted(() => ({
  mockUseFetch: vi.fn(),
}))

mockNuxtImport('useFetch', () => mockUseFetch)

afterEach(() => mockUseFetch.mockReset())

test('affiche état loading', async () => {
  mockUseFetch.mockReturnValue({
    data: ref(null),
    pending: ref(true),
    error: ref(null),
  })

  const wrapper = await mountSuspended(ArticleList)
  expect(wrapper.text()).toContain('Chargement...')
})

test('affiche les articles', async () => {
  mockUseFetch.mockReturnValue({
    data: ref([{ title: 'Guide Testing' }]),
    pending: ref(false),
    error: ref(null),
  })

  const wrapper = await mountSuspended(ArticleList)
  expect(wrapper.text()).toContain('Guide Testing')
})
```

---

## Testing Nuxt Content 3

### Mock des données Content

```typescript
import { mockNuxtImport, mountSuspended } from '@nuxt/test-utils/runtime'
import { ref } from 'vue'
import BlogPost from '~/pages/blog/[slug].vue'

mockNuxtImport('useAsyncData', () => {
  return () => ({
    data: ref({
      title: 'Article Test',
      description: 'Description test',
      body: {
        type: 'root',
        children: [
          { type: 'element', tag: 'p', children: [{ type: 'text', value: 'Contenu article' }] }
        ],
      },
    }),
    pending: ref(false),
    error: ref(null),
  })
})

it('rend la page article', async () => {
  const wrapper = await mountSuspended(BlogPost, {
    route: '/blog/article-test',
  })
  expect(wrapper.text()).toContain('Article Test')
})
```

### Stub ContentRenderer

```typescript
import { mockComponent, mountSuspended } from '@nuxt/test-utils/runtime'

mockComponent('ContentRenderer', {
  props: ['value'],
  template: '<div class="content-stub">{{ value?.title }}</div>',
})
```

---

## Configuration Coverage SSG

Pour les projets SSG, **définir explicitement `include`** pour éviter l'inflation due aux auto-imports :

```typescript
// vitest.config.ts
export default defineVitestConfig({
  test: {
    environment: 'nuxt',
    coverage: {
      provider: 'v8',
      enabled: true,

      // Inclusion explicite (évite inflation auto-imports)
      include: [
        'app/components/**/*.vue',
        'app/composables/**/*.ts',
        'app/utils/**/*.ts',
      ],

      exclude: [
        '**/.nuxt/**',
        '**/.output/**',
        '**/node_modules/**',
        '**/*.d.ts',
        'test/**',
      ],

      reporter: ['text', 'html', 'lcov'],
      reportsDirectory: './coverage',

      thresholds: {
        lines: 80,
        functions: 80,
        branches: 75,
        statements: 80,
      },
    },
  },
})
```

---

## Breaking Changes Nuxt 3 → Nuxt 4

| Aspect | Nuxt 3 | Nuxt 4 |
|--------|--------|--------|
| **Data fetching initial** | `data.value` = `null` | `data.value` = `undefined` |
| **Noms composants** | `'MyComponent'` | `'FolderMyComponent'` (normalisé) |
| **Vitest workspace** | `vitest.workspace.ts` | `projects` dans `vitest.config.ts` |

**Mise à jour des assertions :**

```typescript
// ❌ Nuxt 3
expect(data.value).toBe(null)

// ✅ Nuxt 4
expect(data.value).toBe(undefined)
```

**Mise à jour des sélecteurs :**

```typescript
// ❌ Nuxt 3
wrapper.findComponent({ name: 'MyComponent' })

// ✅ Nuxt 4
wrapper.findComponent({ name: 'ContentMyComponent' })  // Dossier + nom
```

**Codemods disponibles :**

```bash
npx codemod@latest nuxt/4/file-structure
npx codemod@latest nuxt/4/default-data-error-value
```

---

## happy-dom vs jsdom

| Critère | happy-dom | jsdom |
|---------|-----------|-------|
| **Vitesse** | ~3x plus rapide | Plus lent |
| **Compatibilité** | 95% des cas | 100% |
| **Recommandation Nuxt** | ✅ Par défaut | Fallback si problèmes |

**Quand utiliser jsdom :**
- Tests nécessitant `canvas` ou `SVG` complexes
- Composants avec `MutationObserver` avancés
- Si happy-dom cause des erreurs spécifiques
- **Tests d'accessibilité avec vitest-axe** (obligatoire)

**Basculer par fichier avec directive :**

```typescript
// @vitest-environment jsdom
import { axe, toHaveNoViolations } from 'vitest-axe'
import { expect } from 'vitest'

expect.extend(toHaveNoViolations)

describe('Dialog Accessibility', () => {
  it('pas de violations WCAG', async () => {
    const { container } = render(Dialog, { props: { open: true } })
    const results = await axe(container)
    expect(results).toHaveNoViolations()
  })
})
```

---

## Testing Teleport/Portal (shadcn-vue)

Les composants Dialog, Sheet, Dropdown de shadcn-vue utilisent `Teleport`. Deux stratégies :

### Stratégie 1 : Créer le portal target

```typescript
beforeEach(() => {
  const el = document.createElement('div')
  el.id = 'radix-portal'
  document.body.appendChild(el)
})

afterEach(() => {
  document.body.innerHTML = ''
})

it('ouvre le dialog', async () => {
  const wrapper = await mountSuspended(MyDialog, {
    props: { open: true },
  })
  expect(document.body.textContent).toContain('Contenu dialog')
})
```

### Stratégie 2 : Stub Teleport globalement

```typescript
const wrapper = await mountSuspended(MyDialog, {
  props: { open: true },
  global: {
    stubs: {
      teleport: true,
    },
  },
})

// Le contenu reste dans le wrapper au lieu d'être téléporté
expect(wrapper.text()).toContain('Contenu dialog')
```

| Stratégie | Avantage | Inconvénient |
|-----------|----------|--------------|
| Portal target | Teste le vrai comportement | Setup/teardown requis |
| Stub Teleport | Plus simple | Ne teste pas le positionnement réel |

---

## Testing Pinia Stores

**Installation :**

```bash
pnpm add -D @pinia/testing
```

### Pattern avec createTestingPinia

```typescript
import { mount } from '@vue/test-utils'
import { createTestingPinia } from '@pinia/testing'
import { vi } from 'vitest'
import Counter from './Counter.vue'
import { useCounterStore } from '~/stores/counter'

describe('Counter avec Pinia', () => {
  it('affiche le count depuis le store', () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [
          createTestingPinia({
            createSpy: vi.fn,
            initialState: {
              counter: { count: 10 },
            },
          }),
        ],
      },
    })

    expect(wrapper.text()).toContain('10')
  })

  it('appelle l\'action store au clic', async () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [createTestingPinia({ createSpy: vi.fn })],
      },
    })

    const store = useCounterStore()
    await wrapper.find('button').trigger('click')

    // Actions stubbées par défaut
    expect(store.increment).toHaveBeenCalledTimes(1)
  })

  it('exécute les vraies actions avec stubActions: false', async () => {
    const wrapper = mount(Counter, {
      global: {
        plugins: [
          createTestingPinia({
            createSpy: vi.fn,
            stubActions: false,  // Actions réelles
          }),
        ],
      },
    })

    const store = useCounterStore()
    await wrapper.find('button').trigger('click')
    expect(store.count).toBe(1)
  })
})
```

---

## Coverage Thresholds par Type

Configuration tiered basée sur la testabilité de chaque type de code :

```typescript
// vitest.config.ts
coverage: {
  provider: 'v8',
  include: [
    'app/components/**/*.{vue,ts}',
    'app/composables/**/*.ts',
    'app/utils/**/*.ts',
    'app/stores/**/*.ts',
    'app/pages/**/*.vue',
  ],

  thresholds: {
    // Seuils globaux
    lines: 75,
    functions: 70,
    branches: 70,
    statements: 75,
    perFile: true,

    // Seuils par type (priorité haute)
    'app/composables/**/*.ts': {
      lines: 90, functions: 90, branches: 85, statements: 90,
    },
    'app/utils/**/*.ts': {
      lines: 90, functions: 90, branches: 85, statements: 90,
    },

    // Seuils par type (priorité moyenne)
    'app/stores/**/*.ts': {
      lines: 85, functions: 85, branches: 80, statements: 85,
    },
    'app/components/**/*.vue': {
      lines: 70, functions: 70, branches: 65, statements: 70,
    },

    // Seuils par type (priorité basse - E2E préférable)
    'app/pages/**/*.vue': {
      lines: 60, functions: 60, branches: 55, statements: 60,
    },
  },
}
```

| Type | Target | Rationale |
|------|--------|-----------|
| Composables | 90% | Logique pure, haute réutilisation |
| Utils | 90% | Fonctions isolées, critiques |
| Stores | 85% | State management testable |
| Components | 70% | Focus sur comportement utilisateur |
| Pages | 60% | Mieux couvertes par E2E |

---

## Snapshot Testing Guidelines

### Quand utiliser les snapshots

| ✅ Utiliser pour | ❌ Éviter pour |
|------------------|----------------|
| Messages d'erreur | Interactions utilisateur |
| Transformations de code | Contenu dynamique (timestamps, IDs) |
| Petites structures (<10 lignes) | Arbres de composants géants |
| Output CSS-in-JS | Structures internes Reka UI |

### Inline vs File snapshots

```typescript
// ✅ Inline snapshot - output court
it('formate la devise correctement', () => {
  const result = formatCurrency(1234.56, 'EUR')
  expect(result).toMatchInlineSnapshot(`"1 234,56 €"`)
})

// ✅ File snapshot - structure plus grande
it('rend le card complet', () => {
  const wrapper = mount(Card, {
    props: { title: 'Hello', content: 'World' },
  })
  expect(wrapper.html()).toMatchSnapshot()
})
```

### Gérer les valeurs dynamiques

```typescript
// Mock du temps pour snapshots consistants
beforeEach(() => {
  vi.useFakeTimers()
  vi.setSystemTime(new Date('2025-01-01T12:00:00Z'))
})

afterEach(() => {
  vi.useRealTimers()
})

// Property matchers pour IDs générés
it('crée un user avec champs générés', () => {
  const user = createUser({ name: 'John' })

  expect(user).toMatchSnapshot({
    id: expect.any(String),
    createdAt: expect.any(Date),
    updatedAt: expect.any(Date),
  })
})
```

### Anti-patterns snapshots

- ❌ Snapshots >200 lignes - trop de bruit pour la review
- ❌ `expect(2 + 2).toMatchSnapshot()` - n'apporte rien
- ❌ `vitest -u` sans review - toujours vérifier les diffs
- ❌ Snapshoter les structures internes Reka UI - trop fragiles

---

## Anti-patterns Testing

### Ne pas tester les détails d'implémentation

```typescript
// ❌ BAD: Accès à l'état interne
expect(wrapper.vm.internalCounter).toBe(5)

// ✅ GOOD: Test du comportement observable
expect(wrapper.find('[data-testid="count"]').text()).toBe('5')
```

### Ne pas utiliser setData avec Composition API

```typescript
// ❌ BAD: setData ne fonctionne pas avec script setup
await wrapper.setData({ count: 5 })

// ✅ GOOD: Déclencher l'action qui change l'état
await wrapper.find('button').trigger('click')
```

### Toujours await les opérations async

```typescript
// ❌ BAD: await manquant
wrapper.find('button').trigger('click')
expect(wrapper.text()).toContain('Updated')  // Peut échouer!

// ✅ GOOD: Await le trigger
await wrapper.find('button').trigger('click')
expect(wrapper.text()).toContain('Updated')
```

### Ne pas over-stubber

```typescript
// ❌ BAD: shallowMount stubbe tout
const wrapper = shallowMount(Component)

// ✅ GOOD: mount + stub sélectif si nécessaire
const wrapper = mount(Component, {
  global: {
    stubs: {
      HeavyExternalComponent: true,  // Uniquement les dépendances lourdes
    },
  },
})
```

### Éviter les assertions sur wrapper.vm

```typescript
// ❌ BAD: Couplé à l'implémentation
expect(wrapper.vm.isLoading).toBe(true)

// ✅ GOOD: Test ce que l'utilisateur voit
expect(wrapper.find('.loading-spinner').exists()).toBe(true)
```
