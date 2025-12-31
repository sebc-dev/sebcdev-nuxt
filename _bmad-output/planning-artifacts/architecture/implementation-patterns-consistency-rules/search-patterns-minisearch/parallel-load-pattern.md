# Parallel Load Pattern

Charger MiniSearch et l'index simultanément pour réduire le temps de chargement total :

```typescript
// ❌ Séquentiel (~400ms)
const MiniSearch = (await import('minisearch')).default
const indexData = await fetch('/search-index.json').then(r => r.text())

// ✅ Parallèle (~250ms) — 40% plus rapide
const [MiniSearchModule, indexData] = await Promise.all([
  import('minisearch'),
  fetch('/search-index.json').then(r => r.text())
])

const miniSearch = await MiniSearchModule.default.loadJSONAsync(
  indexData,
  MINISEARCH_OPTIONS
)
```

**Impact mesuré :**

| Approche | Temps total | Économie |
|----------|-------------|----------|
| Séquentiel | ~400ms | - |
| Parallèle | ~250ms | **~40%** |
