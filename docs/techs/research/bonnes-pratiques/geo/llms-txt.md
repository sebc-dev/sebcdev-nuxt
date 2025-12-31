# llms.txt et GEO pour Nuxt 4 SSG : guide complet d'implémentation

La spécification llms.txt, créée par **Jeremy Howard** (co-fondateur d'Answer.AI) en septembre 2024, est devenue un standard émergent pour rendre les sites web facilement consommables par les LLMs. Combinée aux pratiques GEO (Generative Engine Optimization), elle permet d'optimiser la visibilité de votre contenu dans les réponses des moteurs IA comme ChatGPT, Claude et Perplexity. Pour un blog Nuxt 4 SSG sur Cloudflare Pages, le module **nuxt-llms v0.1.3** offre une intégration native avec génération automatique au build time.

## La spécification llms.txt structure l'information pour les LLMs

Le fichier `llms.txt` placé à la racine du site (`/llms.txt`) suit un format Markdown strict. L'objectif est de fournir un "sommaire" optimisé pour les fenêtres de contexte limitées des LLMs, leur permettant de comprendre rapidement le contenu d'un site sans parser des centaines de pages HTML.

La structure obligatoire comprend uniquement le **titre H1**, mais la spécification recommande fortement d'ajouter un blockquote descriptif et des sections organisées :

```markdown
# Nom du Projet

> Description concise en blockquote - résumé clé permettant de comprendre le reste

Informations contextuelles importantes (limitations, compatibilités).
Pas de headings dans cette zone introductive.

## Documentation

- [Guide de démarrage](https://example.com/docs/start.md): Introduction complète
- [Référence API](https://example.com/docs/api.md): Endpoints et paramètres

## Tutorials

- [Premier projet](https://example.com/tutorials/first.md): Tutoriel pas à pas

## Optional

- [Ressources externes](https://example.com/extra.md): Contenu secondaire
```

La section **"Optional"** possède une signification spéciale : les LLMs peuvent l'ignorer si leur fenêtre de contexte est limitée. Les liens doivent pointer vers des fichiers `.md` quand disponibles, car le Markdown est nativement consommable par les modèles.

### llms.txt versus llms-full.txt : deux fichiers complémentaires

| Fichier | Fonction | Taille typique | Usage |
|---------|----------|----------------|-------|
| `llms.txt` | Index de navigation avec liens | ~1,600 mots | Guide rapide, économise les tokens |
| `llms-full.txt` | Contenu complet concaténé | ~58,000 mots | Contexte exhaustif pour LLMs 200K+ |

Le format `llms-full.txt` a été développé par **Mintlify en collaboration avec Anthropic**. Il contient le texte intégral de toutes les pages référencées, idéal pour les modèles avec large contexte comme Claude. Des entreprises majeures ont adopté cette spécification : Anthropic, Cloudflare, Vercel, Stripe, Hugging Face et **plus de 2,000 sites** selon BuiltWith.

## Configuration du module nuxt-llms pour génération automatique

Le module officiel **nuxt-llms** (v0.1.3, maintenu par @atinux et l'équipe Nuxt) génère et pré-rend automatiquement les fichiers llms.txt au build time. Il s'intègre nativement avec Nuxt Content 3.2+ pour extraire le contenu Markdown.

Installation et configuration de base :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/content', 'nuxt-llms'],
  
  llms: {
    domain: 'https://votre-blog.com',  // REQUIS - URL complète
    title: 'Mon Blog Nuxt',
    description: 'Articles techniques sur le développement web moderne',
    
    sections: [
      {
        title: 'Articles',
        description: 'Tous les articles du blog',
        contentCollection: 'blog',  // Collection Nuxt Content
        contentFilters: [
          { field: 'draft', operator: '<>', value: true }
        ]
      },
      {
        title: 'Optional',
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
  },
  
  // Configuration SSG Cloudflare Pages
  nitro: {
    preset: 'cloudflare-pages',
    prerender: {
      crawlLinks: true,
      routes: ['/llms.txt', '/llms-full.txt', '/sitemap.xml']
    }
  }
})
```

Quand Nuxt Content détecte nuxt-llms, il **injecte automatiquement** toutes les collections de type `page` dans les fichiers générés. Les hooks Nitro permettent d'étendre dynamiquement le contenu :

```typescript
// server/plugins/llms-extend.ts
export default defineNitroPlugin((nitroApp) => {
  nitroApp.hooks.hook('llms:generate', (event, options) => {
    options.sections.push({
      title: 'API Documentation',
      links: [
        { title: 'Endpoints', href: '/api/docs', description: 'REST API' }
      ]
    })
  })
})
```

## GEO optimise la visibilité dans les réponses des moteurs IA

Le Generative Engine Optimization représente l'évolution du SEO pour l'ère de l'intelligence artificielle. Contrairement au SEO traditionnel qui vise les 10 liens bleus de Google, le GEO optimise le **taux de citation** dans les réponses synthétisées par les LLMs. Une étude Princeton/Georgia Tech a démontré que certaines stratégies peuvent augmenter la visibilité AI de **+40%**.

Les trois stratégies les plus efficaces selon les recherches :

| Stratégie | Impact mesuré | Implémentation |
|-----------|---------------|----------------|
| **Citation Addition** | +115% visibilité | Référencer des sources autoritatives |
| **Statistics Addition** | +22-37% | Une statistique tous les 150-200 mots |
| **Quotation Addition** | Élevé | Intégrer des citations d'experts |

La structure de contenu idéale place la **réponse directe dans les 40-60 premiers mots**, suivie d'une hiérarchie claire H2/H3. Les LLMs privilégient les listes à puces, tableaux comparatifs et FAQ pour l'extraction d'information. Chaque passage doit être **autonome et compréhensible** sans le contexte environnant.

### Schema.org et métadonnées critiques pour les crawlers AI

Les données structurées Schema.org restent essentielles pour le GEO. Les schemas prioritaires incluent `Organization` (homepage), `Article` (blog posts), `FAQPage` (questions fréquentes) et `HowTo` (tutoriels). L'implémentation JSON-LD est préférée :

```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Guide llms.txt pour Nuxt",
  "datePublished": "2025-12-29",
  "dateModified": "2025-12-29",
  "author": {
    "@type": "Person",
    "name": "Votre Nom"
  }
}
```

### Configuration robots.txt pour les crawlers AI

Les principaux crawlers AI respectent robots.txt. Une configuration équilibrée autorise l'indexation tout en contrôlant l'entraînement :

```
# Autoriser les crawlers de recherche AI (citations)
User-agent: OAI-SearchBot
User-agent: ChatGPT-User
User-agent: ClaudeBot
User-agent: PerplexityBot
Allow: /

# Optionnel: bloquer l'entraînement des modèles
User-agent: GPTBot
User-agent: anthropic-ai
User-agent: Google-Extended
Disallow: /

Sitemap: https://votre-blog.com/sitemap.xml
```

Les user-agents à connaître : **GPTBot** (entraînement OpenAI), **OAI-SearchBot** (ChatGPT Search), **ClaudeBot** (citations Claude), **PerplexityBot** (index Perplexity), et **Google-Extended** (contrôle Gemini).

## Déploiement sur Cloudflare Pages avec headers optimisés

Pour un SSG complet, Cloudflare Pages servira les fichiers pré-rendus depuis le CDN edge. La configuration des headers garantit le bon Content-Type :

```
# public/_headers
/llms.txt
  Content-Type: text/plain; charset=utf-8
  Cache-Control: public, max-age=3600, s-maxage=86400
  X-Robots-Tag: noindex

/llms-full.txt
  Content-Type: text/plain; charset=utf-8
  Cache-Control: public, max-age=3600, s-maxage=86400
```

Le build se lance via `nuxt generate`, produisant les fichiers dans `.output/public/`. Sur Cloudflare Pages Dashboard, configurez :
- **Build command** : `nuxt generate`
- **Output directory** : `.output/public`
- **Variables** : `NODE_VERSION=22`, `PNPM_VERSION=10.26`

## Anti-patterns et pièges courants à éviter absolument

L'implémentation llms.txt comporte plusieurs écueils fréquents qui compromettent son efficacité.

**Erreurs de structure** : Omettre le H1 titre rend le fichier invalide. Oublier le blockquote descriptif prive les LLMs du contexte nécessaire. Lier vers des URLs cassées ou du contenu derrière authentification génère des échecs de crawl.

**Erreurs de configuration nuxt-llms** : L'option `domain` est **obligatoire** - son absence provoque une erreur. Définir `ssr: false` bloque complètement le prerendering des fichiers llms.txt. Utiliser simultanément un fichier statique dans `public/` et nuxt-llms crée des conflits.

**Erreurs Cloudflare Pages** : Oublier d'ajouter `/llms.txt` dans `nitro.prerender.routes` génère des 404. Ne pas configurer le Content-Type dans `_headers` fait servir le fichier en `text/html`. Le cache CDN peut persister après déploiement - purgez-le manuellement si nécessaire.

**Limitations actuelles** : Aucun crawler AI majeur (GPTBot, ClaudeBot, PerplexityBot) ne fetch automatiquement llms.txt pendant l'inférence. L'utilisation principale reste les **assistants de code** (Cursor, Claude Code) et les **serveurs MCP** qui chargent explicitement ces fichiers. La spécification reste utile pour préparer l'adoption future et fournir une documentation optimisée pour le copier-coller dans les prompts.

## Checklist de validation avant mise en production

Avant de déployer, vérifiez systématiquement ces points critiques :

**Configuration nuxt-llms** : module déclaré dans `nuxt.config.ts`, `domain` avec URL complète HTTPS, `ssr: true` (défaut), Nuxt Content 3.2+ installé si intégration souhaitée.

**Validation du fichier généré** : commence par `# Titre`, blockquote `>` présent après le titre, tous les liens fonctionnels et accessibles publiquement, taille inférieure à 500KB, format Markdown valide.

**Tests post-déploiement** : `curl -I https://votre-site.com/llms.txt` retourne HTTP 200 avec `Content-Type: text/plain; charset=utf-8`, contenu conforme aux attentes, headers de cache appliqués.

## Conclusion

L'implémentation de llms.txt via nuxt-llms offre une solution élégante et automatisée pour les blogs Nuxt 4 SSG. La spécification, bien que non encore exploitée automatiquement par les crawlers AI en production, prépare l'avenir et améliore immédiatement l'expérience des développeurs utilisant des assistants de code. Combinée aux pratiques GEO - statistiques régulières, citations autoritatives, données structurées Schema.org et configuration robots.txt appropriée - cette approche maximise les chances d'être cité dans les réponses des moteurs IA. L'investissement technique reste minimal : une configuration nuxt.config.ts de quelques lignes et un fichier `_headers` suffisent pour un déploiement Cloudflare Pages fonctionnel.