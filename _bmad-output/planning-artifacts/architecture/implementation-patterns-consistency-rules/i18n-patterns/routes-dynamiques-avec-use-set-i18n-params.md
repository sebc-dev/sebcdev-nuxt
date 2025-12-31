# Routes Dynamiques avec useSetI18nParams()

Pour les routes dynamiques (`[slug]`, `[id]`), le composable `useSetI18nParams()` permet de traduire les paramètres de route entre langues :

```vue
<!-- app/pages/blog/[slug].vue -->
<script setup lang="ts">
const route = useRoute()
const setI18nParams = useSetI18nParams()

// Récupérer l'article avec ses slugs traduits
const { data: article } = await useAsyncData(
  `article-${route.params.slug}`,
  () => queryCollection('articles_fr').path(`/blog/${route.params.slug}`).first()
)

// Définir les slugs traduits pour chaque locale
// Permet au language switcher de pointer vers le bon slug
if (article.value?.slugs) {
  setI18nParams({
    fr: { slug: article.value.slugs.fr },  // 'mon-article'
    en: { slug: article.value.slugs.en }   // 'my-article'
  })
}
</script>
```

**Cas d'usage :**

| Scénario | Sans `useSetI18nParams()` | Avec `useSetI18nParams()` |
|----------|---------------------------|---------------------------|
| FR → EN sur `/blog/mon-article` | `/en/blog/mon-article` (404) | `/en/blog/my-article` ✅ |

**Frontmatter pour slugs traduits :**

```yaml
# content/fr/blog/mon-article.md
---
title: "Mon Article"
slugs:
  fr: "mon-article"
  en: "my-article"
---
```
