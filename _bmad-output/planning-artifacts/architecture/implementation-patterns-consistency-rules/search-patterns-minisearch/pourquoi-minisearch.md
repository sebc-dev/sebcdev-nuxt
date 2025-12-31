# Pourquoi MiniSearch ?

| Critère | MiniSearch 7.x | Pagefind | Lunr.js | Fuse.js |
|---------|----------------|----------|---------|---------|
| **Taille bundle** | ~7KB gzip | ~8KB + chunks | ~8KB gzip | ~4KB gzip |
| **Index 500 posts** | ~150KB | ~100KB chunked | ~500KB+ | N/A (full data) |
| **Latence recherche** | **<10ms** | <100ms | <10ms | **1-5 secondes** |
| **Mémoire** | Modérée | Minimale (chunked) | Élevée | Élevée |
| **Fuzzy search** | ✅ Natif | ✅ Natif | ✅ Natif | ✅ Natif |
| **Prefix search** | ✅ Natif | ✅ Natif | ❌ Plugin | ✅ Natif |
| **Boosting** | ✅ Field + Document + Term | ❌ Limité | ✅ Field | ❌ Limité |
| **Async loading** | ✅ `loadJSONAsync()` | ✅ Chunks | ❌ | ❌ |
| **Multilingue** | Manuel (snowball) | ✅ Built-in | Plugins | Manuel |
| **Intégration SSG** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |

**⚠️ Fuse.js est à exclure** pour les blogs : sa recherche linéaire O(n) produit des latences de **1-5 secondes** à 500 posts avec contenu complet. Lunr.js souffre d'index volumineux : 700 documents peuvent générer ~70MB en mémoire.

MiniSearch utilise une structure **radix tree** optimisée pour la mémoire, ce qui le rend idéal pour les blogs avec 100-500 articles où la taille de l'index peut devenir significative.
