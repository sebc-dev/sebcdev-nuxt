# Guide complet du Language Switcher avec @nuxtjs/i18n v10.2+ et Nuxt 4

L'implémentation d'un sélecteur de langue accessible et performant dans Nuxt 4 avec @nuxtjs/i18n v10.2+ nécessite de maîtriser les nouveaux composables, d'adopter les patterns d'accessibilité WCAG 2.2, et d'éviter les pièges de migration. Pour un blog bilingue FR/EN en SSG sur Cloudflare Pages, la stratégie `prefix_except_default` combinée au composant **`<SwitchLocalePathLink>`** (plutôt que `useSwitchLocalePath()` directement) évite les problèmes d'hydratation tout en garantissant un SEO optimal.

---

## Les composables essentiels : useSwitchLocalePath() vs useLocalePath()

Ces deux composables servent des objectifs distincts et ne sont pas interchangeables. **`useSwitchLocalePath()`** retourne la version localisée de la **page courante** pour une locale donnée, tandis que **`useLocalePath()`** résout **n'importe quel chemin** selon la locale active ou spécifiée.

```typescript
// Signature de useSwitchLocalePath()
declare function useSwitchLocalePath(): (locale?: Locale) => string

// Signature de useLocalePath()
declare function useLocalePath(): (route: RawLocation, locale?: Locale) => string
```

| Composable | Entrée | Cas d'usage |
|------------|--------|-------------|
| `useSwitchLocalePath()` | Code locale uniquement | Language switcher |
| `useLocalePath()` | Route + locale optionnelle | Navigation générale |

Pour les **routes dynamiques** (`[slug]`, `[id]`), le composable **`useSetI18nParams()`** est indispensable pour traduire les paramètres de route :

```vue
<script setup>
// pages/articles/[slug].vue
const { data: article } = await useFetch('/api/article')
const setI18nParams = useSetI18nParams()

// Définir les slugs traduits pour chaque locale
setI18nParams({
  fr: { slug: article.value.slugs.fr },  // 'mon-article'
  en: { slug: article.value.slugs.en }   // 'my-article'
})
</script>
```

**Important pour le SSG** : utilisez le composant `<SwitchLocalePathLink>` plutôt que `switchLocalePath()` directement pour éviter les hydration mismatches. Ce composant met à jour les routes avant l'envoi de la réponse rendue :

```vue
<template>
  <SwitchLocalePathLink locale="en">English</SwitchLocalePathLink>
  <SwitchLocalePathLink locale="fr">Français</SwitchLocalePathLink>
</template>
```

---

## defineI18nRoute() est déprécié : utilisez definePageMeta()

Le macro **`defineI18nRoute()` est déprécié** dans v10 et sera supprimé en v11. La configuration des routes personnalisées se fait désormais via `definePageMeta()` avec la propriété `i18n`.

Activez d'abord le mode dans `nuxt.config.ts` :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  experimental: {
    scanPageMeta: true  // Requis pour les routes nommées personnalisées
  },
  i18n: {
    customRoutes: 'meta'  // Active definePageMeta approach
  }
})
```

Puis définissez vos routes dans chaque page :

```vue
<!-- pages/about.vue -->
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

Pour les **routes dynamiques avec Nuxt Content**, évitez les conflits en définissant explicitement les chemins traduits dans chaque fichier de page plutôt que dans la configuration globale. L'alternative via `customRoutes: 'config'` reste disponible :

```typescript
// nuxt.config.ts - Configuration centralisée alternative
i18n: {
  customRoutes: 'config',
  pages: {
    'about': { fr: '/a-propos', en: '/about' },
    'blog-[slug]': { fr: '/blog/[slug]', en: '/blog/[slug]' }
  }
}
```

---

## Accessibilité WCAG 2.2 pour le Language Switcher

Les exigences d'accessibilité pour un sélecteur de langue touchent plusieurs critères WCAG. Les **Success Criteria 3.1.1** (Language of Page) et **3.1.2** (Language of Parts) sont fondamentaux : chaque option de langue doit porter l'attribut `lang` pour une prononciation correcte par les lecteurs d'écran.

Pour un site **bilingue FR/EN, le pattern toggle (boutons inline)** est recommandé plutôt qu'un dropdown. Il nécessite un seul clic, affiche les deux options en permanence, et correspond aux conventions des sites gouvernementaux bilingues.

```vue
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

Les attributs ARIA essentiels incluent :
- **`aria-label`** sur le conteneur de navigation ("Sélection de langue" / "Language selection")
- **`aria-current="page"`** sur la langue active
- **`lang` et `hreflang`** sur chaque lien/option
- **`aria-expanded`** si vous utilisez un dropdown

**Évitez les drapeaux seuls** : ils représentent des pays, pas des langues. La Suisse a 4 langues officielles, l'espagnol est parlé dans 20+ pays. Utilisez les **noms natifs** ("Français", "English") ou les codes ISO avec texte accessible.

---

## Implémentation UI avec shadcn-vue et Reka UI

Pour un blog bilingue, le **ToggleGroup de shadcn-vue** offre l'équilibre optimal entre accessibilité et esthétique :

```vue
<script setup lang="ts">
import { ToggleGroup, ToggleGroupItem } from '@/components/ui/toggle-group'

const switchLocalePath = useSwitchLocalePath()
const { locale } = useI18n()
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

Le positionnement recommandé est en **haut à droite du header**, à côté des éléments utilitaires (recherche, thème). Pour TailwindCSS 4.x, utilisez les nouvelles variables CSS natives :

```css
@theme {
  --color-lang-active: var(--color-primary);
  --radius-lang: var(--radius-md);
}
```

---

## SEO hreflang et intégration sitemap

Le composable **`useLocaleHead()`** génère automatiquement les balises hreflang, le `lang` sur `<html>`, et les métadonnées OpenGraph. Implémentez-le dans votre layout principal :

```vue
<!-- layouts/default.vue -->
<script setup>
const head = useLocaleHead()
</script>

<template>
  <Html :lang="head.htmlAttrs.lang">
    <Head>
      <template v-for="link in head.link" :key="link.key">
        <Link :rel="link.rel" :href="link.href" :hreflang="link.hreflang" />
      </template>
    </Head>
    <Body><slot /></Body>
  </Html>
</template>
```

Pour le **x-default hreflang**, configurez `isCatchallLocale` sur votre locale par défaut :

```typescript
// nuxt.config.ts
i18n: {
  locales: [
    { 
      code: 'fr', 
      language: 'fr-FR', 
      file: 'fr.json',
      isCatchallLocale: true  // Génère x-default pour FR
    },
    { code: 'en', language: 'en-US', file: 'en.json' }
  ],
  defaultLocale: 'fr',
  baseUrl: 'https://votre-blog.com'  // Requis pour URLs absolues
}
```

L'intégration avec **@nuxtjs/sitemap** est automatique. Le module génère des sitemaps séparés par locale (`/fr-sitemap.xml`, `/en-sitemap.xml`) avec les hreflang correctement liés.

Le nouveau mode **`experimental.strictSeo`** de v10 automatise complètement la gestion hreflang sans nécessiter `useLocaleHead()` :

```typescript
i18n: {
  experimental: {
    strictSeo: {
      enabled: true,
      canonicalQueryParams: ['page']  // Inclure dans l'URL canonique
    }
  }
}
```

---

## Optimisation SSG et déploiement Cloudflare Pages

Pour le SSG complet avec pré-rendu des variantes linguistiques :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare-pages',
    prerender: {
      crawlLinks: true,  // Découvre automatiquement les routes localisées
      routes: ['/sitemap.xml', '/robots.txt']
    }
  },
  
  i18n: {
    bundle: {
      compositionOnly: true,  // Tree-shake Legacy API
      runtimeOnly: true       // Pas de compilateur dans le bundle (JSON uniquement)
    }
  }
})
```

**Le lazy loading est automatique** en v10 quand vous utilisez la propriété `file` ou `files` dans la configuration des locales. L'option `lazy: true` explicite est dépréciée.

Pour **Cloudflare Pages**, désactivez impérativement dans le dashboard :
- Rocket Loader™ (Speed > Optimization)
- Mirage (Image Optimization)
- Email Address Obfuscation (Scrape Shield)
- Auto Minify JavaScript/CSS/HTML

Ces optimisations Cloudflare interfèrent avec l'hydratation Vue et causent des erreurs côté client.

---

## Pièges courants et breaking changes v9 → v10

La migration vers v10 introduit plusieurs changements cassants critiques :

| Changement | Impact | Solution |
|------------|--------|----------|
| Vue I18n v10 → v11 | `$tc()` supprimé | Utiliser `$t()` avec paramètres |
| `lazy` option supprimée | Lazy loading toujours actif | Supprimer l'option |
| `onLanguageSwitched()` supprimé | Hook déprécié | Utiliser `'i18n:localeSwitched'` hook Nuxt |
| `defineI18nRoute()` déprécié | Macro obsolète | Utiliser `definePageMeta({ i18n: {} })` |
| `redirectOn: 'root'` corrigé | Comportement modifié | Utiliser `'all'` si redirection sur tous les chemins |

**Erreur fréquente Nuxt 4** : avec `future.compatibilityVersion: 4`, les fichiers i18n doivent être dans `<rootDir>/i18n/` (pas dans `app/`) :

```
project/
├── app/              # Code Nuxt 4
├── i18n/             # Fichiers i18n (à la racine)
│   ├── locales/
│   │   ├── fr.json
│   │   └── en.json
│   └── i18n.config.ts
└── nuxt.config.ts
```

Le chemin `vueI18n` doit être relatif à la racine :

```typescript
i18n: {
  vueI18n: './i18n/i18n.config.ts'  // Depuis la racine, pas /app
}
```

**Hydration mismatch avec routes dynamiques** : si vous utilisez `switchLocalePath()` directement avec `useSetI18nParams()`, les liens peuvent être rendus avant que les paramètres soient définis. La solution est d'utiliser le composant `<SwitchLocalePathLink>` qui gère correctement le timing.

---

## Configuration complète production-ready

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/i18n', '@nuxtjs/sitemap'],
  
  future: { compatibilityVersion: 4 },
  
  experimental: {
    scanPageMeta: true
  },

  i18n: {
    locales: [
      { 
        code: 'fr', 
        language: 'fr-FR', 
        file: 'fr.json',
        name: 'Français',
        isCatchallLocale: true
      },
      { 
        code: 'en', 
        language: 'en-US', 
        file: 'en.json',
        name: 'English'
      }
    ],
    defaultLocale: 'fr',
    strategy: 'prefix_except_default',
    baseUrl: 'https://votre-blog.com',
    
    detectBrowserLanguage: {
      useCookie: true,
      cookieKey: 'i18n_redirected',
      redirectOn: 'root',
      alwaysRedirect: false
    },
    
    customRoutes: 'meta',
    
    bundle: {
      compositionOnly: true,
      runtimeOnly: true
    },
    
    experimental: {
      strictSeo: true
    }
  },

  nitro: {
    preset: 'cloudflare-pages',
    prerender: {
      crawlLinks: true,
      routes: ['/sitemap.xml', '/robots.txt']
    }
  }
})
```

## Conclusion

L'implémentation d'un language switcher robuste avec @nuxtjs/i18n v10.2+ et Nuxt 4 repose sur trois piliers : l'utilisation du composant `<SwitchLocalePathLink>` pour éviter les hydration mismatches en SSG, l'adoption de `definePageMeta({ i18n: {} })` pour les routes personnalisées, et le respect des patterns d'accessibilité WCAG avec le toggle button pour sites bilingues. Le mode `strictSeo` expérimental simplifie considérablement la gestion hreflang en automatisant les balises meta. Pour Cloudflare Pages, la désactivation des optimisations automatiques (Rocket Loader, Mirage) est critique pour un rendu correct.