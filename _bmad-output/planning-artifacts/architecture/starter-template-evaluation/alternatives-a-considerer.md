# Alternatives à Considérer

**Recherche** :
- Orama (8.2k+ stars) : Plus de fonctionnalités, API TypeScript-first, mais plus lourd (~15KB)
- Pagefind : Index post-build automatique, mais nécessite chunks externes

**Validation (Standard Schema compatible)** :

| Library | Taille (gzip / min) | Tree-shaking | Meilleur pour |
|---------|---------------------|--------------|---------------|
| **Valibot v1.0** | ~1KB / ~2KB | Excellent | Client-side, builds minimaux |
| Zod 4 (`zod/mini`) | ~2KB / ~5KB | Bon | Équilibre taille/écosystème |
| Zod 4 (complet) | ~5KB / ~10KB | Modéré | Schémas complexes, inférence TS |

**Bundle Analysis** :
- `npx nuxi analyze` : Intégré (vite-bundle-visualizer)
- nuxt-bundle-analysis : GitHub Action pour CI/CD
