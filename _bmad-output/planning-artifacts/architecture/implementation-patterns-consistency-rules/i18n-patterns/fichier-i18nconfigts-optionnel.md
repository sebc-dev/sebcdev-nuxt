# Fichier i18n.config.ts (optionnel)

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
