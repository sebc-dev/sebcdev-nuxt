# Structure Patterns

**Path Aliases Nuxt 4:**

| Alias | Pointe vers | Usage |
|-------|-------------|-------|
| `~/` ou `@/` | `app/` | Composants, composables, pages, utils |
| `~~/` ou `@@/` | Racine projet | Accès server/, content/, public/ |
| `#shared` | `shared/` | Types et utilitaires partagés app/server |

```typescript
// Exemples d'imports
import { ArticleCard } from '~/components/content/ArticleCard.vue'  // → app/components/
import { useReadingTime } from '~/composables/useReadingTime'       // → app/composables/
import type { Article } from '#shared/types/article'                // → shared/types/
import { formatSlug } from '~~/server/utils/slug'                   // → server/utils/
```

---

**Test Organization:**
- Co-located avec le code source
- `Component.vue` → `Component.test.ts` (même dossier)

**Composables Organization:**
- Un fichier par composable
- Flat structure dans `app/composables/`
- Préfixe `use` obligatoire
- Retour: objet avec refs (jamais `reactive`)

```
app/composables/
├── useReadingTime.ts
├── useArticleFilters.ts
├── useTableOfContents.ts
└── useSeoMeta.ts
```

---

## Critères d'Extraction de Composable

**Quand créer un composable ?** Un composable est justifié quand la logique combine **state réactif + lifecycle hooks** et apparaît dans 3+ composants.

**Checklist d'extraction :**

| Critère | Composable ? | Exemple |
|---------|-------------|---------|
| State réactif qui change dans le temps | ✅ Oui | Position souris, scroll, timers |
| Lifecycle hooks requis (onMounted, watch) | ✅ Oui | Event listeners, cleanup |
| Pattern identique dans 3+ composants | ✅ Oui | Fetch avec loading/error |
| Async data fetching avec états | ✅ Oui | `useArticle()`, `usePosts()` |
| Setup/cleanup d'event listeners | ✅ Oui | `useEventListener()` |
| Simple computed ou one-liner | ❌ Non | Formater une date |
| Fonction utilitaire stateless | ❌ Non | `formatSlug()`, `truncate()` |

**Anti-pattern : composable stateless**

```typescript
// ❌ INUTILE - simple wrapper sans valeur ajoutée
export function useFormatDate(date: Date) {
  return date.toLocaleDateString('fr-FR')
}

// ✅ CORRECT - fonction utilitaire dans app/utils/
export function formatDate(date: Date, locale = 'fr-FR') {
  return date.toLocaleDateString(locale)
}
```

**Règle simple :** Si la fonction n'utilise pas `ref()`, `computed()`, `watch()`, ou `onMounted()`, c'est une **utility function**, pas un composable. Placez-la dans `app/utils/`.

---

**Pattern Composable Avancé avec MaybeRefOrGetter:**

```typescript
// app/composables/useBlogPost.ts
import type { MaybeRefOrGetter } from 'vue'

interface BlogPost {
  id: string
  slug: string
  title: string
  content: string
}

interface UseBlogPostReturn {
  post: Readonly<Ref<BlogPost | null>>
  isLoading: Readonly<Ref<boolean>>
  error: Readonly<Ref<Error | null>>
  refresh: () => Promise<void>
}

export function useBlogPost(
  slug: MaybeRefOrGetter<string>  // Accepte valeur, ref ou getter
): UseBlogPostReturn {
  // shallowRef pour objets complexes - évite la réactivité profonde
  const post = shallowRef<BlogPost | null>(null)
  const isLoading = ref(false)
  const error = shallowRef<Error | null>(null)

  const fetchPost = async () => {
    const slugValue = toValue(slug) // Unwrap ref, getter ou valeur
    if (!slugValue) return

    isLoading.value = true
    error.value = null

    try {
      post.value = await $fetch(`/api/posts/${slugValue}`)
    } catch (e) {
      error.value = e as Error
      post.value = null
    } finally {
      isLoading.value = false
    }
  }

  // watchEffect track automatiquement les dépendances
  watchEffect(() => {
    toValue(slug)
    fetchPost()
  })

  return {
    post: readonly(post),      // ✅ Retourner readonly pour immutabilité
    isLoading: readonly(isLoading),
    error: readonly(error),
    refresh: fetchPost
  }
}
```

**Utilisation flexible du composable:**

```vue
<script setup lang="ts">
const route = useRoute()

// Toutes ces syntaxes fonctionnent grâce à MaybeRefOrGetter
useBlogPost('mon-article')                    // Valeur directe
useBlogPost(ref('mon-article'))               // Ref
useBlogPost(() => route.params.slug as string) // Getter réactif
</script>
```

**Composables Nuxt Content 3 (collections par langue):**

```typescript
// app/composables/useBlogPost.ts
import type { Collections } from '@nuxt/content'

export function useBlogPost(path: MaybeRefOrGetter<string>) {
  const { locale } = useI18n()

  // Collection dynamique selon la locale
  const collection = computed(() => `articles_${locale.value}` as keyof Collections)

  // Article principal
  const { data: post, pending } = useAsyncData(
    `post-${locale.value}-${toValue(path)}`,
    () => queryCollection(collection.value).path(toValue(path)).first(),
    { watch: [locale] }
  )

  // Navigation prev/next avec queryCollectionItemSurroundings
  const { data: surroundings } = useAsyncData(
    `surroundings-${locale.value}-${toValue(path)}`,
    () => queryCollectionItemSurroundings(collection.value, toValue(path), {
      fields: ['_path', 'title', 'description', 'publishedAt']
    }),
    { watch: [locale] }
  )

  const prev = computed(() => surroundings.value?.[0] ?? null)
  const next = computed(() => surroundings.value?.[1] ?? null)

  return {
    post,
    isLoading: pending,
    prev,
    next
  }
}
```

```typescript
// app/composables/useBlogPosts.ts
import type { Collections } from '@nuxt/content'

export function useBlogPosts() {
  const { locale } = useI18n()
  const posts = shallowRef<Article[]>([])
  const isLoading = shallowRef(false)

  // Collection dynamique selon la locale
  const collection = computed(() => `articles_${locale.value}` as keyof Collections)

  const fetchPosts = async (options: {
    limit?: number
    pillar?: string
    tag?: string
  } = {}) => {
    isLoading.value = true

    try {
      let query = queryCollection(collection.value)
        .select('_path', 'title', 'description', 'publishedAt', 'pillar', 'tags')
        .where('draft', '=', false)
        .order('publishedAt', 'DESC')

      if (options.pillar) {
        query = query.where('pillar', '=', options.pillar)
      }

      if (options.tag) {
        query = query.where('tags', 'LIKE', `%${options.tag}%`)
      }

      if (options.limit) {
        query = query.limit(options.limit)
      }

      posts.value = await query.all()
    } finally {
      isLoading.value = false
    }
  }

  // Re-fetch automatique au changement de langue
  watch(locale, () => fetchPosts())

  return {
    posts: readonly(posts),
    isLoading: readonly(isLoading),
    fetchPosts
  }
}
```

**shallowRef pour grandes listes (>10k éléments):**

```typescript
// app/composables/useBlogArchive.ts
export function useBlogArchive() {
  // shallowRef - ne track que .value, pas les propriétés internes
  const posts = shallowRef<BlogPost[]>([])

  const addPost = (post: BlogPost) => {
    // Obligation de remplacer le tableau entier pour déclencher réactivité
    posts.value = [...posts.value, post]
  }

  // computed avec comparaison intelligente (Vue 3.4+)
  const stats = computed((oldValue) => {
    const newValue = {
      total: posts.value.length,
      published: posts.value.filter(p => p.status === 'published').length
    }

    // Évite de déclencher les effets si valeurs identiques
    if (oldValue?.total === newValue.total &&
        oldValue?.published === newValue.published) {
      return oldValue
    }
    return newValue
  })

  return { posts, addPost, stats }
}
```

**TypeScript Types:**
- Groupés par domaine dans `types/`

```
types/
├── article.ts      # Article, ArticleMeta, Pillar, Category
├── search.ts       # SearchFilters, SearchResult
└── navigation.ts   # NavItem, Breadcrumb
```

## Components Organization

**Structure générale:**

```
app/components/
├── ui/                    # shadcn-vue (Reka UI primitives)
│   ├── button/
│   ├── card/
│   └── alert/
├── content/               # Composants MDC (directement ici, pas de sous-dossier prose/)
│   ├── Alert.vue          # → ::alert
│   ├── FeatureCard.vue    # → ::feature-card
│   ├── MdcAlert.vue       # → ::mdc-alert (wrapper shadcn)
│   ├── MdcCard.vue        # → ::mdc-card (wrapper shadcn)
│   ├── ProseA.vue         # Override liens Markdown
│   ├── ProseH2.vue        # Override h2 Markdown
│   ├── ProseH3.vue        # Override h3 Markdown
│   ├── ProsePre.vue       # Override code blocks Markdown
│   └── heavy/             # Composants lourds (lazy async)
│       ├── TableOfContents.vue
│       └── CodePlayground.vue
├── layout/                # Composants layout
│   ├── TheHeader.vue
│   ├── TheFooter.vue
│   └── LanguageSwitcher.vue
└── search/                # Composants recherche
    ├── SearchCommand.vue
    └── SearchFilters.vue
```

**Conventions de nommage composants MDC:**

| Fichier | Usage MDC | Type |
|---------|-----------|------|
| `Alert.vue` | `::alert` | Composant MDC natif |
| `FeatureCard.vue` | `::feature-card` | PascalCase → kebab-case |
| `MdcAlert.vue` | `::mdc-alert` | Wrapper shadcn-vue |
| `ProseA.vue` | Liens Markdown | Override élément natif |
| `ProsePre.vue` | Blocs code | Override élément natif |

**Préfixe `Prose` réservé:** Override des éléments HTML natifs du Markdown.

**Préfixe `Mdc` recommandé:** Wrappers shadcn-vue pour usage MDC.

**Configuration lazy loading composants lourds:**

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  components: [
    { path: '~/components/content/heavy', isAsync: true },
    '~/components',
  ],
})
```

**Usage avec hydratation différée:**

```vue
<LazyTableOfContents hydrate-on-visible />
<LazyCodePlayground hydrate-on-idle />
```

---

## Component Size Guidelines

**Seuil recommandé : ~200 lignes par composant SFC**

Ce seuil est une guideline de design, pas une règle stricte. Au-delà de 200 lignes, évaluez si le composant devrait être splitté.

**Quand splitter un composant :**

| Signal | Action recommandée |
|--------|-------------------|
| Script > 150 lignes | Extraire la logique vers un composable |
| Template > 100 lignes | Extraire des sections vers des composants enfants |
| 3+ domaines logiques distincts | Séparer par responsabilité |
| Tests difficiles à écrire | Isoler les parties testables |

**3 stratégies de splitting (par ordre de préférence) :**

1. **Extraire la logique vers un composable** — Réduit le script sans créer de nouveaux composants

```vue
<!-- Avant : logique inline -->
<script setup lang="ts">
const user = ref(null)
const isAuthenticated = computed(() => !!user.value)
const login = async () => { /* 50 lignes */ }
const logout = () => { /* 20 lignes */ }
// + 100 lignes de logique projet...
</script>

<!-- Après : composable extrait -->
<script setup lang="ts">
const { user, isAuthenticated, login, logout } = useAuth()
const { projects, loading, fetchProjects } = useProjects()
</script>
```

2. **Extraire des sections UI en composants enfants** — Quand une section du template est cohérente et indépendante

```vue
<!-- Avant : template monolithique 150 lignes -->
<template>
  <header><!-- 40 lignes --></header>
  <main><!-- 80 lignes --></main>
  <footer><!-- 30 lignes --></footer>
</template>

<!-- Après : composants enfants -->
<template>
  <ArticleHeader :article="article" />
  <ArticleContent :content="content" />
  <ArticleFooter :author="author" />
</template>
```

3. **Pattern Smart/Presentational** — Sépare la logique de la présentation

```
app/components/
├── article/
│   ├── ArticleContainer.vue   # Smart: fetch, state, logique
│   └── ArticleView.vue        # Presentational: props → UI
```

| Type | Responsabilité | Localisation typique |
|------|---------------|---------------------|
| **Smart (Container)** | Fetch data, gère state, logique métier | `pages/`, `*Container.vue` |
| **Presentational (Dumb)** | Props → UI, émet events, zéro side-effects | `components/`, `Base*.vue` |

---

## Anti-pattern Singleton Composables

**⚠️ Piège courant : state défini HORS de la fonction = singleton involontaire**

```typescript
// ❌ SINGLETON - state partagé entre TOUS les consommateurs
const globalCount = ref(0)  // Défini au niveau module

export function useCounter() {
  const increment = () => globalCount.value++
  return { count: globalCount, increment }
}

// ✅ INSTANCE - chaque consommateur a son propre state
export function useCounter() {
  const count = ref(0)  // Défini DANS la fonction
  const increment = () => count.value++
  return { count, increment }
}
```

**Quand c'est un bug :**
- Formulaires où chaque instance doit avoir ses propres valeurs
- Composants réutilisables avec état indépendant

**Quand c'est intentionnel :**
- State global partagé (préférer Pinia pour la clarté)
- Cache de données fetch

**Règle simple :** Si vous n'êtes pas sûr, définissez **toujours** le state à l'intérieur de la fonction.

---

## useState vs Module-Level ref (SSG/SSR)

**⚠️ Règle critique pour SSG/SSR :** Ne jamais utiliser `ref()` au niveau module pour du state partagé — cela cause une **pollution cross-request** et des erreurs d'hydratation.

```typescript
// ❌ DANGEREUX en SSG/SSR - state partagé entre toutes les requêtes
const locale = ref('en')  // Au niveau module

export function useLocale() {
  return { locale }
}

// ✅ SSR-SAFE - useState survit à l'hydratation
export function useLocale() {
  const locale = useState('locale', () => 'en')  // Clé unique requise
  return { locale }
}
```

**Différences clés :**

| Aspect | `ref()` module-level | `useState()` |
|--------|---------------------|--------------|
| **SSR** | ❌ Partagé entre requêtes | ✅ Isolé par requête |
| **Hydratation** | ❌ Mismatch possible | ✅ State survit |
| **DevTools** | Non intégré | Intégré Nuxt DevTools |
| **Cas d'usage** | Composable instance-local | State partagé SSR-safe |

**Quand utiliser useState :**
- Préférences utilisateur (locale, theme)
- Progress de lecture
- État de navigation
- Tout state qui doit survivre à l'hydratation

```typescript
// app/composables/useBlogState.ts
export const useCurrentLocale = () => useState<string>('locale', () => 'en')
export const useReadingProgress = () => useState<number>('reading-progress', () => 0)
export const useIsSidebarOpen = () => useState<boolean>('sidebar-open', () => false)
```

---

## Composables vs Pinia

**Règle de décision pour ce projet :** Composables uniquement. Pinia n'est pas utilisé car un blog SSG n'a pas besoin de global store.

**Règle générale (autres projets) :**

| Critère | Composable | Pinia |
|---------|-----------|-------|
| **Instances** | Chaque composant a son propre state | Singleton global partagé |
| **DevTools** | Non intégré | Intégré (time-travel, inspection) |
| **SSR Hydration** | Manuel | Automatique |
| **Persistance** | Manuelle | Plugins disponibles |
| **Testing** | Isolation facile | Mocking store requis |

**Choisir Composable quand :**
- Chaque composant a besoin de son propre state indépendant
- Logique encapsulée utilisée dans 2-3 endroits
- Pas besoin de DevTools pour debug

**Choisir Pinia quand :**
- State partagé entre pages non-parentes (ex: panier e-commerce)
- Besoin de DevTools pour debug temps-réel
- SSR avec hydration complexe
- Persistance localStorage requise

**Pattern hybride (recommandé en général) :**
- Pinia pour les stores globaux (auth, cart, preferences)
- Composables pour la logique réutilisable locale
