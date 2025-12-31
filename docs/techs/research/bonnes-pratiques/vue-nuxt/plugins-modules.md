# Plugins et modules Nuxt 4.2.X pour Cloudflare Pages

Nuxt 4.2.X introduit une structure de projet repensée avec le répertoire `app/` et des améliorations TypeScript significatives, tout en conservant la compatibilité avec les patterns établis pour les plugins et modules. Pour un blog hébergé sur Cloudflare Pages, **les quatre piliers techniques essentiels** sont : les suffixes contextuels pour optimiser les bundles, le chargement parallèle pour réduire le temps de démarrage jusqu'à 70%, les modules custom via `@nuxt/kit` pour encapsuler la logique métier, et une configuration runtime sécurisée adaptée à l'edge runtime. Ces pratiques, combinées à la génération de code avec Claude Code, permettent de créer des patterns reproductibles et maintenables.

---

## Les suffixes .client.ts et .server.ts déterminent le contexte d'exécution

Les suffixes contextuels constituent le mécanisme fondamental de Nuxt pour contrôler où s'exécute votre code. Un plugin avec le suffixe `.client.ts` est **exclu du bundle serveur** et ne s'exécute qu'après l'hydratation dans le navigateur. Inversement, `.server.ts` est exclu du bundle client. Cette séparation impacte directement la taille des bundles et la sécurité.

**Cas d'usage optimaux pour chaque suffixe** :

| Suffixe | Utilisation | Exemples concrets |
|---------|-------------|-------------------|
| `.client.ts` | APIs navigateur, DOM, localStorage | Analytics, animations, directives focus |
| `.server.ts` | Secrets, APIs Node.js, initialisation SSR | Tokens API, configuration base de données |
| Sans suffixe | Code universel isomorphique | Helpers, composables génériques |

Sur Cloudflare Pages (edge runtime), les plugins serveur présentent une contrainte majeure : **les APIs Node.js natives ne sont pas disponibles** sans activer le flag `nodejs_compat`. Les modules comme `fs`, `net` ou `child_process` échoueront silencieusement. Pour contourner cette limitation, ajoutez `compatibility_flags = ["nodejs_compat"]` dans votre configuration Cloudflare.

```typescript
// app/plugins/analytics.client.ts
// Plugin client-only pour Google Analytics
import VueGtag from 'vue-gtag-next'

export default defineNuxtPlugin((nuxtApp) => {
  const config = useRuntimeConfig()
  nuxtApp.vueApp.use(VueGtag, {
    property: { id: config.public.gaTrackingId }
  })
})
```

```typescript
// app/plugins/cloudflare-bindings.server.ts
// Accès aux bindings Cloudflare (D1, KV) côté serveur uniquement
export default defineNuxtPlugin(() => {
  const nuxtApp = useNuxtApp()
  
  nuxtApp.hook('app:created', () => {
    // Les bindings sont disponibles via event.context.cloudflare
    console.log('Server plugin initialized for Cloudflare edge')
  })
})
```

Un piège fréquent avec Claude Code : générer du code qui utilise `window` ou `document` dans un plugin sans suffixe. **Toujours vérifier** que les APIs browser sont dans des fichiers `.client.ts` ou protégées par `if (import.meta.client)`.

---

## Le chargement parallèle accélère drastiquement le démarrage

Par défaut, Nuxt exécute les plugins de manière séquentielle (waterfall). Avec **3 plugins async de 100ms chacun**, le temps total est de 300ms. En activant `parallel: true`, ces plugins s'exécutent simultanément et le temps tombe à environ **100ms** — une réduction de 67%.

La configuration utilise la syntaxe objet de `defineNuxtPlugin()` :

```typescript
// app/plugins/heavy-init.client.ts
export default defineNuxtPlugin({
  name: 'heavy-init',           // Requis pour dependsOn
  parallel: true,               // Active le chargement concurrent
  async setup(nuxtApp) {
    const heavyLib = await import('heavy-library')
    return {
      provide: { heavyLib: heavyLib.default }
    }
  }
})
```

**Quand utiliser parallel: true** : plugins indépendants, analytics, monitoring, connexions API tierces, intégrations Vue autonomes (dayjs, vue-gtag). **Quand rester séquentiel** : plugins qui modifient `nuxtApp` globalement, Pinia store initialization, plugins interdépendants.

Pour les dépendances entre plugins parallèles, utilisez `dependsOn` :

```typescript
// app/plugins/02.consumer.ts
export default defineNuxtPlugin({
  name: 'data-consumer',
  dependsOn: ['data-provider'],  // Attend que data-provider soit terminé
  parallel: true,                // Mais reste parallèle aux autres plugins
  async setup(nuxtApp) {
    const { $dataProvider } = nuxtApp
    // Accès garanti au provider
  }
})
```

Les propriétés `name`, `parallel`, `enforce` et `dependsOn` doivent être **statiques**. Définir dynamiquement ces valeurs (ex: `parallel: import.meta.server ? true : false`) empêche les optimisations de build de Nuxt.

---

## La création de modules custom avec @nuxt/kit structure votre code

Nuxt 4.x encourage l'encapsulation de la logique métier dans des modules locaux. La structure recommandée pour un module dans votre projet :

```
my-nuxt-app/
├── app/                    # Code applicatif Nuxt 4
├── modules/                # Modules locaux
│   └── blog-seo/
│       ├── index.ts        # defineNuxtModule()
│       └── runtime/
│           ├── components/
│           ├── composables/
│           └── plugin.ts
├── server/
└── nuxt.config.ts
```

L'API `defineNuxtModule()` avec la méthode `.with()` offre un typage TypeScript optimal pour les options résolues :

```typescript
// modules/blog-seo/index.ts
import { 
  defineNuxtModule, 
  createResolver, 
  addPlugin, 
  addImports 
} from '@nuxt/kit'

export interface ModuleOptions {
  siteUrl: string
  siteName: string
  defaultOgImage?: string
}

export default defineNuxtModule<ModuleOptions>().with({
  meta: {
    name: 'blog-seo',
    configKey: 'blogSeo',
    compatibility: { nuxt: '>=4.0.0' }
  },
  defaults: {
    defaultOgImage: '/og-default.png'
  },
  setup(options, nuxt) {
    const { resolve } = createResolver(import.meta.url)
    
    // Injection de la config publique
    nuxt.options.runtimeConfig.public.blogSeo = options
    
    // Plugin pour les meta tags
    addPlugin(resolve('./runtime/seo-plugin'))
    
    // Composable auto-importé
    addImports({
      name: 'useSeoMeta',
      from: resolve('./runtime/composables/useSeoMeta')
    })
  }
})
```

Les utilitaires `@nuxt/kit` essentiels pour un blog sont `createResolver` (chemins absolus), `addPlugin` (injection plugins), `addComponent` (composants globaux), `addImports` (composables auto-importés), et `addServerHandler` (routes API). Un changement notable dans Nuxt 4 : `installModule()` est déprécié au profit de `moduleDependencies` dans la définition du module.

Pour un blog, plutôt que recréer la logique SEO, **@nuxtjs/seo** combine sitemap, robots.txt, Schema.org et génération d'images OG en un seul module optimisé pour Cloudflare Pages.

---

## La configuration runtime sécurise vos secrets sur l'edge

La distinction entre `runtimeConfig` (privé) et `runtimeConfig.public` (exposé au client) est cruciale pour la sécurité. Les valeurs dans `runtimeConfig.public` sont **sérialisées dans le payload HTML** de chaque page et accessibles côté navigateur.

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare-pages',
    cloudflare: { nodeCompat: true }
  },
  
  runtimeConfig: {
    // Serveur uniquement - JAMAIS exposé au client
    contentfulToken: '',      // NUXT_CONTENTFUL_TOKEN
    emailApiKey: '',          // NUXT_EMAIL_API_KEY
    
    public: {
      // Client + Serveur - visible dans le HTML
      apiBase: '/api',        // NUXT_PUBLIC_API_BASE  
      siteUrl: '',            // NUXT_PUBLIC_SITE_URL
      gaTrackingId: ''        // NUXT_PUBLIC_GA_TRACKING_ID
    }
  }
})
```

**Particularité critique Cloudflare Pages** : les variables d'environnement ne sont pas accessibles globalement comme sur Node.js. Dans les routes serveur, passez toujours `event` à `useRuntimeConfig()` :

```typescript
// server/api/posts.ts
export default defineEventHandler(async (event) => {
  // ✅ Correct pour Cloudflare
  const { contentfulToken } = useRuntimeConfig(event)
  
  const posts = await $fetch('https://api.contentful.com/...', {
    headers: { Authorization: `Bearer ${contentfulToken}` }
  })
  return posts
})
```

Les conventions de nommage transforment automatiquement les variables : `NUXT_API_SECRET` devient `runtimeConfig.apiSecret`, et `NUXT_PUBLIC_API_BASE` devient `runtimeConfig.public.apiBase`. Pour le développement local avec `wrangler pages dev`, utilisez un fichier `.dev.vars` (pas `.env`) :

```env
NUXT_CONTENTFUL_TOKEN=ctf_dev_token
NUXT_PUBLIC_API_BASE=http://localhost:3001/api
NUXT_PUBLIC_SITE_URL=http://localhost:3000
```

Dans le Dashboard Cloudflare Pages, définissez ces mêmes variables sous **Settings → Environment Variables**. Un redéploiement est nécessaire après modification des variables de production.

---

## Patterns TypeScript pour Claude Code

Pour une génération de code cohérente avec Claude Code, établissez ces conventions dans votre projet :

```typescript
// types/plugins.d.ts - Déclaration des plugins
declare module '#app' {
  interface NuxtApp {
    $analytics: { track: (event: string, data?: Record<string, any>) => void }
    $markdown: { render: (content: string) => string }
  }
}

// Augmentation de la config runtime
declare module 'nuxt/schema' {
  interface PublicRuntimeConfig {
    apiBase: string
    siteUrl: string
    blogSeo: {
      siteName: string
      defaultOgImage: string
    }
  }
}
```

Cette déclaration permet à Claude Code de générer du code typé qui utilise `useNuxtApp().$analytics` ou `useRuntimeConfig().public.blogSeo` avec autocomplétion.

**Pattern reproductible pour un plugin blog** :

```typescript
// app/plugins/reading-time.ts
export default defineNuxtPlugin({
  name: 'reading-time',
  parallel: true,
  setup() {
    const calculateReadingTime = (content: string): number => {
      const wordsPerMinute = 200
      const words = content.trim().split(/\s+/).length
      return Math.ceil(words / wordsPerMinute)
    }
    
    return {
      provide: { readingTime: calculateReadingTime }
    }
  }
})
```

---

## Conclusion

L'architecture plugins/modules de Nuxt 4.2.X sur Cloudflare Pages repose sur quatre principes : **séparation contextuelle** via les suffixes pour optimiser les bundles et la sécurité, **parallélisation explicite** avec `parallel: true` et `dependsOn` pour réduire significativement le temps de démarrage, **encapsulation modulaire** via `@nuxt/kit` pour une base de code maintenable, et **isolation des secrets** entre `runtimeConfig` privé et public avec attention particulière au comportement de l'edge runtime.

Pour un blog, privilégiez les modules communautaires (`@nuxtjs/seo`, `@nuxt/content`) et créez des modules locaux uniquement pour la logique métier spécifique. Avec Claude Code, documentez vos conventions TypeScript dans des fichiers `.d.ts` pour garantir une génération de code cohérente avec votre architecture.