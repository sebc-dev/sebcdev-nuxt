# Bonnes pratiques MDC pour Nuxt Content 3.10+ avec Nuxt 4

Nuxt Content 3.x représente une refonte complète du module avec une architecture basée sur **SQLite**, nécessitant obligatoirement un fichier `content.config.ts` pour définir les collections. La syntaxe MDC reste largement compatible avec la version 2, mais les composables et composants ont été significativement modifiés. Pour un stack technique comprenant Nuxt 4.2.x, TailwindCSS 4.1.x et un déploiement SSG sur Cloudflare Pages, l'optimisation passe par une configuration fine de Nitro, le lazy loading des composants lourds, et la directive `@source` pour scanner les fichiers Markdown avec Tailwind.

## Syntaxe MDC : inline, block et nesting

La syntaxe MDC distingue deux formes principales d'utilisation des composants. Les **composants inline** utilisent la syntaxe `:component[contenu]{props}` pour les éléments simples sans slots comme les badges ou icônes. Les **composants block** avec `::component{props}` permettent d'encapsuler du contenu Markdown complet. Un piège courant survient lorsqu'un composant inline est suivi de `-`, `_` ou `:` — il faut alors ajouter un spécificateur vide : `:hello{}-world`.

Pour les props simples, la syntaxe inline `{key="value" .class #id}` suffit. Les configurations complexes privilégient le bloc YAML avec triple tirets :

```markdown
::icon-card
---
icon: IconNuxt
title: Architecture Nuxt
description: Exploitez toute la puissance de Nuxt
---
::
```

Le binding dynamique depuis le frontmatter utilise le préfixe `:` pour déclencher l'évaluation : `::alert{:type="type"}` où `type` provient des métadonnées du document. Pour les tableaux et objets JSON, encadrez avec des guillemets simples externes et doubles internes : `:items='["Nuxt", "Vue", "React"]'`.

L'imbrication de composants repose sur l'ajout de colons supplémentaires (`::`→`:::`→`::::`). La **profondeur recommandée est de 3-4 niveaux maximum** ; au-delà, restructurez en sous-composants distincts. Les composants async imbriqués nécessitent un wrapper `<Suspense suspensible>` avec l'option `immediate: true` sur `useAsyncData`.

## Named slots et unwrapping du contenu

Les slots nommés s'utilisent avec la syntaxe `#slotName` dans le contenu MDC :

```markdown
::hero{image="/images/bg.jpg"}
#title
Bienvenue sur notre site

#description
Une description avec du **Markdown** formaté.

Contenu du slot par défaut ici.
::
```

Le composant Vue correspondant doit utiliser l'attribut `mdc-unwrap` pour supprimer les balises `<p>` automatiquement générées par le parser Markdown. C'est essentiel pour les headings où un `<p>` à l'intérieur d'un `<h1>` serait invalide :

```vue
<h1 v-if="$slots.title" class="hero__title">
  <slot name="title" mdc-unwrap="p" />
</h1>
```

La vérification conditionnelle `$slots.slotName` permet d'afficher les éléments uniquement si le slot est fourni. Les conventions de nommage recommandées sont `#title`, `#description`, `#content`, `#footer` et `#default` pour le slot principal.

## Création de composants MDC personnalisés

### Structure de fichiers avec Nuxt 4

La nouvelle structure `app/` de Nuxt 4 place les composants MDC dans `app/components/content/`, tandis que le dossier `content/` **reste à la racine du projet** :

```
projet/
├── app/
│   └── components/
│       └── content/       # Composants MDC auto-globaux
│           ├── Alert.vue
│           ├── Card.vue
│           └── ProseA.vue # Override Prose
├── content/               # À la racine, pas dans app/
│   └── blog/
├── content.config.ts      # Obligatoire en v3
└── nuxt.config.ts
```

Le mapping de nommage transforme automatiquement **PascalCase en kebab-case** : `FeatureCard.vue` devient `::feature-card`. Le préfixe `Prose` est réservé aux overrides des éléments HTML natifs du Markdown (ProseA, ProseH1, ProseP, ProsePre, etc.).

### Pattern TypeScript recommandé

```vue
<script setup lang="ts">
interface AlertProps {
  type?: 'info' | 'warning' | 'error' | 'success'
  title?: string
}

const props = withDefaults(defineProps<AlertProps>(), {
  type: 'info',
  title: ''
})
</script>

<template>
  <div class="alert" :class="`alert-${props.type}`">
    <h4 v-if="props.title">{{ props.title }}</h4>
    <div class="alert-content">
      <slot mdc-unwrap="p" />
    </div>
  </div>
</template>
```

L'utilisation de `defineProps<Interface>()` avec génériques TypeScript et `withDefaults()` assure un typage strict end-to-end. Le composant `<ContentSlot>` de la v2 est **supprimé** — utilisez directement le slot Vue natif avec `mdc-unwrap`.

## Optimisations SSG pour Cloudflare Pages

### Configuration Nitro optimale

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true,
  modules: ['@nuxt/content'],
  
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      crawlLinks: true,
      routes: ['/'],
      autoSubfolderIndex: false,
      concurrency: 4,
    },
    compressPublicAssets: {
      gzip: true,
      brotli: true,
    },
  },
  
  routeRules: {
    '/': { prerender: true },
    '/blog/**': { prerender: true },
    '/docs/**': { prerender: true },
  },
  
  content: {
    build: {
      markdown: {
        highlight: {
          theme: 'github-dark',
          langs: ['typescript', 'vue', 'bash', 'json'],
        },
      },
    },
  },
  
  experimental: {
    payloadExtraction: true,
  },
})
```

Le preset `cloudflare_pages` est détecté automatiquement via Git integration. L'option `autoSubfolderIndex: false` évite les conflits avec le routage Cloudflare. La compression Brotli réduit significativement la taille des payloads.

### Lazy loading et hydratation différée

Pour les composants lourds, configurez un répertoire dédié avec chargement async automatique :

```typescript
components: [
  { path: '~/components/content/heavy', isAsync: true },
  '~/components',
],
```

Nuxt 4 introduit les directives d'hydratation différée directement sur les composants :

```vue
<LazyCodeBlock hydrate-on-visible :code="example" />
<LazyTableOfContents hydrate-on-idle />
<LazyCommentSection hydrate-on-interaction="click" />
<LazyStaticBanner hydrate-never />
```

L'option `hydrate-never` convient aux contenus purement statiques qui n'ont jamais besoin d'interactivité côté client.

### Headers de cache Cloudflare

Créez le fichier `public/_headers` :

```
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

/*
  Cache-Control: public, max-age=0, must-revalidate
  X-Content-Type-Options: nosniff

/api/_content/*
  Cache-Control: public, max-age=3600, stale-while-revalidate=86400
```

Désactivez dans le dashboard Cloudflare : **Rocket Loader™**, **Mirage** et **Email Address Obfuscation** qui peuvent interférer avec le JavaScript hydraté.

## Intégration TailwindCSS 4.1.x et shadcn-vue

### Configuration CSS-first avec @source

TailwindCSS 4.x abandonne la configuration JavaScript au profit d'une approche CSS-first. Le point critique pour utiliser les classes dans le contenu MDC est la directive `@source` :

```css
/* app/assets/css/main.css */
@import "tailwindcss";

/* Scanner explicitement les fichiers Markdown */
@source "../../../content/**/*";

@theme {
  --font-sans: 'Public Sans', system-ui, sans-serif;
  --color-primary: #1E40AF;
}
```

```typescript
// nuxt.config.ts
import tailwindcss from "@tailwindcss/vite";

export default defineNuxtConfig({
  css: ['./app/assets/css/main.css'],
  vite: {
    plugins: [tailwindcss()],
  },
})
```

Les classes s'utilisent ensuite directement dans le Markdown via la syntaxe d'attributs : `[texte]{.bg-blue-200 .rounded-lg .px-2}` ou sur les blocs `::div{.w-full .p-4}`.

### Wrapper pattern pour shadcn-vue

Les composants shadcn-vue doivent être wrappés dans `components/content/` pour l'usage MDC :

```vue
<!-- app/components/content/MdcAlert.vue -->
<script setup lang="ts">
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'

interface Props {
  variant?: 'default' | 'destructive'
  title?: string
}

withDefaults(defineProps<Props>(), {
  variant: 'default'
})
</script>

<template>
  <Alert :variant="variant">
    <AlertTitle v-if="title">{{ title }}</AlertTitle>
    <AlertDescription>
      <slot mdc-unwrap="p" />
    </AlertDescription>
  </Alert>
</template>
```

Utilisation : `::mdc-alert{title="Note" variant="destructive"} Contenu Markdown ::`. Les primitives Reka UI (anciennement Radix Vue) suivent le même pattern avec l'avantage d'une accessibilité WAI-ARIA complète intégrée.

## Internationalisation du contenu MDC

### Structure par langue recommandée

```
content/
├── en/
│   ├── index.md
│   └── docs/
├── fr/
│   ├── index.md
│   └── docs/
```

### Collections séparées par locale

```typescript
// content.config.ts
import { defineCollection, defineContentConfig } from '@nuxt/content'
import { z } from 'zod'

export default defineContentConfig({
  collections: {
    content_en: defineCollection({
      type: 'page',
      source: { include: 'en/**', prefix: '' },
      schema: z.object({ title: z.string() }),
    }),
    content_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/**', prefix: '' },
      schema: z.object({ title: z.string() }),
    }),
  },
})
```

La page catch-all gère le fallback vers la langue par défaut :

```vue
<script setup lang="ts">
const { locale } = useI18n()
const { data: page } = await useAsyncData(`page-${locale.value}`, async () => {
  const collection = `content_${locale.value}` as keyof Collections
  let content = await queryCollection(collection).path(slug.value).first()
  
  // Fallback anglais si non trouvé
  if (!content && locale.value !== 'en') {
    content = await queryCollection('content_en').path(slug.value).first()
  }
  return content
}, { watch: [locale] })
</script>
```

## Migration Content 2.x vers 3.x : changements critiques

### Composables renommés

| v2 | v3 |
|---|---|
| `queryContent()` | `queryCollection('collection')` |
| `fetchContentNavigation()` | `queryCollectionNavigation()` |
| `queryContent().findSurround()` | `queryCollectionItemSurroundings()` |
| `useContent()` | **Supprimé** — requêtes explicites |

### Composants supprimés

Les composants `<ContentDoc>`, `<ContentList>`, `<ContentNavigation>`, `<ContentQuery>` et `<ContentSlot>` sont **entièrement supprimés**. Remplacez-les par `<ContentRenderer :value="page" />` et des requêtes avec `queryCollection()`.

Le mode **Document Driven est supprimé** — créez manuellement une page `[...slug].vue` qui effectue les requêtes.

### Dépréciations à éviter absolument

```typescript
// ❌ Déprécié
import { z } from '@nuxt/content'
import { queryCollection } from '@nuxt/content/nitro'

// ✅ Correct
import { z } from 'zod'
import { queryCollection } from '@nuxt/content/server'
```

Les fichiers `_dir.yml` deviennent `.navigation.yml`. Les champs préfixés `_` perdent leur underscore (`_path` → `path`).

### Prose components consolidés

Les composants `ProseCode`, `ProseCodeInline` et `ProsePre` sont réduits à deux : **`ProseCode`** pour le code inline et **`ProsePre`** pour les blocs. Migrez la logique de highlight depuis `ProseCode` vers `ProsePre`.

## Conclusion

L'adoption de Nuxt Content 3.10+ avec Nuxt 4 nécessite une refonte de l'architecture vers le système de collections SQL, mais la syntaxe MDC dans les fichiers Markdown reste stable. Les gains de performance sont significatifs grâce à SQLite, particulièrement pour les sites volumineux. L'intégration avec TailwindCSS 4 requiert la directive `@source` pour scanner le contenu, tandis que shadcn-vue fonctionne via des composants wrapper dans `components/content/`. Pour le SSG sur Cloudflare Pages, le preset Nitro dédié, la compression Brotli et les headers de cache appropriés garantissent des temps de chargement optimaux avec un First Contentful Paint typiquement entre **1.0 et 1.5 seconde** après optimisation.