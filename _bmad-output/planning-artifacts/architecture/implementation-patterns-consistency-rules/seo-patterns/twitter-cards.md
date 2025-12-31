# Twitter Cards

## Comportement des fallbacks

Twitter utilise l'attribut `name` (pas `property`) et **requiert obligatoirement `twitter:card`**. Cependant, les autres propriétés tombent sur leurs équivalents OG :

| Twitter | Fallback OG | Obligatoire |
|---------|------------|-------------|
| `twitter:card` | ❌ Aucun | **OUI** |
| `twitter:title` | `og:title` | Non |
| `twitter:description` | `og:description` | Non |
| `twitter:image` | `og:image` | Non |

```typescript
useSeoMeta({
  // Ces props OG servent de fallback pour Twitter
  ogTitle: 'Mon Article',
  ogDescription: 'Description de mon article',
  ogImage: 'https://sebc.dev/og.png',

  // twitter:card est OBLIGATOIRE (pas de fallback)
  twitterCard: 'summary_large_image',

  // Optionnels mais recommandés
  twitterSite: '@sebcdev',     // Compte du site
  twitterCreator: '@sebcdev', // Compte de l'auteur
})
```

## `summary_large_image` vs `summary`

| Type | Dimensions image | Recommandé pour |
|------|-----------------|-----------------|
| `summary_large_image` | 1200×628px min | **Blog** (image proéminente) |
| `summary` | 144×144px | Profils, pages sans visuel |

**Recommandation :** Toujours utiliser `summary_large_image` pour un blog.
