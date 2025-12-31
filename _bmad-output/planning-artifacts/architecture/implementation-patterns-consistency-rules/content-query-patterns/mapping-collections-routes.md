# Mapping collections ↔ routes

| Locale | Collection | Route générée |
|--------|------------|---------------|
| `fr` | `articles_fr` | `/blog/mon-article` |
| `en` | `articles_en` | `/en/blog/my-article` |

Le `prefix: '/blog'` dans `content.config.ts` retire le préfixe de langue du path.
