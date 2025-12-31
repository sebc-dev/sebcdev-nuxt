# SSG Build Performance

## Benchmark temps de build

Nuxt Content v3 affiche une complexité **O(n²)** pour les builds SSG avec de grands volumes de contenu (benchmarks communautaires) :

| Documents | Build Time approx | Recommandation |
|-----------|-------------------|----------------|
| 100-500 | ~30s - 1 min | ✅ Optimal |
| 500-1,000 | ~1-2 min | ✅ Acceptable |
| 1,000-2,000 | ~2-3 min | ⚠️ Surveiller |
| 2,000-5,000 | ~3-6 min | ⚠️ Considérer split |
| 5,000-10,000 | ~6-12 min | ❌ Split collections |

**Stratégies pour grands volumes :**
- Diviser les collections (par année, par pilier, etc.)
- Utiliser `crawlLinks: true` avec parcimonie
- Réduire les transformations MDC complexes

## Optimisation prerender partagé

Activer le partage de données entre routes prérendues pour réduire les requêtes redondantes :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  experimental: {
    // Partage les données entre pages prérendues (Nuxt 3.10+)
    sharedPrerenderData: true,

    // Extraction des payloads pour mise en cache
    payloadExtraction: true
  }
})
```

**Impact :**
- Réduit les requêtes `queryCollection` dupliquées entre pages
- Les données communes (listes, navigation) sont calculées une seule fois
- Particulièrement efficace pour les blogs avec sidebars et navigation partagées

**Exemple de gain :**
| Configuration | Requêtes totales | Build time |
|---------------|------------------|------------|
| Sans optimisation | 500 requêtes | 45s |
| `sharedPrerenderData: true` | 180 requêtes | 28s |
