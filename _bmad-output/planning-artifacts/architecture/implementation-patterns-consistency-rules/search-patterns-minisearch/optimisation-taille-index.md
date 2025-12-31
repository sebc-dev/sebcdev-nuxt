# Optimisation Taille Index

## Fonctions de nettoyage

```typescript
// app/lib/search-utils.ts (suite)

// Supprimer le Markdown pour réduire la taille
export function stripMarkdown(content: string): string {
  return content
    // Blocs de code
    .replace(/```[\s\S]*?```/g, '')
    .replace(/`[^`]*`/g, '')
    // Links (garder le texte)
    .replace(/\[([^\]]+)\]\([^)]+\)/g, '$1')
    // Images
    .replace(/!\[[^\]]*\]\([^)]+\)/g, '')
    // Headers
    .replace(/#{1,6}\s*/g, '')
    // Emphasis
    .replace(/[*_]{1,2}([^*_]+)[*_]{1,2}/g, '$1')
    // Whitespace
    .replace(/\s+/g, ' ')
    .trim()
}

// Tronquer le contenu long (économise ~40% sur gros articles)
export function truncateContent(content: string, maxLength = 3000): string {
  if (content.length <= maxLength) return content
  // Couper à la fin d'une phrase
  const truncated = content.slice(0, maxLength)
  const lastPeriod = truncated.lastIndexOf('.')
  return lastPeriod > maxLength * 0.8
    ? truncated.slice(0, lastPeriod + 1)
    : truncated
}
```

## Estimations de taille

| Articles | Index brut | Avec optimisations | Brotli (~85% compression) | Temps chargement (3G) |
|----------|------------|-------------------|--------------------------|----------------------|
| 20 | ~25KB | ~15KB | **~3KB** | <50ms |
| 100 | ~100KB | ~60KB | **~10-15KB** | ~100ms |
| 250 | ~250KB | ~150KB | **~25-40KB** | ~300ms |
| 500 | ~500KB | ~300KB | **~50-80KB** | ~600ms |
| 1000 | ~1MB | ~600KB | **~100-150KB** | ~1.2s |

**Note** : Cloudflare Pages applique automatiquement Brotli (~85% compression vs ~75% gzip). Les temps supposent une connexion 3G (~750KB/s).

**Impact des optimisations :**
- Stop words bilingues : **-20%**
- `stripMarkdown()` : **-15%**
- `truncateContent(3000)` : **-10-20%** (selon longueur articles)
- IDs entiers vs slugs : jusqu'à **-90%** (cas extrême documenté: 700KB → 62KB)

## Contraintes Cloudflare Pages

| Limite | Valeur | Impact sur la recherche |
|--------|--------|------------------------|
| **Taille max fichier** | 25 MiB | Index doit rester < 25MB |
| **Fichiers par site** | 20,000 | Non limitant pour index unique |
| **Bande passante** | Illimitée | Lazy loading sans coût |

**⚠️ Règle pratique** : Maintenir l'index < 10MB (gzipped < 3MB) pour un temps de chargement acceptable. Au-delà de 500 articles, envisager Pagefind.
