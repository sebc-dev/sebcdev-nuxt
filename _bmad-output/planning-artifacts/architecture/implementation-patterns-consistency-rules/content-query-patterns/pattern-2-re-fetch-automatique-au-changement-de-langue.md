# Pattern 2 : Re-fetch automatique au changement de langue

L'option `watch: [locale]` déclenche automatiquement un re-fetch quand la locale change :

```typescript
// pages/blog/index.vue
<script setup lang="ts">
import type { Collections } from '@nuxt/content'

const { locale } = useI18n()

const { data: posts } = await useAsyncData(
  `blog-list-${locale.value}`,
  () => {
    const collection = `articles_${locale.value}` as keyof Collections
    return queryCollection(collection)
      .where('draft', '=', false)
      .order('publishedAt', 'DESC')
      .all()
  },
  {
    watch: [locale]  // Re-fetch automatique au changement de langue
  }
)
</script>
```

**Comportement :**
- Premier chargement : fetch des données de la collection courante
- Changement de locale (ex: FR → EN) : fetch de `articles_en` au lieu de `articles_fr`
- La clé `blog-list-${locale.value}` garantit un cache séparé par langue
