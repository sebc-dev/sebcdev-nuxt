# Intégration Frontmatter avec asSeoCollection()

Le wrapper `asSeoCollection()` active les clés frontmatter SEO dans les fichiers Markdown :

```yaml
---
title: "Mon article"
description: "Description SEO (max 160 caractères)"
image: "/images/cover.jpg"

# Overrides OG optionnels (si différents du titre/description)
ogTitle: "Titre optimisé pour les réseaux sociaux"
ogDescription: "Description plus courte pour OG"

# Schema.org structuré (optionnel)
schemaOrg:
  - "@type": "BlogPosting"
    headline: "Mon article"
    author:
      "@type": "Person"
      name: "Sébastien C."
    datePublished: "2025-01-15"

# Contrôle sitemap (optionnel)
sitemap:
  lastmod: 2025-01-16
  priority: 0.8

# Contrôle robots (optionnel)
robots:
  noindex: false
  nofollow: false
---
```
