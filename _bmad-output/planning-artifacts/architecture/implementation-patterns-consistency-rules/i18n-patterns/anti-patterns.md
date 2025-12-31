# Anti-patterns

| ❌ Anti-pattern | ✅ Pattern correct |
|-----------------|-------------------|
| Hardcoder la locale dans les composants | Utiliser `useI18n()` |
| Ignorer `watch: [locale]` dans useAsyncData | Toujours inclure pour content |
| Utiliser `onLanguageSwitched()` (v9) | Utiliser hook `i18n:localeSwitched` |
| Oublier `language` dans locales config | Requis pour hreflang auto |
| `setLocale('fr')` seul | `navigateTo(switchLocalePath('fr'))` |
| Canonical cross-langue | Canonical auto-référencé par langue |
| `defineI18nRoute()` (v9 déprécié) | `definePageMeta({ i18n: {} })` |
| `$tc()` pour pluralisation | `$t(key, count)` (Vue I18n v11) |
| Language switcher sans attributs `lang` | Ajouter `lang` et `hreflang` sur chaque option |
| Drapeaux seuls comme indicateurs | Codes ISO ou noms natifs ("Français") |

## setLocale() vs useSwitchLocalePath()

`setLocale()` change la locale **sans changer l'URL** — cela crée une incohérence entre l'état i18n et l'URL affichée.

```typescript
// ❌ MAUVAIS : Change la locale mais pas l'URL
const { setLocale } = useI18n()
await setLocale('fr')
// URL reste /about au lieu de /fr/about

// ✅ BON : Change la locale ET l'URL
const switchLocalePath = useSwitchLocalePath()
await navigateTo(switchLocalePath('fr'))
// URL devient /fr/about, locale synchronisée
```

**Cas d'usage de `setLocale()`** : Uniquement pour des changements temporaires sans navigation (ex: preview d'une traduction).

## Clé useAsyncData et fragmentation du cache

La clé de `useAsyncData` affecte le comportement du cache. Pour le contenu multilingue :

```typescript
// ⚠️ Fragmente le cache (2 entrées séparées pour le même article)
const { data } = await useAsyncData(
  `blog-${slug.value}-${locale.value}`,  // Clé unique par locale
  () => queryCollection(...).path(slug.value).first(),
  { watch: [locale] }
)

// ✅ Optimisé : clé indépendante de la locale (watch gère le refetch)
const { data } = await useAsyncData(
  `blog-${slug.value}`,  // Clé basée sur le path uniquement
  () => queryCollection(`articles_${locale.value}`).path(slug.value).first(),
  { watch: [locale] }
)
```

| Stratégie clé | Comportement cache | Usage |
|---------------|-------------------|-------|
| Avec locale | Cache séparé par langue | Quand les données EN et FR doivent coexister |
| Sans locale | Cache partagé, refetch au switch | **Recommandé** pour économiser mémoire |

**Recommandation** : Utiliser des clés sans locale quand `watch: [locale]` est présent. Le refetch remplace les données correctement.
