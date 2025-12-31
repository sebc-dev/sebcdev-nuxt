# Résumé migration v9 → v10

| Aspect | v9 | v10 |
|--------|----|----|
| Lazy loading | `lazy: true` optionnel | Toujours lazy (défaut) |
| `redirectOn: 'root'` | Redirige toutes pages sans préfixe | Redirige **uniquement** `/` |
| Hook changement langue | `onLanguageSwitched()` | `i18n:localeSwitched` hook |
| Vue I18n | v9 | v11 |
| Nitro language detection | Non disponible | Ajouté |
| Routes personnalisées | `defineI18nRoute()` | `definePageMeta({ i18n: {} })` |
| Pluralisation | `$tc(key, count)` | `$t(key, count)` ou `$t(key, { n: count })` |
| Champ locale | `iso` | `language` (requis pour hreflang) |

## Migration $tc() → $t() (Vue I18n v11)

**`$tc()` est supprimé** dans Vue I18n v11. La pluralisation se fait maintenant avec `$t()` :

```typescript
// ❌ ANCIEN (Vue I18n v9)
$tc('items', 5)
$tc('items', 5, { count: 5 })

// ✅ NOUVEAU (Vue I18n v11)
$t('items', 5)
$t('items', { n: 5 })
$t('items', { count: 5 }, 5)  // Avec contexte additionnel
```

**Format des messages de pluralisation :**

```json
{
  "items": "no item | one item | {n} items"
}
```

Les séparateurs `|` définissent les formes : zéro, singulier, pluriel.
