# Structure de dossiers Nuxt 4 pour blog SSG Cloudflare Pages

La nouvelle structure `app/` de Nuxt 4 représente un changement architectural majeur qui sépare clairement le code applicatif Vue du code serveur Nitro. Pour un blog multilingue SSG déployé sur Cloudflare Pages, cette organisation améliore significativement la performance des watchers (évite le scan de `node_modules/`), la type-safety IDE, et la clarté du projet. L'arborescence recommandée place `app/`, `server/`, `shared/`, `content/` et `public/` à la racine, chacun avec un rôle distinct.

## Arborescence complète recommandée

```
mon-blog/
├── app/                          # srcDir par défaut (code Vue)
│   ├── assets/
│   │   └── css/
│   │       └── app.css           # TailwindCSS 4 avec @theme
│   ├── components/
│   │   ├── ui/                   # shadcn-vue (atomiques)
│   │   │   ├── button/
│   │   │   │   ├── Button.vue
│   │   │   │   └── index.ts
│   │   │   └── card/
│   │   ├── blog/                 # Feature: Blog
│   │   │   ├── BlogCard.vue
│   │   │   └── BlogList.vue
│   │   └── layout/
│   │       ├── LayoutHeader.vue
│   │       └── LayoutFooter.vue
│   ├── composables/
│   │   ├── useAuth.ts            # Préfixe use* obligatoire
│   │   └── useBlog.ts
│   ├── layouts/
│   │   ├── default.vue
│   │   └── blog.vue
│   ├── middleware/
│   │   └── i18n.global.ts        # .global pour middleware global
│   ├── pages/
│   │   ├── index.vue
│   │   └── [locale]/
│   │       └── blog/
│   │           ├── index.vue
│   │           └── [...slug].vue
│   ├── plugins/
│   │   └── ssr-width.ts          # Requis pour shadcn-vue
│   ├── utils/
│   │   ├── cn.ts                 # Utilitaire shadcn class merge
│   │   └── formatDate.ts
│   ├── app.vue
│   └── app.config.ts
│
├── content/                       # Nuxt Content 3 (RACINE)
│   ├── fr/
│   │   └── blog/
│   │       ├── premier-article.md
│   │       └── deuxieme-article.md
│   └── en/
│       └── blog/
│           ├── first-post.md
│           └── second-post.md
│
├── server/                        # Nitro (RACINE)
│   ├── api/                       # Build-time uniquement en SSG
│   ├── routes/
│   │   ├── sitemap.xml.ts
│   │   └── robots.txt.ts
│   └── utils/
│
├── shared/                        # Code partagé app/server
│   ├── types/                     # Auto-importé ✓
│   │   └── post.ts
│   └── utils/                     # Auto-importé ✓
│       └── validation.ts
│
├── public/                        # Fichiers statiques (RACINE)
│   ├── images/
│   │   └── blog/
│   ├── favicon.ico
│   └── _headers                   # Headers Cloudflare
│
├── layers/                        # Layers locaux (optionnel, auto-scan)
│
├── nuxt.config.ts
├── content.config.ts              # Configuration Nuxt Content
├── tsconfig.json
├── package.json
└── .env
```

Cette structure sépare explicitement trois contextes d'exécution : **app/** pour le code Vue côté client/SSR, **server/** pour Nitro, et **shared/** pour le code isomorphe pur. Les dossiers `content/`, `public/`, `modules/` et `layers/` restent impérativement à la racine—les placer dans `app/` est une erreur courante à éviter.

## Configuration centrale `nuxt.config.ts`

La configuration Nuxt 4 doit explicitement déclarer la date de compatibilité et intégrer TailwindCSS 4 via le plugin Vite natif plutôt que le module `@nuxtjs/tailwindcss` :

```typescript
// nuxt.config.ts
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  compatibilityDate: '2024-12-01',
  future: { compatibilityVersion: 4 },
  
  css: ['~/assets/css/app.css'],
  
  vite: {
    plugins: [tailwindcss()]
  },
  
  modules: [
    '@nuxt/content',
    '@nuxtjs/i18n',
    'shadcn-nuxt'
  ],
  
  shadcn: {
    prefix: '',
    componentDir: '@/components/ui'
  },
  
  i18n: {
    locales: [
      { code: 'fr', language: 'fr-FR' },
      { code: 'en', language: 'en-US' }
    ],
    defaultLocale: 'fr',
    strategy: 'prefix_except_default'
  },
  
  nitro: {
    preset: 'cloudflare_pages',
    prerender: {
      crawlLinks: true,
      routes: ['/', '/sitemap.xml', '/robots.txt']
    }
  }
})
```

L'alias **@/** pointe désormais vers `app/` (et non la racine), tandis que **@@/** ou **~~/** pointent vers la racine du projet. L'alias **#shared** accède au dossier `shared/`. Ces conventions permettent des imports cohérents comme `import logo from '~/assets/images/logo.png'`.

## Configuration Nuxt Content 3 pour blog bilingue

Le fichier `content.config.ts` définit des collections séparées par langue avec validation Zod des frontmatter. La structure `content/{locale}/blog/` est préférée à `content/blog_{locale}/` car elle s'aligne avec les routes i18n :

```typescript
// content.config.ts
import { defineContentConfig, defineCollection } from '@nuxt/content'
import { z } from 'zod'

const blogSchema = z.object({
  title: z.string().min(1),
  description: z.string().max(160).optional(),
  date: z.date(),
  author: z.object({
    name: z.string(),
    avatar: z.string().optional()
  }).optional(),
  tags: z.array(z.string()).default([]),
  draft: z.boolean().default(false),
  image: z.object({
    src: z.string(),
    alt: z.string()
  }).optional(),
  translationKey: z.string().optional()  // Lier articles FR/EN
})

export default defineContentConfig({
  collections: {
    blog_fr: defineCollection({
      type: 'page',
      source: { include: 'fr/blog/**/*.md', prefix: '/fr/blog' },
      schema: blogSchema
    }),
    blog_en: defineCollection({
      type: 'page',
      source: { include: 'en/blog/**/*.md', prefix: '/en/blog' },
      schema: blogSchema
    })
  }
})
```

Les collections SQLite de Nuxt Content 3 fonctionnent automatiquement en SSG : au build, un dump WASM est généré pour les requêtes côté client. Les images de contenu doivent être placées dans `public/images/blog/` plutôt que dans `assets/`—les chemins relatifs depuis Markdown ne fonctionnent qu'avec le module optionnel `nuxt-content-assets`.

## Spécificités SSG sur Cloudflare Pages

**Point critique** : en mode SSG (`nuxt generate`), les routes `server/api/` ne sont **pas disponibles à l'exécution**. Les appels API s'exécutent uniquement au build-time, leurs résultats étant sauvegardés dans des payload files. Le dossier `server/` sert donc principalement à générer des fichiers statiques comme `sitemap.xml` ou `robots.txt`.

| Fonctionnalité | Disponibilité SSG |
|----------------|-------------------|
| Routes API build-time | ✅ |
| Pré-rendu via `useFetch` | ✅ |
| Routes API runtime | ❌ |
| Server middleware runtime | ❌ |
| Génération sitemap/robots | ✅ |

La configuration Cloudflare Pages ne nécessite aucun `wrangler.toml` pour un SSG pur. Dans le dashboard Cloudflare Pages, configurez :
- **Build command** : `pnpm run generate`
- **Output directory** : `.output/public`
- **Variable NODE_VERSION** : `22`

Créez un fichier `public/_headers` pour optimiser le caching :

```
/_nuxt/*
  Cache-Control: public, max-age=31536000, immutable

/*.html
  Cache-Control: public, max-age=3600
```

**Important** : désactivez Rocket Loader™, Mirage, et Email Obfuscation dans les paramètres Cloudflare pour éviter les erreurs d'hydratation Vue.

## Dossier `shared/` et architecture layers

Le dossier `shared/` (introduit en Nuxt 3.14) permet de partager du code entre `app/` et `server/`. **Seuls `shared/utils/` et `shared/types/` sont auto-importés**—les autres fichiers nécessitent un import explicite via `#shared/path`.

```typescript
// shared/types/post.ts - Auto-importé, accessible partout
export interface Post {
  id: number
  title: string
  slug: string
}

// shared/utils/formatDate.ts - Auto-importé
export const formatDate = (date: Date) => 
  new Intl.DateTimeFormat('fr-FR').format(date)
```

⚠️ Le code dans `shared/` ne peut **pas** importer de dépendances Vue ou Nitro spécifiques—uniquement du code TypeScript isomorphe pur.

Les **layers** (`layers/`) sont utiles pour l'extensibilité future ou les monorepos, mais inutiles pour un blog simple. Depuis Nuxt 3.12, les layers dans `~/layers/` sont automatiquement enregistrés. Chaque layer est une mini-application Nuxt complète avec son propre `nuxt.config.ts`, `app/`, `server/`, etc.

## TailwindCSS 4 avec configuration CSS-native

TailwindCSS v4 abandonne `tailwind.config.ts` au profit d'une configuration dans le CSS via `@theme`. Cette approche s'intègre naturellement avec les variables CSS et simplifie la configuration :

```css
/* app/assets/css/app.css */
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.72 0.11 178);
  --color-secondary: oklch(0.65 0.15 250);
  
  --font-sans: "Inter", system-ui, sans-serif;
  --font-display: "Satoshi", sans-serif;
  
  --breakpoint-3xl: 1920px;
}

/* Variables pour shadcn-vue */
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 0 0% 3.9%;
    --primary: 0 0% 9%;
  }
  
  .dark {
    --background: 0 0% 3.9%;
    --foreground: 0 0% 98%;
  }
}
```

Le module `@nuxtjs/tailwindcss` ne supporte pas encore pleinement TailwindCSS v4—utilisez directement `@tailwindcss/vite` pour les nouveaux projets.

## Conventions de nommage essentielles

| Type | Convention | Exemple |
|------|------------|---------|
| Composants Vue | PascalCase | `BlogCard.vue`, `LayoutHeader.vue` |
| Composables | camelCase + préfixe `use` | `useAuth.ts`, `useBlog.ts` |
| Utils | camelCase | `formatDate.ts`, `cn.ts` |
| Pages | kebab-case | `about-us.vue` |
| Middleware | kebab-case, `.global` pour global | `auth.ts`, `i18n.global.ts` |
| Plugins | kebab-case, `.client`/`.server` | `analytics.client.ts` |

Les composables dans des sous-dossiers de `composables/` ne sont **pas** auto-scannés—utilisez un re-export dans `composables/index.ts` ou configurez `imports.dirs` dans `nuxt.config.ts`.

## Anti-patterns à éviter absolument

- **Placer `server/`, `public/`, `content/` dans `app/`** — Ces dossiers doivent rester à la racine
- **Composables sans préfixe `use`** — Convention obligatoire pour l'auto-import
- **Code Vue/Nitro dans `shared/`** — Uniquement du code TypeScript isomorphe pur
- **`tailwind.config.ts` avec TailwindCSS v4** — Utilisez `@theme` dans le CSS
- **Routes API dynamiques en SSG** — Elles ne fonctionnent qu'au build-time
- **Modifier `tsconfig.json` directement** — Utilisez `alias` dans `nuxt.config.ts`
- **Ignorer la validation des frontmatter** — Définissez toujours des schémas Zod pour vos collections

## Conclusion

L'architecture Nuxt 4 avec `app/` comme srcDir offre une séparation claire des responsabilités qui facilite la maintenance et améliore les performances de développement. Pour un blog multilingue SSG sur Cloudflare Pages, privilégiez la simplicité : collections Content séparées par langue, TailwindCSS 4 en configuration CSS-native, et `shared/` uniquement pour les types et utilitaires partagés. Les layers peuvent être ajoutés plus tard pour l'extensibilité sans refactoring majeur. La clé du succès réside dans le respect strict des conventions de nommage et des emplacements de dossiers—les anti-patterns listés causent la majorité des problèmes de configuration rencontrés en production.