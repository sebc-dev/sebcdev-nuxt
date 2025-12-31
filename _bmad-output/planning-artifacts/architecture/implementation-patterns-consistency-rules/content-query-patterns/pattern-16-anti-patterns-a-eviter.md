# Pattern 16 : Anti-patterns à éviter

## ❌ Requête dans `onMounted()`

```typescript
// ❌ MAUVAIS - Exécute uniquement côté client, casse le SSG
onMounted(async () => {
  const data = await queryCollection('blog').all()
})

// ✅ BON - useAsyncData au top-level
const { data } = await useAsyncData('blog', () =>
  queryCollection('blog').all()
)
```

**Pourquoi** : `onMounted()` s'exécute uniquement côté client. Le prerendering SSG ne verra pas ces données.

## ❌ Requête sur champs imbriqués

```typescript
// ❌ MAUVAIS - SQL ne supporte pas la notation pointée
queryCollection('blog')
  .where('meta.published', '=', true)
  .all()

// ✅ BON - Aplatir le schema
// content.config.ts
schema: z.object({
  published: z.boolean()  // Au lieu de meta.published
})

// Requête
queryCollection('blog')
  .where('published', '=', true)
  .all()
```

**Pourquoi** : La base SQLite stocke les données à plat. Les objets imbriqués sont sérialisés en JSON.

## ❌ Clés `useAsyncData` dupliquées

```typescript
// ❌ MAUVAIS - Même clé pour requêtes différentes (cache collision)
await useAsyncData('blog', () =>
  queryCollection('blog').where('tag', '=', 'vue').all()
)
await useAsyncData('blog', () =>
  queryCollection('blog').where('tag', '=', 'react').all()
)

// ✅ BON - Clés uniques incluant les paramètres
await useAsyncData(`blog-tag-vue`, () =>
  queryCollection('blog').where('tag', '=', 'vue').all()
)
await useAsyncData(`blog-tag-react`, () =>
  queryCollection('blog').where('tag', '=', 'react').all()
)
```

**Pourquoi** : Nuxt utilise la clé pour la déduplication et le cache. Des clés identiques provoquent des collisions silencieuses.

## ❌ Préfixes numériques sans zero-padding

```
# ❌ MAUVAIS - Tri alphabétique incorrect
content/
├── 1.intro.md
├── 10.advanced.md
├── 2.basics.md

# ✅ BON - Zero-padding pour tri correct
content/
├── 01.intro.md
├── 02.basics.md
├── 10.advanced.md
```

**Pourquoi** : `.order()` utilise le tri alphabétique. `"10"` vient avant `"2"` alphabétiquement.

## ❌ Oublier `watch` pour les paramètres dynamiques

```typescript
// ❌ MAUVAIS - Ne se met pas à jour quand locale change
const { data } = await useAsyncData('articles', () =>
  queryCollection(`articles_${locale.value}`).all()
)

// ✅ BON - Re-fetch automatique
const { data } = await useAsyncData('articles', () =>
  queryCollection(`articles_${locale.value}`).all(),
  { watch: [locale] }
)
```

**Pourquoi** : Sans `watch`, la requête ne se ré-exécute pas quand la dépendance change.

---
