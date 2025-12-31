# Configuration source avec prefix pour coordination i18n

Le `prefix` dans la configuration source **retire le préfixe de langue** du path généré, permettant à @nuxtjs/i18n de gérer les URLs :

```typescript
// content.config.ts
export default defineContentConfig({
  collections: {
    articles_en: defineCollection({
      source: {
        include: 'en/blog/**',
        prefix: '/blog'  // Génère /blog/article au lieu de /en/blog/article
      },
    }),
    articles_fr: defineCollection({
      source: {
        include: 'fr/blog/**',
        prefix: '/blog'  // Génère /blog/article au lieu de /fr/blog/article
      },
    }),
  }
})
```

**Coordination Content + i18n :**

| Fichier source | Path Content | URL finale (i18n) |
|----------------|--------------|-------------------|
| `content/en/blog/my-post.md` | `/blog/my-post` | `/blog/my-post` (EN, défaut) |
| `content/fr/blog/mon-article.md` | `/blog/mon-article` | `/fr/blog/mon-article` |

Le prefix `/blog` dans source évite la duplication du préfixe de langue entre Content et i18n.
