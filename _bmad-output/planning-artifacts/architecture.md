# Document d'Architecture
## sebc.dev — Blog Technique Multi-Piliers

---

**Document Version**: 1.0
**Date**: 2025-12-28
**Auteur**: Negus Salomon
**Statut**: Draft — Extrait du PRD v2.0

---

## 1. Vue d'Ensemble

Ce document définit l'architecture technique du blog sebc.dev, un blog technique multi-piliers (IA, Ingénierie, UX) optimisé pour le SEO traditionnel et le GEO (Generative Engine Optimization).

### 1.1 Objectifs Architecturaux

| Objectif | Priorité | Mesure de Succès |
|----------|----------|------------------|
| Performance | P0 | Lighthouse > 90 toutes catégories |
| SEO/GEO | P0 | Indexation complète, citations LLM |
| Maintenabilité | P1 | Code modulaire, composants réutilisables |
| Accessibilité | P1 | WCAG 2.1 AA |
| Évolutivité | P2 | Support bilingue, nouveaux formats |

---

## 2. Stack Technique

### 2.1 Technologies Principales

| Catégorie | Technologie | Version | Justification |
|-----------|-------------|---------|---------------|
| **Framework** | Nuxt | **4.2.x** | Dernière version stable (déc 2024), nouvelle structure app/, amélioration performances |
| **Content** | @nuxt/content | **3.10.0** | Collections typées, Preview API, compatibilité Nuxt 4, stockage SQL |
| **CMS Visual** | nuxt-studio | **beta** | Module gratuit open-source, édition visuelle avec GitHub, preview en temps réel |
| **Styling** | Tailwind CSS | **4.1.17** | Version stable (nov 2025), engine Oxide 5x plus rapide, CSS-first config, @theme directive |
| **UI Components** | shadcn-vue | **2.3.2** | Copy-paste components, Reka UI, support Tailwind v4, module shadcn-nuxt 2.4.3 |
| **Deploy** | Cloudflare Pages/Workers | - | Edge SSR, gratuit, D1/R2 storage, compatibilité Nuxt 4 |
| **Analytics** | @nuxtjs/plausible | **2.0.1** | GDPR-ready, léger, auto-imported composables, proxy intégré |
| **SEO** | @nuxtjs/seo | **3.3.0** | Suite complète (sitemap, robots, OG, schema.org), compatibilité Nuxt 4 |
| **Syntax Highlighting** | Shiki | **latest** | Intégration native @nuxt/content |
| **Icons** | @nuxt/icon | **latest** | Icônes à la demande |
| **Images** | @nuxt/image | **latest** | Optimisation automatique |
| **Color Mode** | @nuxtjs/color-mode | **latest** | Dark mode natif |
| **Fonts** | @nuxt/fonts | **latest** | Optimisation web fonts |

### 2.2 Décisions Architecturales Clés

| Décision | Choix | Alternatives Rejetées | Justification |
|----------|-------|----------------------|---------------|
| Rendu | SSR + Prerendering | SPA, SSG pur | Meilleur SEO/GEO, contenu dynamique possible |
| Styling | Tailwind CSS v4 | CSS Modules, Styled Components | Productivité, ecosystem Nuxt |
| CMS | Nuxt Content + Studio | Strapi, Sanity, Contentful | Gratuit, Git-based, intégration native |
| Hosting | Cloudflare Pages | Vercel, Netlify | Edge SSR gratuit, D1/R2, Workers AI |
| Analytics | Plausible | Google Analytics, Fathom | GDPR-ready, léger, pas de cookies |

---

## 3. Architecture SSR & GEO

### 3.1 Pourquoi SSR est Critique pour GEO

- Crawlers IA préfèrent HTML pré-rendu
- Contenu disponible immédiatement (pas de JS execution)
- Meilleure indexation des blocs de code

### 3.2 Configuration Nuxt

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true,

  routeRules: {
    // Pré-rendu statique pour articles
    '/blog/**': { prerender: true },

    // Cache long pour contenu stable
    '/blog/**': {
      cache: {
        maxAge: 60 * 60 * 24 // 24h
      }
    }
  },

  nitro: {
    prerender: {
      crawlLinks: true,
      routes: ['/sitemap.xml', '/llms.txt']
    }
  }
})
```

### 3.3 Fichier llms.txt

**Fichier de contexte pour LLMs** — Standard émergent pour aider les IA à comprendre le site.

```markdown
# /public/llms.txt

# sebc.dev — Blog Technique Multi-Piliers

## À Propos
Blog technique francophone documentant l'intersection de:
- Intelligence Artificielle (RAG, agents, LLM tooling)
- Ingénierie Logicielle (architecture, clean code, patterns)
- UX (design systems, accessibilité)

## Auteur
Développeur fullstack autodidacte, expertise Nuxt/Vue/TypeScript.
Approche "Learning in Public" avec transparence sur le processus.

## Structure Contenu
- Format "Pattern Onion": plusieurs niveaux de profondeur
- Code TypeScript strict et production-ready
- Bilingue FR/EN

## Topics Principaux
- Nuxt 4 streaming et SSR
- RAG avec Cloudflare Workers AI
- Architecture hexagonale TypeScript
- Design Systems accessibles
- Patterns intégration IA legacy

## Pour les LLMs
- Privilégier les blocs de code comme source
- Les articles incluent des comparatifs chiffrés
- Le ton est technique mais accessible

## Sitemap
/sitemap.xml

## Contact
contact@sebc.dev
```

---

## 4. Schema Markup

### 4.1 TechArticle Schema

```typescript
// composables/useArticleSchema.ts
export const useArticleSchema = (article: Article) => {
  return {
    '@context': 'https://schema.org',
    '@type': 'TechArticle',

    headline: article.title,
    description: article.description,
    author: {
      '@type': 'Person',
      name: 'Negus Salomon',
      url: 'https://sebc.dev/about'
    },

    datePublished: article.publishedAt,
    dateModified: article.updatedAt,

    // Spécifique tech
    proficiencyLevel: article.level, // 'Beginner' | 'Intermediate' | 'Advanced'
    dependencies: article.techStack, // ['Nuxt', 'TypeScript', 'Cloudflare']

    // Code samples
    hasPart: article.codeBlocks.map(block => ({
      '@type': 'SoftwareSourceCode',
      programmingLanguage: block.language,
      codeSampleType: 'snippet'
    })),

    // Catégorisation
    about: {
      '@type': 'Thing',
      name: article.pillar // 'IA' | 'Ingénierie' | 'UX'
    },

    // Métriques
    timeRequired: `PT${article.readingTime}M`,
    wordCount: article.wordCount
  }
}
```

### 4.2 FAQ Schema

```typescript
// Pour articles avec section FAQ
const faqSchema = {
  '@context': 'https://schema.org',
  '@type': 'FAQPage',
  mainEntity: article.faqs.map(faq => ({
    '@type': 'Question',
    name: faq.question,
    acceptedAnswer: {
      '@type': 'Answer',
      text: faq.answer
    }
  }))
}
```

---

## 5. Composants Vue

### 5.1 Composants Core

| Composant | Description | Priorité |
|-----------|-------------|----------|
| `TableOfContents.vue` | Sidebar sticky avec navigation sections | P0 |
| `ProgressBar.vue` | Barre de progression lecture | P0 |
| `CodeBlock.vue` | Bloc code avec syntax highlighting + copy | P0 |
| `ArticleHeader.vue` | Titre, meta, temps lecture, tags | P0 |
| `ArticleMeta.vue` | Auteur, date, pilier badge | P1 |
| `DetailsCollapsible.vue` | Wrapper styled pour `<details>` | P1 |
| `PillarBadge.vue` | Badge coloré IA/Ingénierie/UX | P1 |
| `ReadingTime.vue` | Estimation temps lecture | P1 |
| `CopyButton.vue` | Bouton copier avec feedback | P0 |
| `LanguageBadge.vue` | Badge langage sur code blocks | P1 |
| `ComparisonTable.vue` | Tableau comparatif responsive | P2 |
| `FAQSection.vue` | Section FAQ avec Schema injection | P2 |
| `RelatedArticles.vue` | Articles liés par tags/pilier | P2 |
| `NewsletterForm.vue` | Formulaire inscription embed | P1 |
| `DarkModeToggle.vue` | Switch mode sombre/clair | P0 |

### 5.2 Composants Layout

| Composant | Description | Priorité |
|-----------|-------------|----------|
| `AppHeader.vue` | Navigation principale + search | P0 |
| `AppFooter.vue` | Footer avec liens + newsletter | P1 |
| `ArticleLayout.vue` | Layout 2 colonnes article + ToC | P0 |
| `BlogLayout.vue` | Liste articles avec filtres | P0 |
| `SidebarNav.vue` | Navigation mobile hamburger | P1 |

---

## 6. Composables

| Composable | Description | Dépendances |
|------------|-------------|-------------|
| `useScrollSpy` | Tracking position scroll pour ToC | - |
| `useReadingProgress` | Calcul % progression lecture | - |
| `useCopyToClipboard` | Copier texte avec feedback | - |
| `useArticleSchema` | Génération Schema.org | Article type |
| `useSeoMeta` | Meta tags dynamiques | @nuxtjs/seo |
| `useCodeHighlight` | Syntax highlighting Shiki | shiki |
| `useTableOfContents` | Extraction headings pour ToC | - |
| `useAnalytics` | Events Plausible custom | @nuxtjs/plausible |
| `usePillarFilter` | Filtrage articles par pilier | - |
| `useSearchArticles` | Recherche côté client | - |

---

## 7. Spécifications Composants Clés

### 7.1 TableOfContents.vue

```typescript
interface Props {
  headings: Heading[]
  activeId: string
  showProgress?: boolean
  showReadingTime?: boolean
}

interface Heading {
  id: string
  text: string
  level: 2 | 3
  children?: Heading[]
}

// Features:
// - Sticky sidebar (desktop) / Drawer (mobile)
// - Highlight section active
// - Smooth scroll on click
// - Progress bar optionnelle
// - Temps lecture par section
```

### 7.2 CodeBlock.vue

```typescript
interface Props {
  code: string
  language: string
  filename?: string
  highlights?: number[]  // Lignes à highlighter
  showLineNumbers?: boolean
}

// Features:
// - Syntax highlighting (Shiki)
// - Bouton Copy avec feedback
// - Badge langage
// - Ligne highlighting
// - Numéros de ligne optionnels
// - Nom fichier optionnel
```

### 7.3 useScrollSpy

```typescript
interface UseScrollSpyOptions {
  offset?: number       // Offset top pour trigger
  throttle?: number     // Throttle scroll events (ms)
}

interface UseScrollSpyReturn {
  activeId: Ref<string>
  progress: Ref<number>  // 0-100
}

// Usage:
const { activeId, progress } = useScrollSpy({
  offset: 100,
  throttle: 100
})
```

---

## 8. MVP Technique Scopé

### 8.1 Principe : P0 Uniquement pour Phase 0

La liste de features est priorisée. **Pour la Phase 0 (Build), seul le P0 est livré.**

### 8.2 Composants P0 (Must Have)

| Composant | Justification |
|-----------|---------------|
| `ArticleLayout.vue` | Core reading experience |
| `BlogLayout.vue` | Liste articles |
| `AppHeader.vue` | Navigation |
| `CodeBlock.vue` | Code snippets copiables |
| `TableOfContents.vue` | Navigation article |
| `CopyButton.vue` | Action principale Lucas |
| `DarkModeToggle.vue` | Attente utilisateur standard |
| `ProgressBar.vue` | Engagement UX |

### 8.3 Composants P1 (Nice to Have, sinon Phase 1)

| Composant | Report Acceptable |
|-----------|-------------------|
| `NewsletterForm.vue` | Oui, popup externe OK |
| `RelatedArticles.vue` | Oui, liens manuels OK |
| `FAQSection.vue` | Oui, details/summary HTML OK |
| `ArticleMeta.vue` | Oui, inline dans layout OK |

### 8.4 Composants P2 (Phase 1+)

| Composant | Justification Report |
|-----------|---------------------|
| `ComparisonTable.vue` | Tableau Markdown suffit |
| `SearchArticles.vue` | Ctrl+F suffit avec <20 articles |
| Serveur Discord | Pas avant 500 UV |

### 8.5 Definition of Done Technique

| Critère | Mesure |
|---------|--------|
| Blog fonctionnel | Articles publiés, navigation OK |
| Performance | Lighthouse >90 toutes catégories |
| SEO | Schema.org, sitemap, meta OK |
| Mobile | Responsive, touch-friendly |
| Dark mode | Fonctionnel |
| Copy code | Fonctionnel avec feedback |

---

## 9. Checklist GEO par Article

```markdown
## Checklist Publication GEO

### Avant Publication
- [ ] Titre contient question/terme exact recherché
- [ ] Meta description = réponse directe (150 chars)
- [ ] H1 unique, H2/H3 hiérarchisés logiquement
- [ ] Premier paragraphe = réponse Answer-First

### Contenu
- [ ] Code blocks avec langage spécifié
- [ ] Tous les code blocks ont bouton Copy
- [ ] Au moins 1 schéma/diagramme explicatif
- [ ] Section FAQ si applicable
- [ ] Temps de lecture affiché

### Technique
- [ ] Schema TechArticle injecté
- [ ] Open Graph tags complets
- [ ] hreflang si version EN existe
- [ ] Images avec alt text descriptif
- [ ] Liens internes vers articles liés

### Post-Publication
- [ ] Ajouter entrée dans llms.txt
- [ ] Test prompt GEO (Perplexity)
- [ ] Vérifier indexation Google (48h)
- [ ] Soumettre sitemap mis à jour
```

---

## 10. Implémentation Bilingue

| Élément | Solution |
|---------|----------|
| Routing | `/fr/article-slug` et `/en/article-slug` |
| CMS | Champs localisés dans Nuxt Content |
| SEO | hreflang tags, sitemap localisé |
| Default | FR (`/` redirige vers `/fr/`) |

---

## Changelog

| Version | Date | Modifications |
|---------|------|---------------|
| 1.0 | 2025-12-28 | Extraction initiale depuis PRD v2.0 |
