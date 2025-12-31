# i18n Patterns

Patterns d'internationalisation avec @nuxtjs/i18n v10.2.1+ et Nuxt Content 3.

## Configuration detectBrowserLanguage

Configuration recommandée pour la détection de langue navigateur :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  i18n: {
    locales: [
      { code: 'en', language: 'en-US', name: 'English' },
      { code: 'fr', language: 'fr-FR', name: 'Français' },
    ],
    defaultLocale: 'en',
    strategy: 'prefix_except_default',  // /about (en), /fr/about (fr)

    detectBrowserLanguage: {
      useCookie: true,
      cookieKey: 'i18n_locale',
      cookieSecure: true,       // HTTPS obligatoire (Cloudflare = HTTPS)
      cookieCrossOrigin: false,
      redirectOn: 'root',       // ⚠️ Comportement changé en v10 - CRITIQUE pour SEO SSG
      alwaysRedirect: false,
      fallbackLocale: 'en',
    }
  }
})
```

### Options redirectOn

| Valeur | Comportement | Usage |
|--------|--------------|-------|
| `'root'` | Redirige **uniquement** `/` vers `/fr/` | **Recommandé** - moins intrusif |
| `'all'` | Redirige toutes les pages sans préfixe | Migration depuis v9 |
| `'no prefix'` | Redirige pages sans préfixe de langue | Similaire à `'all'` |

⚠️ **Breaking change v9→v10** : `redirectOn: 'root'` ne redirige maintenant QUE la racine. Si vous aviez le comportement v9 (redirection sur toutes les pages), utilisez `redirectOn: 'all'`.

## Hook i18n:localeSwitched

Exécuter du code après un changement de langue :

```typescript
// plugins/i18n-watcher.client.ts
export default defineNuxtPlugin((nuxtApp) => {
  // ⚠️ Remplace onLanguageSwitched() de v9
  nuxtApp.hook('i18n:localeSwitched', ({ oldLocale, newLocale }) => {
    console.log(`Langue changée: ${oldLocale} → ${newLocale}`)

    // Exemples d'actions post-switch :
    // - Analytics tracking
    // - Mise à jour préférences utilisateur
    // - Refresh de données externes
  })
})
```

### Autres hooks i18n disponibles

| Hook | Déclencheur | Usage |
|------|-------------|-------|
| `i18n:beforeLocaleSwitch` | Avant le changement | Validation, confirmation |
| `i18n:localeSwitched` | Après le changement | Analytics, refresh données |

```typescript
// Exemple: Confirmation avant changement
nuxtApp.hook('i18n:beforeLocaleSwitch', async ({ oldLocale, newLocale, initialSetup }) => {
  if (initialSetup) return // Pas de confirmation au chargement initial

  // Retourner false pour annuler le changement
  const confirmed = await confirmLanguageChange()
  if (!confirmed) {
    return false
  }
})
```

## Language Switcher Component

### Pattern Toggle Accessible (WCAG 2.2 - Recommandé pour bilingue)

Pour un site **bilingue FR/EN**, le pattern toggle inline est recommandé : un seul clic, options toujours visibles, conforme aux conventions des sites gouvernementaux bilingues.

```vue
<!-- app/components/layout/LanguageSwitcher.vue -->
<script setup lang="ts">
const { locale } = useI18n()
const switchLocalePath = useSwitchLocalePath()
</script>

<template>
  <nav aria-label="Sélection de langue">
    <span
      v-if="locale === 'fr'"
      aria-current="page"
      lang="fr"
      class="font-semibold"
    >
      FR
    </span>
    <NuxtLink
      v-else
      :to="switchLocalePath('fr')"
      lang="fr"
      hreflang="fr"
    >
      FR
    </NuxtLink>

    <span aria-hidden="true" class="mx-2">|</span>

    <span
      v-if="locale === 'en'"
      aria-current="page"
      lang="en"
      class="font-semibold"
    >
      EN
    </span>
    <NuxtLink
      v-else
      :to="switchLocalePath('en')"
      lang="en"
      hreflang="en"
    >
      EN
    </NuxtLink>
  </nav>
</template>
```

**Attributs ARIA essentiels :**

| Attribut | Élément | Rôle |
|----------|---------|------|
| `aria-label` | `<nav>` | Identifie la navigation pour lecteurs d'écran |
| `aria-current="page"` | Langue active | Indique la sélection courante |
| `lang` | Chaque option | Prononciation correcte par lecteur d'écran |
| `hreflang` | Liens | SEO + indication langue cible |
| `aria-hidden="true"` | Séparateur `\|` | Masqué des lecteurs d'écran |

**⚠️ Éviter les drapeaux seuls** : Ils représentent des pays, pas des langues. Utiliser les codes ISO ou noms natifs ("Français", "English").

### Pattern ToggleGroup shadcn-vue (Alternative stylisée)

Utilise le composant ToggleGroup de Reka UI pour un rendu plus moderne :

```vue
<!-- app/components/layout/LanguageSwitcher.vue -->
<script setup lang="ts">
import { ToggleGroup, ToggleGroupItem } from '@/components/ui/toggle-group'

const { locale } = useI18n()
const switchLocalePath = useSwitchLocalePath()
</script>

<template>
  <ToggleGroup
    type="single"
    :model-value="locale"
    variant="outline"
    size="sm"
    class="border rounded-md"
  >
    <ToggleGroupItem
      value="fr"
      as-child
      aria-label="Français"
      class="px-3 data-[state=on]:bg-primary data-[state=on]:text-primary-foreground"
    >
      <NuxtLink :to="switchLocalePath('fr')" lang="fr">FR</NuxtLink>
    </ToggleGroupItem>
    <ToggleGroupItem
      value="en"
      as-child
      aria-label="English"
      class="px-3 data-[state=on]:bg-primary data-[state=on]:text-primary-foreground"
    >
      <NuxtLink :to="switchLocalePath('en')" lang="en">EN</NuxtLink>
    </ToggleGroupItem>
  </ToggleGroup>
</template>
```

**Positionnement recommandé** : Haut à droite du header, à côté des éléments utilitaires (recherche, thème).

### Pattern NuxtLink simple (liste de langues)

Pour plus de 2 langues ou un design minimaliste :

```vue
<template>
  <nav class="flex gap-2">
    <NuxtLink
      v-for="loc in locales"
      :key="loc.code"
      :to="switchLocalePath(loc.code)"
      :lang="loc.code"
      :hreflang="loc.code"
      :class="{ 'font-bold': locale === loc.code }"
    >
      {{ loc.code.toUpperCase() }}
    </NuxtLink>
  </nav>
</template>
```

## useLocaleHead() pour SEO

Injection automatique des meta tags hreflang :

```vue
<!-- app/layouts/default.vue -->
<script setup lang="ts">
const head = useLocaleHead({
  addSeoAttributes: true,
  addDirAttribute: true,
  seo: {
    // Préserve certains query params dans l'URL canonique (optionnel)
    canonicalQueries: ['ref', 'utm_source']
  }
})
</script>

<template>
  <Html :lang="head.htmlAttrs.lang" :dir="head.htmlAttrs.dir">
    <Head>
      <template v-for="link in head.link" :key="link.hid">
        <Link
          :rel="link.rel"
          :href="link.href"
          :hreflang="link.hreflang"
        />
      </template>
      <template v-for="meta in head.meta" :key="meta.hid">
        <Meta :property="meta.property" :content="meta.content" />
      </template>
    </Head>
    <Body>
      <slot />
    </Body>
  </Html>
</template>
```

**Génère automatiquement :**
- `<link rel="alternate" hreflang="en" href="https://sebc.dev/blog/article" />`
- `<link rel="alternate" hreflang="fr" href="https://sebc.dev/fr/blog/article" />`
- `<link rel="alternate" hreflang="x-default" href="https://sebc.dev/blog/article" />`
- `<meta property="og:locale" content="en_US" />`
- `<meta property="og:locale:alternate" content="fr_FR" />`

## Routes Dynamiques avec useSetI18nParams()

Pour les routes dynamiques (`[slug]`, `[id]`), le composable `useSetI18nParams()` permet de traduire les paramètres de route entre langues :

```vue
<!-- app/pages/blog/[slug].vue -->
<script setup lang="ts">
const route = useRoute()
const setI18nParams = useSetI18nParams()

// Récupérer l'article avec ses slugs traduits
const { data: article } = await useAsyncData(
  `article-${route.params.slug}`,
  () => queryCollection('articles_fr').path(`/blog/${route.params.slug}`).first()
)

// Définir les slugs traduits pour chaque locale
// Permet au language switcher de pointer vers le bon slug
if (article.value?.slugs) {
  setI18nParams({
    fr: { slug: article.value.slugs.fr },  // 'mon-article'
    en: { slug: article.value.slugs.en }   // 'my-article'
  })
}
</script>
```

**Cas d'usage :**

| Scénario | Sans `useSetI18nParams()` | Avec `useSetI18nParams()` |
|----------|---------------------------|---------------------------|
| FR → EN sur `/blog/mon-article` | `/en/blog/mon-article` (404) | `/en/blog/my-article` ✅ |

**Frontmatter pour slugs traduits :**

```yaml
# content/fr/blog/mon-article.md
---
title: "Mon Article"
slugs:
  fr: "mon-article"
  en: "my-article"
---
```

## Routes Personnalisées avec definePageMeta()

**⚠️ `defineI18nRoute()` est DÉPRÉCIÉ** en v10 et sera supprimé en v11. Utiliser `definePageMeta()` avec la propriété `i18n` :

### Activation (nuxt.config.ts)

```typescript
export default defineNuxtConfig({
  experimental: {
    scanPageMeta: true  // Requis pour les routes nommées personnalisées
  },
  i18n: {
    customRoutes: 'meta'  // Active l'approche definePageMeta
  }
})
```

### Définition par page

```vue
<!-- app/pages/about.vue -->
<script setup>
definePageMeta({
  i18n: {
    paths: {
      fr: '/a-propos',
      en: '/about'
    }
  }
})
</script>
```

### Alternative : Configuration centralisée

Pour une gestion centralisée des routes traduites :

```typescript
// nuxt.config.ts
i18n: {
  customRoutes: 'config',
  pages: {
    'about': { fr: '/a-propos', en: '/about' },
    'blog-[slug]': { fr: '/blog/[slug]', en: '/blog/[slug]' },
    'categories-[category]': { fr: '/categories/[category]', en: '/categories/[category]' }
  }
}
```

| Approche | Avantages | Inconvénients |
|----------|-----------|---------------|
| `customRoutes: 'meta'` | Co-localisé avec la page, visible dans le fichier | Dispersé dans le code |
| `customRoutes: 'config'` | Vue d'ensemble centralisée | Maintenance séparée |

**Recommandation** : `meta` pour projets avec peu de routes traduites, `config` pour projets complexes.

## Lazy Loading Translations

En v10, le lazy loading est **activé par défaut** — ne configurez plus `lazy: true`. Le `langDir` est résolu relativement à `restructureDir` (défaut: `'i18n'`), lui-même relatif au `rootDir` du projet (pas `srcDir`).

**⚠️ Structure CRITIQUE pour Nuxt 4 avec `app/` comme srcDir :**

```
projet/
├── app/                    # srcDir
│   ├── pages/
│   ├── components/
│   └── ...
├── i18n/                   # À la RACINE, PAS dans app/
│   ├── locales/
│   │   ├── en.json
│   │   └── fr.json
│   └── i18n.config.ts      # Config Vue I18n (auto-détecté)
├── content/
│   ├── en/
│   └── fr/
└── nuxt.config.ts
```

Les fichiers doivent rester **au niveau racine** car ils sont utilisés côté client ET serveur. Le placement dans `app/` causerait des problèmes de résolution.

**Configuration nuxt.config.ts :**

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  i18n: {
    // langDir résolu: <rootDir>/i18n/locales/ (défaut v10)
    locales: [
      { code: 'en', language: 'en-US', name: 'English', file: 'en.json' },
      { code: 'fr', language: 'fr-FR', name: 'Français', file: 'fr.json' },
    ],
  }
})
```

**Format JSON pour les traductions** (recommandé pour SSG - plus léger et statiquement analysable) :

```json
// i18n/locales/en.json
{
  "nav": {
    "home": "Home",
    "blog": "Blog",
    "about": "About"
  },
  "blog": {
    "title": "Articles",
    "readMore": "Read more"
  },
  "seo": {
    "defaultTitle": "My Blog",
    "defaultDescription": "A bilingual blog about..."
  }
}
```

## Fichier i18n.config.ts (optionnel)

Configuration Vue I18n runtime pour personnaliser le comportement global :

```typescript
// i18n/i18n.config.ts
export default defineI18nConfig(() => ({
  legacy: false,           // Utilise Composition API uniquement
  fallbackLocale: 'en',    // Langue de repli
  missingWarn: false,      // Désactive warnings console (prod)
  fallbackWarn: false      // Désactive warnings fallback (prod)
}))
```

Ce fichier est auto-détecté par @nuxtjs/i18n v10 s'il existe dans le dossier `i18n/`.

## Mode strictSeo expérimental (v10+)

Le nouveau mode `strictSeo` automatise **complètement** la gestion des tags SEO i18n. Quand activé, `useLocaleHead()` devient **inutile** (et lance une erreur si appelé) :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  i18n: {
    experimental: {
      strictSeo: true,
      // ou avec options :
      strictSeo: { canonicalQueryParams: ['ref'] }
    }
  }
})
```

**Avantages du mode strictSeo :**

| Fonctionnalité | Comportement |
|----------------|--------------|
| Tags hreflang | Générés automatiquement sans `useLocaleHead()` |
| Locales non supportées | Pas de tags alternatifs générés sur routes dynamiques |
| `<SwitchLocalePathLink>` | Désactive automatiquement les liens vers locales indisponibles |
| Migration | Deviendra probablement le défaut en v11 |

**Note :** Ce mode est encore expérimental. Pour la production actuelle, continuez à utiliser `useLocaleHead()` dans les layouts.

## Limitations SSG + Cloudflare Pages

La détection de langue présente des **limitations importantes en SSG** sur hébergement statique :

| Mécanisme | Disponibilité | Notes |
|-----------|---------------|-------|
| `Accept-Language` header | ❌ Non disponible | Serveur statique = pas d'accès aux headers HTTP |
| `navigator.language` | ✅ Client uniquement | Disponible après hydratation |
| Cookies | ✅ Client uniquement | Cloudflare ne lit pas les cookies pour fichiers statiques |

**Comportement en SSG :**

```
Premier accès sur /
       ↓
HTML statique servi (pas de détection serveur)
       ↓
Hydratation client
       ↓
navigator.language détecté → redirection si nécessaire
       ↓
Cookie posé pour visites futures
```

**Pourquoi `redirectOn: 'root'` est CRITIQUE pour SEO :**

| redirectOn | Comportement Google Crawler | Impact SEO |
|------------|----------------------------|------------|
| `'root'` | Seule `/` redirige | ✅ `/fr/article` indexé correctement |
| `'all'` | Toutes pages redirigent | ❌ Crawler (sans Accept-Language) redirigé partout |

**Recommandation SSG :** Pour les sites où le SEO prime sur l'UX de détection automatique, considérez `detectBrowserLanguage: false` et laissez les utilisateurs choisir via un sélecteur de langue explicite.

## Pattern Content + i18n avec Fallback

Page dynamique pour contenu multilingue avec fallback vers la langue par défaut :

```vue
<!-- app/pages/blog/[...slug].vue -->
<script setup lang="ts">
import { withLeadingSlash, joinURL } from 'ufo'
import type { Collections } from '@nuxt/content'

const { locale, t } = useI18n()
const route = useRoute()

// Construire le chemin du contenu
const contentPath = computed(() =>
  withLeadingSlash(joinURL('blog', ...(
    Array.isArray(route.params.slug)
      ? route.params.slug
      : [route.params.slug]
  )))
)

// Query avec collection basée sur la locale
const { data: article } = await useAsyncData(
  `blog-${contentPath.value}-${locale.value}`,
  async () => {
    const collection = `articles_${locale.value}` as keyof Collections
    let content = await queryCollection(collection)
      .path(contentPath.value)
      .first()

    // Fallback vers anglais si contenu manquant
    if (!content && locale.value !== 'en') {
      content = await queryCollection('articles_en')
        .path(contentPath.value)
        .first()
      if (content) content._isFallback = true
    }

    return content
  },
  { watch: [locale] }  // ⚠️ OBLIGATOIRE pour re-fetch au changement de langue
)

// SEO avec hreflang automatique
const head = useLocaleHead({ addSeoAttributes: true })
useHead(() => ({
  htmlAttrs: { lang: head.value.htmlAttrs?.lang },
  link: [...(head.value.link || [])],
  meta: [...(head.value.meta || [])],
  title: article.value?.title || t('blog.title')
}))
</script>

<template>
  <article v-if="article">
    <!-- Notice si contenu non disponible dans la langue demandée -->
    <div
      v-if="article._isFallback"
      class="rounded-md bg-muted p-4 text-sm text-muted-foreground mb-6"
    >
      {{ t('content.notAvailableInLanguage') }}
    </div>

    <h1>{{ article.title }}</h1>
    <time>{{ new Date(article.date).toLocaleDateString(locale) }}</time>
    <ContentRenderer :value="article" />
  </article>
</template>
```

**Points clés du pattern :**

| Aspect | Implementation | Raison |
|--------|----------------|--------|
| `watch: [locale]` | Dans `useAsyncData` | Re-fetch automatique au changement de langue |
| `_isFallback` flag | Ajouté dynamiquement | Permet d'afficher une notice utilisateur |
| Collection dynamique | `articles_${locale.value}` | Évite les if/else multiples |
| Clé cache unique | `blog-${path}-${locale}` | Évite les collisions de cache |

**Traduction pour la notice fallback :**

```json
// i18n/locales/fr.json
{
  "content": {
    "notAvailableInLanguage": "Cet article n'est pas encore disponible en français. Vous consultez la version anglaise."
  }
}
```

## Résumé migration v9 → v10

| Aspect | v9 | v10 |
|--------|----|----|
| Lazy loading | `lazy: true` optionnel | Toujours lazy (défaut) |
| `redirectOn: 'root'` | Redirige toutes pages sans préfixe | Redirige **uniquement** `/` |
| Hook changement langue | `onLanguageSwitched()` | `i18n:localeSwitched` hook |
| Vue I18n | v9 | v11 |
| Nitro language detection | Non disponible | Ajouté |
| Routes personnalisées | `defineI18nRoute()` | `definePageMeta({ i18n: {} })` |
| Pluralisation | `$tc(key, count)` | `$t(key, count)` ou `$t(key, { n: count })` |
| Champ locale | `iso` | `language` (requis pour hreflang) |

### Migration $tc() → $t() (Vue I18n v11)

**`$tc()` est supprimé** dans Vue I18n v11. La pluralisation se fait maintenant avec `$t()` :

```typescript
// ❌ ANCIEN (Vue I18n v9)
$tc('items', 5)
$tc('items', 5, { count: 5 })

// ✅ NOUVEAU (Vue I18n v11)
$t('items', 5)
$t('items', { n: 5 })
$t('items', { count: 5 }, 5)  // Avec contexte additionnel
```

**Format des messages de pluralisation :**

```json
{
  "items": "no item | one item | {n} items"
}
```

Les séparateurs `|` définissent les formes : zéro, singulier, pluriel.

## Anti-patterns

| ❌ Anti-pattern | ✅ Pattern correct |
|-----------------|-------------------|
| Hardcoder la locale dans les composants | Utiliser `useI18n()` |
| Ignorer `watch: [locale]` dans useAsyncData | Toujours inclure pour content |
| Utiliser `onLanguageSwitched()` (v9) | Utiliser hook `i18n:localeSwitched` |
| Oublier `language` dans locales config | Requis pour hreflang auto |
| `setLocale('fr')` seul | `navigateTo(switchLocalePath('fr'))` |
| Canonical cross-langue | Canonical auto-référencé par langue |
| `defineI18nRoute()` (v9 déprécié) | `definePageMeta({ i18n: {} })` |
| `$tc()` pour pluralisation | `$t(key, count)` (Vue I18n v11) |
| Language switcher sans attributs `lang` | Ajouter `lang` et `hreflang` sur chaque option |
| Drapeaux seuls comme indicateurs | Codes ISO ou noms natifs ("Français") |

### setLocale() vs useSwitchLocalePath()

`setLocale()` change la locale **sans changer l'URL** — cela crée une incohérence entre l'état i18n et l'URL affichée.

```typescript
// ❌ MAUVAIS : Change la locale mais pas l'URL
const { setLocale } = useI18n()
await setLocale('fr')
// URL reste /about au lieu de /fr/about

// ✅ BON : Change la locale ET l'URL
const switchLocalePath = useSwitchLocalePath()
await navigateTo(switchLocalePath('fr'))
// URL devient /fr/about, locale synchronisée
```

**Cas d'usage de `setLocale()`** : Uniquement pour des changements temporaires sans navigation (ex: preview d'une traduction).

### Clé useAsyncData et fragmentation du cache

La clé de `useAsyncData` affecte le comportement du cache. Pour le contenu multilingue :

```typescript
// ⚠️ Fragmente le cache (2 entrées séparées pour le même article)
const { data } = await useAsyncData(
  `blog-${slug.value}-${locale.value}`,  // Clé unique par locale
  () => queryCollection(...).path(slug.value).first(),
  { watch: [locale] }
)

// ✅ Optimisé : clé indépendante de la locale (watch gère le refetch)
const { data } = await useAsyncData(
  `blog-${slug.value}`,  // Clé basée sur le path uniquement
  () => queryCollection(`articles_${locale.value}`).path(slug.value).first(),
  { watch: [locale] }
)
```

| Stratégie clé | Comportement cache | Usage |
|---------------|-------------------|-------|
| Avec locale | Cache séparé par langue | Quand les données EN et FR doivent coexister |
| Sans locale | Cache partagé, refetch au switch | **Recommandé** pour économiser mémoire |

**Recommandation** : Utiliser des clés sans locale quand `watch: [locale]` est présent. Le refetch remplace les données correctement.
