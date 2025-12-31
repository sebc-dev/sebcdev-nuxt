# SEO & GEO Implementation

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Suite SEO** | @nuxtjs/seo (bundle) | Inclut sitemap, robots, schema.org, og-image, link-checker |
| **OG Images** | nuxt-og-image (zeroRuntime) | Génération au build via Satori, 100% SSG, CF Pages gratuit |
| **llms.txt** | Module nuxt-llms (auto) | Navigation IA efficace, génération automatique avec @nuxt/content ^3.2.0 |
| **RSS** | Server route generation | Native Content 3 integration |
| **i18n SEO** | useLocaleHead() | Injection auto hreflang + og:locale avec `language` obligatoire |

**Sous-modules inclus dans @nuxtjs/seo :**
- `@nuxtjs/sitemap` - Sitemap XML avec hreflang i18n auto
- `@nuxtjs/robots` - robots.txt dynamique
- `nuxt-schema-org` - JSON-LD Schema.org
- `nuxt-og-image` - Génération images OG au build
- `nuxt-link-checker` - Validation liens (dev)
- `nuxt-seo-utils` - Utilitaires (useSiteConfig, useLocaleHead...)

**Optimisations GEO (Princeton/Georgia Tech):**

| Stratégie | Impact mesuré | Implémentation |
|-----------|---------------|----------------|
| **Citation Addition** | +115% visibilité | Référencer des sources autoritatives |
| **Statistics Addition** | +22-37% | Une statistique tous les 150-200 mots |
| **Quotation Addition** | Élevé | Intégrer des citations d'experts |

**Structure contenu GEO :**
- Réponse directe dans les **40-60 premiers mots**
- Hiérarchie claire H2/H3
- Listes à puces et tableaux comparatifs
- Passages **autonomes et compréhensibles** sans contexte

**Configuration nuxt-llms détaillée :**

```typescript
// nuxt.config.ts
llms: {
  domain: 'https://sebc.dev',  // REQUIS - URL complète
  title: 'sebc.dev',
  description: 'Blog technique sur le développement web moderne',

  sections: [
    {
      title: 'Articles',
      description: 'Tous les articles du blog',
      contentCollection: 'articles_fr',  // Collection Nuxt Content
      contentFilters: [
        { field: 'draft', operator: '<>', value: true }
      ]
    },
    {
      title: 'Optional',  // Section ignorée si contexte LLM limité
      links: [
        { title: 'À propos', href: '/about', description: 'Qui suis-je' }
      ]
    }
  ],

  // Activer llms-full.txt (désactivé par défaut)
  full: {
    title: 'Documentation Complète',
    description: 'Tout le contenu du blog en un fichier'
  }
}
```

**llms.txt vs llms-full.txt :**

| Fichier | Fonction | Taille typique | Usage |
|---------|----------|----------------|-------|
| `llms.txt` | Index avec liens | ~1,600 mots | Guide rapide, économise tokens |
| `llms-full.txt` | Contenu complet | ~58,000 mots | Contexte exhaustif (Claude 200K+) |

**Extension dynamique via hooks Nitro :**

```typescript
// server/plugins/llms-extend.ts
export default defineNitroPlugin((nitroApp) => {
  // Hook pour /llms.txt
  nitroApp.hooks.hook('llms:generate', (event, options) => {
    options.sections.push({
      title: 'API Documentation',
      links: [
        { title: 'Endpoints', href: '/api/docs', description: 'REST API' }
      ]
    })
  })

  // Hook pour /llms-full.txt (contenu détaillé)
  nitroApp.hooks.hook('llms:generate:full', (event, options, contents) => {
    contents.push(`
# Informations techniques

## Stack technologique
- **Framework**: Nuxt 4.2.x avec Vue 3.5
- **CMS**: Nuxt Content 3 avec SQLite/D1
- **Déploiement**: Cloudflare Pages (SSG)
- **Langues**: Français (fr), English (en)

## Formats de contenu
Tous les articles sont disponibles en Markdown avec:
- Blocs de code avec coloration syntaxique Shiki
- Métadonnées Schema.org structurées
    `)
  })
})
```

**Plugin i18n multilingue pour nuxt-llms :**

```typescript
// server/plugins/llms-i18n.ts
export default defineNitroPlugin((nitroApp) => {
  nitroApp.hooks.hook('llms:generate', (event, options) => {
    const locale = detectLocaleFromEvent(event)

    const config = {
      en: {
        title: 'sebc.dev - Technical Blog',
        description: 'Technical articles on modern web development with Nuxt and Vue.js.',
        contentCollection: 'articles_en'
      },
      fr: {
        title: 'sebc.dev - Blog Technique',
        description: 'Articles techniques sur le développement web moderne avec Nuxt et Vue.js.',
        contentCollection: 'articles_fr'
      }
    }

    const localeConfig = config[locale] || config.en
    Object.assign(options, localeConfig)

    // Mettre à jour les sections avec la bonne collection
    options.sections = options.sections.map(section => ({
      ...section,
      contentCollection: section.contentCollection
        ? localeConfig.contentCollection
        : section.contentCollection
    }))
  })
})

function detectLocaleFromEvent(event: H3Event): 'en' | 'fr' {
  const acceptLanguage = getHeader(event, 'accept-language') || ''
  return acceptLanguage.startsWith('fr') ? 'fr' : 'en'
}
```

**Blockquote GEO enrichi (stats auteur) :**

Le blockquote llms.txt doit inclure des métriques concrètes pour renforcer l'autorité :

```markdown
# sebc.dev - Blog Technique

> Blog spécialisé développement Nuxt.js et Vue.js. 30+ articles techniques
> publiés depuis 2023. Mise à jour hebdomadaire. Auteur: développeur senior
> avec 10 ans d'expérience Vue.js, contributeur open source Nuxt.

Ce blog couvre les patterns avancés Nuxt 4, l'optimisation performance,
et les architectures SSG. Chaque article inclut des exemples testés.
```

**Statistiques trafic IA 2025 :**
- Trafic référé IA : **+527%** entre janvier et mai 2025
- Taux de conversion visiteurs IA : **4,4x supérieur** au trafic organique
- Source : Études BrightEdge et Search Engine Land

**Anti-patterns nuxt-llms à éviter :**

| Erreur | Conséquence | Solution |
|--------|-------------|----------|
| `domain` absent | Erreur build | Toujours définir l'URL complète HTTPS |
| `ssr: false` | Fichiers non générés | Garder `ssr: true` (défaut) |
| Fichier dans `public/llms.txt` | Conflit avec module | Supprimer le fichier statique |
| Liens vers pages authentifiées | Échec crawl AI | Liens publics uniquement |
| Ordre modules incorrect | Intégration i18n cassée | `@nuxtjs/i18n` avant `@nuxt/content` |
| contentFilters trop restrictifs | Aucun contenu généré | Tester filtres avec `queryCollection()` d'abord |
