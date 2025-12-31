# Routes Personnalisées avec definePageMeta()

**⚠️ `defineI18nRoute()` est DÉPRÉCIÉ** en v10 et sera supprimé en v11. Utiliser `definePageMeta()` avec la propriété `i18n` :

## Activation (nuxt.config.ts)

```typescript
export default defineNuxtConfig({
  experimental: {
    scanPageMeta: true  // Requis pour les routes nommées personnalisées
  },
  i18n: {
    customRoutes: 'meta'  // Active l'approche definePageMeta
  }
})
```

## Définition par page

```vue
<!-- app/pages/about.vue -->
<script setup>
definePageMeta({
  i18n: {
    paths: {
      fr: '/a-propos',
      en: '/about'
    }
  }
})
</script>
```

## Alternative : Configuration centralisée

Pour une gestion centralisée des routes traduites :

```typescript
// nuxt.config.ts
i18n: {
  customRoutes: 'config',
  pages: {
    'about': { fr: '/a-propos', en: '/about' },
    'blog-[slug]': { fr: '/blog/[slug]', en: '/blog/[slug]' },
    'categories-[category]': { fr: '/categories/[category]', en: '/categories/[category]' }
  }
}
```

| Approche | Avantages | Inconvénients |
|----------|-----------|---------------|
| `customRoutes: 'meta'` | Co-localisé avec la page, visible dans le fichier | Dispersé dans le code |
| `customRoutes: 'config'` | Vue d'ensemble centralisée | Maintenance séparée |

**Recommandation** : `meta` pour projets avec peu de routes traduites, `config` pour projets complexes.
