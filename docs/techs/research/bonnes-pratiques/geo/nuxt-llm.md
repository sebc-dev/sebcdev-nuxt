# nuxt-llms et GEO : guide technique complet pour blogs Nuxt 4 multilingues

L'optimisation pour les moteurs IA représente un nouveau paradigme distinct du SEO traditionnel. Les recherches de Princeton/Georgia Tech démontrent que les techniques GEO peuvent **augmenter la visibilité dans les réponses IA de 30 à 40%**, tandis que le module nuxt-llms (v0.1.3+) offre une intégration native avec Nuxt Content 3 pour générer automatiquement les fichiers llms.txt conformes à la spécification. Ce guide couvre l'implémentation complète pour un stack Nuxt 4.2.x avec SSG Cloudflare Pages et internationalisation.

## Le module nuxt-llms automatise la génération de documentation pour LLMs

Le module **nuxt-llms** (maintenu par nuxt-content sur GitHub) génère automatiquement des fichiers `/llms.txt` et `/llms-full.txt` suivant la spécification proposée par Jeremy Howard en septembre 2024. La version **0.1.3** (juin 2025) corrige notamment le support des descriptions personnalisées via les hooks.

### Configuration de base dans nuxt.config.ts

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  future: { compatibilityVersion: 4 },
  compatibilityDate: '2024-12-01',
  
  modules: [
    '@nuxtjs/i18n',
    '@nuxt/content',
    'nuxt-llms',
    'nuxt-schema-org'
  ],
  
  llms: {
    domain: 'https://example.com',
    title: 'Mon Blog Technique',
    description: 'Articles techniques sur le développement web moderne, Nuxt, Vue.js et les bonnes pratiques.',
    
    sections: [
      // Section avec collection Content 3
      {
        title: 'Articles',
        description: 'Guides techniques et tutoriels',
        contentCollection: 'blog',
        contentFilters: [
          { field: 'extension', operator: '=', value: 'md' },
          { field: 'draft', operator: '<>', value: true },
        ]
      },
      // Section manuelle
      {
        title: 'Ressources',
        links: [
          { title: 'À propos', description: 'Présentation du blog', href: '/about' },
          { title: 'Contact', href: '/contact' }
        ]
      },
      // Section optionnelle (peut être ignorée par les LLMs)
      {
        title: 'Optional',
        links: [
          { title: 'Changelog', href: '/changelog' }
        ]
      }
    ],
    
    notes: 'Documentation générée automatiquement. Dernière mise à jour: 2025.',
    
    // Activer llms-full.txt
    full: {
      title: 'Documentation Complète',
      description: 'Contenu intégral de tous les articles du blog',
    }
  }
})
```

### Les hooks Nitro permettent la génération dynamique

Les hooks `llms:generate` et `llms:generate:full` s'exécutent à chaque requête sur les endpoints respectifs, permettant d'ajouter du contenu dynamiquement :

```typescript
// server/plugins/llms-multilang.ts
export default defineNitroPlugin((nitroApp) => {
  // Hook pour /llms.txt
  nitroApp.hooks.hook('llms:generate', (event, options) => {
    // Détecter la locale depuis l'URL ou headers
    const locale = event.path.startsWith('/fr') ? 'fr' : 'en'
    
    // Modifier le titre selon la langue
    options.title = locale === 'fr' 
      ? 'Mon Blog Technique' 
      : 'My Technical Blog'
    
    // Ajouter une section dynamique
    options.sections.push({
      title: locale === 'fr' ? 'API Publique' : 'Public API',
      description: locale === 'fr' 
        ? 'Documentation des endpoints REST' 
        : 'REST endpoints documentation',
      links: [
        {
          title: 'OpenAPI Spec',
          description: 'Spécification OpenAPI 3.0',
          href: `${options.domain}/api/openapi.json`
        }
      ]
    })
  })
  
  // Hook pour /llms-full.txt
  nitroApp.hooks.hook('llms:generate:full', (event, options, contents) => {
    // Ajouter du contenu markdown détaillé
    contents.push(`
## Informations techniques

### Stack technologique
- **Framework**: Nuxt 4.2.x avec Vue 3
- **CMS**: Nuxt Content 3 avec SQLite
- **Déploiement**: Cloudflare Pages (SSG)
- **Langues**: Français (fr), English (en)

### Formats de contenu
Tous les articles sont disponibles en Markdown enrichi avec:
- Blocs de code avec coloration syntaxique
- Diagrammes Mermaid
- Métadonnées Schema.org structurées
    `)
  })
})
```

## La spécification llms.txt définit un format Markdown structuré

La spécification officielle (llmstxt.org, version 0.0.4) impose un format Markdown précis conçu pour être lu directement par les LLMs tout en restant parseable programmatiquement.

### Structure canonique du fichier

```markdown
# Nom du Projet

> Description concise avec les informations clés nécessaires à la compréhension

Notes importantes et contexte additionnel en texte libre.
Peut inclure plusieurs paragraphes.

## Documentation

- [Guide de démarrage](https://example.com/docs/start): Introduction rapide aux concepts
- [Référence API](https://example.com/docs/api): Documentation complète des endpoints

## Tutoriels

- [Premier projet](https://example.com/tutorials/first): Création pas à pas
- [Cas avancés](https://example.com/tutorials/advanced): Patterns complexes

## Optional

- [Changelog](https://example.com/changelog)
- [Contributions](https://example.com/contributing)
```

### Différences clés entre llms.txt et llms-full.txt

| Caractéristique | llms.txt | llms-full.txt |
|-----------------|----------|---------------|
| **Taille typique** | ~1 600 mots | ~58 000+ mots |
| **Contenu** | Liens annotés (sitemap intelligent) | Texte intégral de la documentation |
| **Usage** | Navigation, découverte de structure | Contexte complet sans fetch additionnel |
| **Analogue** | Table des matières | Encyclopédie complète |

La section `## Optional` a une signification spéciale : les URLs qu'elle contient peuvent être ignorées par les LLMs quand le contexte est limité. Cette section est idéale pour le changelog, les contributions, ou le contenu secondaire.

## L'étude Princeton/Georgia Tech valide l'efficacité du GEO

L'étude "GEO: Generative Engine Optimization" publiée à **KDD 2024** par des chercheurs de Princeton, Georgia Tech, Allen Institute for AI et IIT Delhi constitue la référence académique sur l'optimisation pour moteurs génératifs. Les chercheurs ont créé **GEO-bench**, un benchmark de 10 000 requêtes diverses, et testé 9 méthodes d'optimisation.

### Les trois techniques les plus efficaces

Les méthodes **Quotation Addition** (ajout de citations), **Statistics Addition** (ajout de statistiques), et **Cite Sources** (citation de sources) ont démontré des améliorations de **30-40% sur la métrique Position-Adjusted Word Count** :

- **Quotation Addition** : 27,3% de score absolu vs 19,3% baseline (+41% relatif)
- **Statistics Addition** : 25,2% de score (+30% relatif)  
- **Cite Sources** : 24,6% de score (+27% relatif)

Ces résultats ont été **validés sur Perplexity.ai** avec des améliorations similaires. Fait crucial : le **keyword stuffing traditionnel performe 10% moins bien** que le baseline sur Perplexity, démontrant que les techniques SEO classiques peuvent nuire en contexte GEO.

### Métriques GEO à optimiser

**Fact Density** (densité factuelle) : Fréquence de données vérifiables, statistiques et chiffres. La recommandation est d'inclure **une statistique tous les 150-200 mots**. Le contenu avec statistiques montre 41% de visibilité IA supplémentaire.

**Answer Capsules** (capsules de réponse) : Résumés concis de **40-60 mots** répondant directement aux requêtes utilisateurs dans un format facilement extractible. Le contenu avec answer capsules est **40% plus susceptible d'être cité** selon un audit BrightEdge de 2 millions de sessions ChatGPT.

**Extractible Content** : Contenu structuré permettant l'identification facile de segments citables — hiérarchie claire (H1→H2→H3), titres en format question, listes à puces, tableaux de comparaison, sections FAQ avec markup schema.

## Configuration Content 3 avec collections multilingues

Pour un blog multilingue avec Nuxt Content 3.10.0+ et SQLite, la structure recommandée utilise des collections séparées par langue :

```typescript
// content.config.ts
import { defineContentConfig, defineCollection, z } from '@nuxt/content'
import { asSchemaOrgCollection } from 'nuxt-schema-org/content'

const blogSchema = z.object({
  title: z.string(),
  description: z.string().min(50).max(160), // Optimisé pour les answer capsules
  date: z.string(),
  author: z.string(),
  tags: z.array(z.string()).optional(),
  // Métadonnées GEO
  factDensity: z.number().optional(), // Score de densité factuelle
  lastReviewed: z.string().optional(), // Fraîcheur du contenu
})

export default defineContentConfig({
  collections: {
    blog_en: defineCollection(
      asSchemaOrgCollection({
        type: 'page',
        source: { include: 'en/blog/**/*.md', prefix: '/blog' },
        schema: blogSchema,
      })
    ),
    blog_fr: defineCollection(
      asSchemaOrgCollection({
        type: 'page',
        source: { include: 'fr/blog/**/*.md', prefix: '/blog' },
        schema: blogSchema,
      })
    ),
  },
})
```

### Structure de fichiers pour blog multilingue

```
content/
├── en/
│   └── blog/
│       ├── getting-started.md
│       └── advanced-patterns.md
└── fr/
    └── blog/
        ├── premiers-pas.md
        └── patterns-avances.md
```

### Configuration i18n intégrée

```typescript
// nuxt.config.ts (suite)
export default defineNuxtConfig({
  i18n: {
    locales: [
      { code: 'en', language: 'en-US', name: 'English' },
      { code: 'fr', language: 'fr-FR', name: 'Français' },
    ],
    defaultLocale: 'en',
    strategy: 'prefix_except_default', // /blog (en), /fr/blog (fr)
    detectBrowserLanguage: {
      useCookie: true,
      cookieKey: 'i18n_locale',
      redirectOn: 'root',
    },
  },
  
  // SSG Cloudflare Pages
  nitro: {
    preset: 'cloudflare-pages',
    prerender: {
      crawlLinks: true,
      routes: ['/', '/fr', '/llms.txt', '/llms-full.txt'],
      failOnError: false,
    },
  },
  
  site: {
    url: 'https://example.com',
    name: 'Mon Blog Technique',
  },
})
```

## Implémentation d'une structure llms.txt multilingue optimale

Pour un blog bilingue, deux approches sont possibles : fichiers llms.txt séparés par locale ou fichier unifié avec sections par langue.

### Approche recommandée : fichiers séparés via hooks

```typescript
// server/plugins/llms-i18n.ts
export default defineNitroPlugin((nitroApp) => {
  nitroApp.hooks.hook('llms:generate', async (event, options) => {
    const locale = detectLocaleFromEvent(event) // 'en' ou 'fr'
    
    // Configuration par langue
    const config = {
      en: {
        title: 'My Technical Blog',
        description: 'Technical articles on modern web development with Nuxt, Vue.js, and best practices. Content optimized for AI assistants.',
        sections: [
          {
            title: 'Latest Articles',
            description: 'Recent technical guides and tutorials',
            contentCollection: 'blog_en',
            contentFilters: [
              { field: 'draft', operator: '<>', value: true }
            ]
          }
        ]
      },
      fr: {
        title: 'Mon Blog Technique',
        description: 'Articles techniques sur le développement web moderne avec Nuxt, Vue.js et les bonnes pratiques. Contenu optimisé pour les assistants IA.',
        sections: [
          {
            title: 'Derniers Articles',
            description: 'Guides techniques et tutoriels récents',
            contentCollection: 'blog_fr',
            contentFilters: [
              { field: 'draft', operator: '<>', value: true }
            ]
          }
        ]
      }
    }
    
    const localeConfig = config[locale] || config.en
    Object.assign(options, localeConfig)
  })
})

function detectLocaleFromEvent(event: H3Event): 'en' | 'fr' {
  // Logique de détection (headers, cookies, URL)
  const acceptLanguage = getHeader(event, 'accept-language') || ''
  return acceptLanguage.startsWith('fr') ? 'fr' : 'en'
}
```

### Structure llms.txt optimisée pour GEO

Appliquez les principes GEO directement dans la structure du fichier :

```markdown
# Mon Blog Technique

> Blog spécialisé développement Nuxt.js et Vue.js. 47 articles techniques publiés depuis 2023. Mise à jour hebdomadaire. Auteur: expert certifié Vue.js avec 8 ans d'expérience.

Ce blog couvre les patterns avancés Nuxt 4, l'optimisation performance, et les architectures SSG/SSR. Chaque article inclut des exemples de code testés et des benchmarks quantifiés.

## Guides Essentiels

- [Migration Nuxt 3 vers 4](https://example.com/blog/nuxt4-migration): Guide complet avec checklist 23 points, temps moyen migration 4h
- [Optimisation Core Web Vitals](https://example.com/blog/cwv-nuxt): Techniques ayant amélioré LCP de 40% sur 12 sites audités
- [SSG avec Cloudflare Pages](https://example.com/blog/cloudflare-ssg): Configuration détaillée, temps de build réduit de 60%

## Référence Technique

- [API Nuxt Content 3](https://example.com/blog/content3-api): Documentation complète queryCollection avec 15 exemples
- [Hooks Nitro](https://example.com/blog/nitro-hooks): Liste exhaustive des 24 hooks disponibles

## Optional

- [Changelog](https://example.com/changelog)
- [À propos](https://example.com/about)
```

## Anti-patterns à éviter dans l'implémentation

### Erreurs nuxt-llms courantes

**Ne pas confondre les versions de hooks.** Avant v0.1.0, le module utilisait des hooks Nuxt. Depuis v0.1.0, seuls les hooks Nitro sont supportés. Utilisez `nitroApp.hooks.hook()` et non `nuxt.hooks.hook()`.

**Ne pas oublier d'activer llms-full.txt.** Par défaut, seul `/llms.txt` est généré. Pour activer `/llms-full.txt`, vous devez explicitement configurer `full.title` et `full.description`.

**Éviter les contentFilters trop restrictifs.** Un filtre mal configuré peut exclure tout le contenu. Testez vos filtres avec `queryCollection()` avant de les appliquer aux sections llms.

### Erreurs GEO à ne pas commettre

**Keyword stuffing.** Les recherches Princeton montrent que cette technique SEO classique **dégrade les performances de 10%** sur Perplexity. Les moteurs génératifs pénalisent la sur-optimisation de mots-clés.

**Contenu sans données quantifiées.** Le contenu purement qualitatif sans statistiques, chiffres ou sources citées performe significativement moins bien. Visez une statistique tous les 150-200 mots.

**Sections trop longues sans structure.** Les LLMs ont besoin de chunks sémantiques autonomes. Chaque section H2 doit être compréhensible indépendamment avec son propre "answer capsule" dans les 40-60 premiers mots.

**Ignorer la fraîcheur du contenu.** Perplexity favorise fortement le contenu de moins de 90 jours. Mettez à jour vos articles régulièrement et indiquez la date de dernière révision.

### Erreurs d'architecture Nuxt 4

**Mauvais ordre des modules.** L'ordre dans `modules[]` importe : chargez `@nuxtjs/i18n` avant `@nuxt/content` pour une intégration correcte.

**Oublier les routes de prérendu.** Pour SSG Cloudflare, ajoutez explicitement `/llms.txt` et `/llms-full.txt` dans `nitro.prerender.routes` pour garantir leur génération.

**Collections mal configurées pour i18n.** Utilisez des collections séparées par langue (`blog_en`, `blog_fr`) plutôt qu'une collection unique avec filtres de langue, pour une meilleure performance et maintenabilité.

## Conclusion : stratégie d'implémentation progressive

L'intégration nuxt-llms avec optimisation GEO requiert une approche en trois phases. **Premièrement**, configurez le module de base avec `contentCollection` pointant vers vos collections Content 3 multilingues — cela génère automatiquement un llms.txt fonctionnel. **Deuxièmement**, implémentez les hooks Nitro pour personnaliser la génération par locale et ajouter du contenu dynamique. **Troisièmement**, appliquez les principes GEO à votre contenu : descriptions riches de 50-160 caractères, statistiques régulières, citations de sources autoritaires, et structure facilitant l'extraction de réponses.

Les données montrent que cette optimisation n'est pas théorique : le trafic référé par IA a augmenté de **527% entre janvier et mai 2025**, et les visiteurs IA convertissent à un taux **4,4 fois supérieur** au trafic organique traditionnel. Le moment d'implémenter ces techniques est maintenant, avant que la concurrence ne s'intensifie dans l'espace GEO.