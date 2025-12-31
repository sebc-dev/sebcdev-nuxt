# Hook i18n:localeSwitched

Exécuter du code après un changement de langue :

```typescript
// plugins/i18n-watcher.client.ts
export default defineNuxtPlugin((nuxtApp) => {
  // ⚠️ Remplace onLanguageSwitched() de v9
  nuxtApp.hook('i18n:localeSwitched', ({ oldLocale, newLocale }) => {
    console.log(`Langue changée: ${oldLocale} → ${newLocale}`)

    // Exemples d'actions post-switch :
    // - Analytics tracking
    // - Mise à jour préférences utilisateur
    // - Refresh de données externes
  })
})
```

## Autres hooks i18n disponibles

| Hook | Déclencheur | Usage |
|------|-------------|-------|
| `i18n:beforeLocaleSwitch` | Avant le changement | Validation, confirmation |
| `i18n:localeSwitched` | Après le changement | Analytics, refresh données |

```typescript
// Exemple: Confirmation avant changement
nuxtApp.hook('i18n:beforeLocaleSwitch', async ({ oldLocale, newLocale, initialSetup }) => {
  if (initialSetup) return // Pas de confirmation au chargement initial

  // Retourner false pour annuler le changement
  const confirmed = await confirmLanguageChange()
  if (!confirmed) {
    return false
  }
})
```
