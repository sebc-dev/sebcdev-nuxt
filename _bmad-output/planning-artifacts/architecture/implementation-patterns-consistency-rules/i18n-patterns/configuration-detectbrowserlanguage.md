# Configuration detectBrowserLanguage

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

## Options redirectOn

| Valeur | Comportement | Usage |
|--------|--------------|-------|
| `'root'` | Redirige **uniquement** `/` vers `/fr/` | **Recommandé** - moins intrusif |
| `'all'` | Redirige toutes les pages sans préfixe | Migration depuis v9 |
| `'no prefix'` | Redirige pages sans préfixe de langue | Similaire à `'all'` |

⚠️ **Breaking change v9→v10** : `redirectOn: 'root'` ne redirige maintenant QUE la racine. Si vous aviez le comportement v9 (redirection sur toutes les pages), utilisez `redirectOn: 'all'`.
