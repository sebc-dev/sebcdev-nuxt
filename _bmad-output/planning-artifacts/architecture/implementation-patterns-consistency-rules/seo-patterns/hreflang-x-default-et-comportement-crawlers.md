# hreflang x-default et Comportement Crawlers

## Qu'est-ce que x-default ?

Le `x-default` désigne l'URL de **fallback** pour les utilisateurs dont la langue n'est pas supportée. Avec `strategy: 'prefix_except_default'`, l'URL sans préfixe (`/` = anglais) devient naturellement le candidat x-default.

**Important** : @nuxtjs/i18n **ne génère PAS** x-default via `useLocaleHead()`. Cependant, @nuxtjs/sitemap l'ajoute automatiquement dans le `sitemap.xml`.

## Comportement par moteur de recherche

| Moteur | Support hreflang | Notes |
|--------|------------------|-------|
| **Google** | ✅ Complet | HTML, HTTP headers, ou sitemap XML. Exige liens bidirectionnels |
| **Bing** | ❌ Non officiel | Utilise ses propres signaux (Bing Webmaster Tools) |
| **Yandex** | ✅ Partiel | Supporte hreflang basique |

## Sitemap i18n avec @nuxtjs/sitemap

L'intégration sitemap-i18n fonctionne **automatiquement** quand les deux modules sont installés. Le sitemap génère les balises `<xhtml:link rel="alternate" hreflang="">` incluant x-default :

```xml
<url>
  <loc>https://sebc.dev/</loc>
  <xhtml:link rel="alternate" href="https://sebc.dev/" hreflang="x-default" />
  <xhtml:link rel="alternate" href="https://sebc.dev/" hreflang="en-US" />
  <xhtml:link rel="alternate" href="https://sebc.dev/fr" hreflang="fr-FR" />
</url>
```

## _i18nTransform pour URLs dynamiques

Pour les URLs provenant d'une API ou d'un CMS, utilisez `_i18nTransform: true` pour générer automatiquement toutes les variantes localisées :

```typescript
// server/api/__sitemap__/urls.ts
export default defineSitemapEventHandler(() => [
  // Génère /about, /fr/a-propos avec hreflang complet
  { loc: '/about', _i18nTransform: true },

  // Pour les articles de blog
  { loc: '/blog/my-article', _i18nTransform: true },
])
```

**Désactiver l'intégration auto (rare) :**

```typescript
// nuxt.config.ts
sitemap: {
  autoI18n: false  // Désactive l'intégration automatique
}
```
