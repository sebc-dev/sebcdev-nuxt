# Code Highlighting (Shiki)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Configuration path** | `content.build.markdown.highlight` | ⚠️ `content.highlight` (v2) est **déprécié** |
| **Dual-theme** | `{ default: 'github-light', dark: 'github-dark' }` | Switch automatique via classe `html.dark` |
| **Langages** | Set minimal dans `langs` | Réduit le bundle (évite `shiki/bundle/full` 6.4MB) |
| **Accessibilité** | Thèmes high-contrast | Ratio contraste ≥4.5:1 (WCAG AA) |

## Thèmes recommandés

| Mode | Thème standard | Thème high-contrast | Alternatifs |
|------|----------------|---------------------|-------------|
| Light | `github-light` | `github-light-high-contrast` | `min-light`, `vitesse-light` |
| Dark | `github-dark` | `github-dark-high-contrast` | `nord`, `one-dark-pro` |

**Paires équilibrées (WCAG AA - ratio ≥4.5:1) :**
- `github-light` / `github-dark` — Défaut recommandé, excellent contraste
- `github-light-high-contrast` / `github-dark-high-contrast` — Ratio ≥5.5:1, accessibilité maximale
- `min-light` / `min-dark` — Minimaliste, très lisible
- `vitesse-light` / `vitesse-dark` — Design épuré

**Critères de sélection :**
- Privilégier les thèmes `*-high-contrast` pour le contenu technique
- Tester le contraste des commentaires (souvent problématique)
- Utiliser `colorReplacements` pour corriger des couleurs spécifiques

⚠️ **Thème `css-variables` cassé** depuis Content 2.8.5+ (migration Shikiji). Utiliser les thèmes nommés.

## Correction couleurs low-contrast

Pour corriger des couleurs problématiques dans un thème :

```typescript
// nuxt.config.ts
content: {
  build: {
    markdown: {
      highlight: {
        theme: { default: 'min-light', dark: 'min-dark' },
        colorReplacements: {
          '#ff79c6': '#c678dd',  // Remplace couleur trop claire
          '#6272a4': '#5c6370'   // Améliore contraste commentaires
        }
      }
    }
  }
}
```

## Impact bundle size

| Bundle | Taille min | Taille gzip |
|--------|------------|-------------|
| `shiki/bundle/full` | 6.4 MB | 1.2 MB |
| `shiki/bundle/web` | 3.8 MB | 695 KB |
| Fine-grained (core) | ~12 KB | + thèmes/langages |

**Recommandation :** Nuxt Content utilise l'approche fine-grained via `@nuxtjs/mdc`. Limiter `langs` au strict nécessaire :

```typescript
langs: ['json', 'js', 'ts', 'html', 'css', 'vue', 'shell', 'yaml', 'md', 'mdc']
```
