# Breaking Changes @nuxtjs/i18n v9 → v10

| Aspect | v9 (ancien) | v10 (actuel) |
|--------|-------------|--------------|
| **Lazy loading** | `lazy: true` optionnel | Toujours lazy (défaut) |
| **`redirectOn: 'root'`** | Redirige toutes pages sans préfixe | Redirige **uniquement** `/` |
| **Hook changement langue** | `onLanguageSwitched()` | `i18n:localeSwitched` hook |
| **Vue I18n** | v9 | v11 |
| **Nitro detection** | Non disponible | Language detection ajoutée |
| **Locale field** | `iso` | `language` (pour hreflang) |

**Migration `redirectOn` :**

```typescript
// v9 - Ce comportement redirigeait TOUTES les pages
detectBrowserLanguage: {
  redirectOn: 'root'  // → Redirige /about, /blog, etc.
}

// v10 - Même config, comportement différent
detectBrowserLanguage: {
  redirectOn: 'root'  // → Redirige UNIQUEMENT /
}

// v10 - Pour retrouver le comportement v9
detectBrowserLanguage: {
  redirectOn: 'all'  // → Redirige toutes les pages
}
```

**Migration hook :**

```typescript
// ❌ v9 (obsolète)
onLanguageSwitched((oldLocale, newLocale) => {
  console.log(oldLocale, newLocale)
})

// ✅ v10 (correct)
nuxtApp.hook('i18n:localeSwitched', ({ oldLocale, newLocale }) => {
  console.log(oldLocale, newLocale)
})
```

**Migration locales config :**

```typescript
// ❌ v9 - iso pour hreflang
locales: [
  { code: 'en', iso: 'en-US', name: 'English' }
]

// ✅ v10 - language remplace iso + isCatchallLocale pour x-default
locales: [
  {
    code: 'fr',
    language: 'fr-FR',
    name: 'Français',
    file: 'fr.json',
    isCatchallLocale: true  // Génère hreflang="x-default" pour FR
  },
  { code: 'en', language: 'en-US', name: 'English', file: 'en.json' }
]
```

**Configuration `isCatchallLocale`** : Définit quelle locale génère le tag `hreflang="x-default"`. Utilisé par les moteurs de recherche pour les visiteurs dont la langue n'est pas supportée.
