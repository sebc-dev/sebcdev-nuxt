# @nuxt/content v3 API Changes

**Composants supprimés (migration v2 → v3) :**
- ❌ `<ContentDoc />` - supprimé
- ❌ `<ContentList />` - supprimé
- ❌ `<ContentQuery />` - supprimé
- ✅ Utiliser `<ContentRenderer>` pour tout le rendu

**API de requête :**
```typescript
// ❌ ANCIEN (Content v2)
const posts = await queryContent('blog').find()

// ✅ NOUVEAU (Content v3)
const posts = await queryCollection('blog').all()
```

**Autres changements :**
- Mode document-driven supprimé - créer les pages manuellement
- Composants prose personnalisés dans `components/prose/` (non plus `components/content/`)
- Index de recherche MiniSearch : `public/search-index.json`
