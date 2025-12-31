# Implémentation hreflang automatique avec @nuxtjs/i18n 10.2+ et Nuxt 4 SSG

L'implémentation SEO multilingue avec @nuxtjs/i18n 10.2+ nécessite une configuration précise du composable `useLocaleHead()`, une intégration correcte avec @nuxtjs/sitemap 7.5+, et une attention particulière aux contraintes SSG de Cloudflare Pages. La version 10 introduit des changements majeurs : lazy loading activé par défaut, nouveau mode `strictSeo` expérimental, et restructuration obligatoire du dossier i18n. Les erreurs les plus fréquentes concernent l'absence de la propriété `language` sur les locales et le manque de `baseUrl` pour générer des URLs absolues.

## Configuration fondamentale de useLocaleHead() dans Nuxt 4

Le composable `useLocaleHead()` génère automatiquement les balises hreflang, les meta OpenGraph locale, et les URLs canoniques. Pour Nuxt 4 avec sa nouvelle structure `app/`, le dossier i18n doit impérativement rester **à la racine du projet** (pas dans `app/`) car les fichiers sont utilisés côté client et serveur.

**Structure de répertoire Nuxt 4 recommandée :**
```
project/
├── app/
│   ├── pages/
│   ├── layouts/
│   └── app.vue
├── i18n/                    # À la racine, PAS dans app/
│   ├── locales/
│   │   ├── en-US.json
│   │   └── fr-FR.json
│   └── i18n.config.ts
├── server/
└── nuxt.config.ts
```

La configuration minimale pour un SEO fonctionnel exige deux propriétés souvent oubliées : **`language`** (format BCP 47) sur chaque locale et **`baseUrl`** pour les URLs absolues :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/i18n', '@nuxtjs/sitemap'],
  i18n: {
    locales: [
      { code: 'en', language: 'en-US', file: 'en-US.json' },
      { code: 'fr', language: 'fr-FR', file: 'fr-FR.json' }
    ],
    defaultLocale: 'en',
    strategy: 'prefix_except_default',
    baseUrl: 'https://mon-site.com'  // OBLIGATOIRE pour hreflang
  }
})
```

## Implémentation useLocaleHead() dans les layouts

L'intégration correcte dans `layouts/default.vue` utilise les composants Meta de Nuxt pour assurer la compatibilité SSR/SSG :

```vue
<script setup>
const head = useLocaleHead({ seo: { canonicalQueries: [] } })
</script>

<template>
  <Html :lang="head.htmlAttrs.lang" :dir="head.htmlAttrs.dir">
    <Head>
      <template v-for="link in head.link" :key="link.key">
        <Link :rel="link.rel" :href="link.href" :hreflang="link.hreflang" />
      </template>
      <template v-for="meta in head.meta" :key="meta.key">
        <Meta :property="meta.property" :content="meta.content" />
      </template>
    </Head>
    <Body><slot /></Body>
  </Html>
</template>
```

Le composable génère automatiquement : les balises `<link rel="alternate" hreflang="x">` pour toutes les locales configurées, l'attribut `lang` sur `<html>`, les meta `og:locale` et `og:locale:alternate` (format underscore : `en_US`), le lien canonical auto-référencé par langue, et les liens "catchall" pour les groupes linguistiques (ex: `hreflang="en"` pour le groupe `en-*`).

## Gestion du x-default et stratégie prefix_except_default

Le `x-default` désigne l'URL de fallback pour les utilisateurs dont la langue n'est pas supportée. **Important** : @nuxtjs/i18n ne génère pas automatiquement le x-default dans `useLocaleHead()`. Le module @nuxtjs/sitemap l'ajoute cependant dans le sitemap.xml.

Avec `strategy: 'prefix_except_default'`, la locale par défaut n'a pas de préfixe URL (`/` = anglais, `/fr` = français). Cette URL sans préfixe devient naturellement le candidat x-default. Pour les crawlers :

- **Google** : Supporte pleinement hreflang via HTML, HTTP headers, ou sitemap XML. Exige des liens bidirectionnels et auto-référencés.
- **Bing** : Ne supporte **pas** officiellement hreflang. Utilisez plutôt `<meta http-equiv="content-language" content="fr-FR">`.

Pour une locale catchall spécifique dans un groupe linguistique :

```typescript
locales: [
  { code: 'en', language: 'en-US' },
  { code: 'gb', language: 'en-GB', isCatchallLocale: true }  // Override catchall
]
```

## Intégration @nuxtjs/sitemap 7.5+ avec génération hreflang

L'intégration sitemap-i18n fonctionne **automatiquement sans configuration supplémentaire** quand les deux modules sont installés. Le sitemap détecte @nuxtjs/i18n au build et génère les balises `<xhtml:link rel="alternate" hreflang="">` incluant le x-default.

**XML généré :**
```xml
<url>
  <loc>https://example.com/</loc>
  <xhtml:link rel="alternate" href="https://example.com/" hreflang="x-default" />
  <xhtml:link rel="alternate" href="https://example.com/" hreflang="en-US" />
  <xhtml:link rel="alternate" href="https://example.com/fr" hreflang="fr-FR" />
</url>
```

Pour les URLs dynamiques (API, CMS), utilisez `_i18nTransform: true` pour générer toutes les variantes localisées :

```typescript
// server/api/__sitemap__/urls.ts
export default defineSitemapEventHandler(() => [
  { loc: '/about', _i18nTransform: true }  // Génère /about, /fr/a-propos, etc.
])
```

L'option `autoI18n` permet de désactiver ou personnaliser l'intégration :

```typescript
sitemap: {
  autoI18n: false,  // Désactiver
  // ou configuration manuelle :
  autoI18n: {
    locales: [...],
    defaultLocale: 'en',
    strategy: 'prefix_except_default'
  }
}
```

## Canonical URLs cross-language et évitement du contenu dupliqué

**Règle critique** : chaque version linguistique doit avoir un canonical auto-référencé. Ne jamais pointer le canonical vers une autre langue — cela invalide tous les signaux hreflang.

```html
<!-- Sur /fr/page - CORRECT -->
<link rel="canonical" href="https://example.com/fr/page" />
<link rel="alternate" hreflang="fr-FR" href="https://example.com/fr/page" />
<link rel="alternate" hreflang="en-US" href="https://example.com/page" />
```

Le contenu traduit n'est **pas** considéré comme dupliqué par Google. Seul le contenu identique non traduit pose problème. L'option `canonicalQueries` permet de conserver certains paramètres dans l'URL canonique :

```typescript
const head = useLocaleHead({
  seo: { canonicalQueries: ['utm_source', 'ref'] }
})
```

## Contraintes SSG sur Cloudflare Pages

Nuxt détecte automatiquement l'environnement Cloudflare Pages. Pour le SSG, configurez :

```typescript
export default defineNuxtConfig({
  ssr: true,
  nitro: {
    preset: 'cloudflare_pages',
    static: true,
    prerender: {
      crawlLinks: true,
      routes: ['/', '/fr', '/en/about', '/fr/a-propos']
    }
  }
})
```

**Problème connu** : avec `strategy: 'prefix'`, le fichier `index.html` peut ne pas être généré. Workaround avec un hook Nitro pour créer une redirection :

```typescript
nitro: {
  hooks: {
    'prerender:generate'(route, nitro) {
      if (route?.route === '/200.html') {
        writeFileSync(
          path.join(nitro.options.output.publicDir, 'index.html'),
          '<!DOCTYPE html><html><head><meta http-equiv="refresh" content="0; url=/en"></head></html>'
        )
      }
    }
  }
}
```

Pour les problèmes de résolution de packages, inlinez les dépendances i18n :

```typescript
nitro: {
  inlinePackages: ['@intlify/core', '@intlify/utils', 'vue-i18n']
}
```

## Anti-patterns et pièges courants de la v10

Les erreurs les plus fréquentes à éviter :

- **Manque de `language`** : `locales: ['en', 'fr']` ne génère aucun hreflang. Utilisez `{ code: 'en', language: 'en-US' }`.
- **Absence de `baseUrl`** : génère des URLs relatives inutilisables pour le SEO.
- **Canonical cross-langue** : pointer le canonical français vers la page anglaise invalide tout le hreflang.
- **Utiliser `setLocale()` pour changer de langue** : change la locale sans l'URL. Préférez `useSwitchLocalePath()`.
- **Combiner incorrectement `useLocaleHead()` et `useSetI18nParams()`** : cause des tags dupliqués au refresh.

**Breaking changes majeurs v10 :**

| Changement | Impact |
|------------|--------|
| `lazy` supprimé | Toujours activé par défaut |
| `iso` → `language` | Renommer la propriété |
| `restructureDir` obligatoire | Dossier `i18n/` à la racine |
| `redirectOn: 'root'` corrigé | Seul `/` redirige, pas `/search` |
| `onLanguageSwitched()` supprimé | Utiliser le hook `i18n:localeSwitched` |

## Mode strictSeo expérimental (v10+)

Le nouveau mode `strictSeo` automatise complètement la gestion des tags SEO i18n — **`useLocaleHead()` devient inutile** (et lance une erreur si appelé) :

```typescript
i18n: {
  experimental: {
    strictSeo: true,
    // ou avec options :
    strictSeo: { canonicalQueryParams: ['ref'] }
  }
}
```

Avantages : pas de tags alternatifs générés pour les locales non supportées sur les routes dynamiques, et `<SwitchLocalePathLink>` désactive automatiquement les liens vers les locales indisponibles. Ce mode deviendra probablement le défaut en v11.

## Conclusion

L'implémentation hreflang réussie avec @nuxtjs/i18n 10.2+ repose sur trois piliers : une configuration complète avec `language` et `baseUrl`, l'utilisation correcte de `useLocaleHead()` dans les layouts (ou le mode strictSeo), et l'intégration native avec @nuxtjs/sitemap pour le XML. Le déploiement SSG sur Cloudflare Pages nécessite une attention aux routes pre-render et aux potentiels problèmes d'index.html avec la stratégie prefix. La migration depuis v9 demande de renommer `iso` en `language`, supprimer l'option `lazy`, et adapter le comportement de détection de langue si vous utilisez `redirectOn: 'root'`.