# Multilingue (og:locale)

## Format og:locale vs BCP 47

Le format `og:locale` utilise un **underscore** (`fr_FR`) alors que BCP 47 utilise un tiret (`fr-FR`). **Nuxt SEO Utils convertit automatiquement**.

```typescript
// i18n.config.ts - Utiliser BCP 47
locales: [
  { code: 'fr', language: 'fr-FR', name: 'Français' },
  { code: 'en', language: 'en-US', name: 'English' },
]
```

## Rendu automatique avec useLocaleHead()

```html
<!-- Généré automatiquement -->
<link rel="alternate" hreflang="fr-FR" href="https://sebc.dev/blog/article">
<link rel="alternate" hreflang="en-US" href="https://sebc.dev/en/blog/article">
<link rel="alternate" hreflang="x-default" href="https://sebc.dev/blog/article">
<meta property="og:locale" content="fr_FR">
<meta property="og:locale:alternate" content="en_US">
```
