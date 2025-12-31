# Optimisation Bundle (Vite)

Isoler MiniSearch dans son propre chunk évite d'alourdir le bundle principal :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  vite: {
    build: {
      rollupOptions: {
        output: {
          manualChunks: {
            // Sépare MiniSearch (~7KB) du bundle principal
            search: ['minisearch']
          }
        }
      }
    }
  }
})
```

**Impact :**

| Configuration | Bundle principal | Chunk search | Total |
|---------------|------------------|--------------|-------|
| Sans `manualChunks` | ~150KB | - | ~150KB |
| Avec `manualChunks` | ~143KB | ~7KB | ~150KB |

**Avantage** : Le chunk `search` n'est chargé qu'au premier accès à la recherche (lazy loading), réduisant le TTI (Time to Interactive) initial.
