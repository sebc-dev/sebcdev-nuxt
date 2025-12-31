# Optimisations Bundle i18n (Tree-shaking)

Options de bundle pour réduire la taille du bundle client :

```typescript
// nuxt.config.ts
i18n: {
  bundle: {
    compositionOnly: true,  // Tree-shake Legacy API ($t, $tc, etc. dans Options API)
    runtimeOnly: true       // Pas de compilateur dans le bundle (JSON uniquement)
  }
}
```

| Option | Effet | Économie |
|--------|-------|----------|
| `compositionOnly: true` | Supprime le support Options API | ~30% taille vue-i18n |
| `runtimeOnly: true` | Supprime le compilateur de messages | ~20% taille vue-i18n |

**⚠️ `runtimeOnly: true` requis** : Uniquement si vos traductions sont en **JSON pur**. Si vous utilisez des messages inline dans le code (`$t('Hello {name}')` avec interpolation dynamique), gardez `runtimeOnly: false`.

**Vérification** : Les fichiers dans `i18n/locales/*.json` doivent contenir uniquement du JSON statique :

```json
// ✅ Compatible runtimeOnly: true
{ "greeting": "Bonjour {name}" }

// ❌ Incompatible (nécessite compilateur)
// Messages avec syntaxe ICU complexe ou fonctions
```
