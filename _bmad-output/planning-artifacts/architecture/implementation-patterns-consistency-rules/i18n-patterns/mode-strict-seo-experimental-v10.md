# Mode strictSeo expérimental (v10+)

Le nouveau mode `strictSeo` automatise **complètement** la gestion des tags SEO i18n. Quand activé, `useLocaleHead()` devient **inutile** (et lance une erreur si appelé) :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  i18n: {
    experimental: {
      strictSeo: true,
      // ou avec options :
      strictSeo: { canonicalQueryParams: ['ref'] }
    }
  }
})
```

**Avantages du mode strictSeo :**

| Fonctionnalité | Comportement |
|----------------|--------------|
| Tags hreflang | Générés automatiquement sans `useLocaleHead()` |
| Locales non supportées | Pas de tags alternatifs générés sur routes dynamiques |
| `<SwitchLocalePathLink>` | Désactive automatiquement les liens vers locales indisponibles |
| Migration | Deviendra probablement le défaut en v11 |

**Note :** Ce mode est encore expérimental. Pour la production actuelle, continuez à utiliser `useLocaleHead()` dans les layouts.
