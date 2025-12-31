# Lazy Loading Translations

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
