# Epic 7: SEO & Discovery Optimization

**Objectif :** Le contenu est découvrable par les moteurs de recherche traditionnels et les LLMs.

## Story 7.1: llms.txt for AI Discoverability

As a LLM (AI assistant),
I want to access a structured llms.txt file,
So that I can understand and cite the blog's content accurately.

**Acceptance Criteria:**

**Given** un LLM ou crawler accède à `/llms.txt`
**When** le fichier est servi
**Then** il contient une présentation du site
**And** il liste les articles avec leurs descriptions
**And** il inclut des guidelines pour les citations
**And** le format est conforme à https://llmstxt.org/
**And** le fichier est mis à jour automatiquement à chaque build

**Technical Scope:**
- Server route `server/routes/llms.txt.ts` ou génération statique
- Template llms.txt avec liste d'articles dynamique
- Génération au build time

**FRs:** FR41

---

## Story 7.2: Schema.org Markup (TechArticle & FAQ)

As a moteur de recherche,
I want to parse structured Schema.org data,
So that I can display rich snippets for the blog's content.

**Acceptance Criteria:**

**Given** je visite une page article
**When** je consulte le code source
**Then** je trouve un bloc JSON-LD `TechArticle` avec : headline, author, datePublished, description
**And** si l'article contient une FAQ, un schema `FAQPage` est inclus
**And** la validation sur https://validator.schema.org/ retourne 0 erreur, 0 warning

**Technical Scope:**
- nuxt-schema-org v5.0.10
- Composable useSchemaOrg() ou defineArticle()
- Détection automatique des sections FAQ dans le contenu

**FRs:** FR42
**ARCH:** ARCH-13
**NFRs:** NFR13

---

## Story 7.3: XML Sitemap Generation

As a moteur de recherche,
I want to access a complete XML sitemap,
So that I can discover and index all pages efficiently.

**Acceptance Criteria:**

**Given** je accède à `/sitemap.xml`
**When** le fichier est servi
**Then** il inclut toutes les pages publiques (accueil, articles, pages statiques)
**And** les versions FR et EN sont listées avec hreflang
**And** chaque URL a un lastmod basé sur la date de modification
**And** le sitemap est généré automatiquement

**Technical Scope:**
- @nuxtjs/sitemap v7.5+
- asSitemapCollection() pour Nuxt Content v3
- Configuration multilingue

**FRs:** FR43
**ARCH:** ARCH-12
**NFRs:** NFR11

---

## Story 7.4: Open Graph & Twitter Cards

As a utilisateur partageant un article sur les réseaux sociaux,
I want to see a rich preview with image and description,
So that my shared links are attractive and informative.

**Acceptance Criteria:**

**Given** je partage un article sur Twitter/X, LinkedIn, ou Facebook
**When** la plateforme génère un aperçu
**Then** le titre de l'article est affiché (og:title)
**And** la description est affichée (og:description)
**And** une image est affichée (og:image) - générée dynamiquement ou image de couverture
**And** l'URL est correcte (og:url)
**And** le type est "article" (og:type)
**And** Twitter Card est de type "summary_large_image"

**Technical Scope:**
- useSeoMeta() ou useHead() pour meta tags
- Nuxt OG Image pour génération dynamique d'images (optionnel)
- Template OG image avec titre et thème

**FRs:** FR44

---

## Story 7.5: RSS Feeds

As a lecteur utilisant un agrégateur RSS,
I want to subscribe to the blog's RSS feed,
So that I receive new articles automatically.

**Acceptance Criteria:**

**Given** je accède à `/rss.xml` ou `/feed.xml`
**When** le flux est servi
**Then** il contient les 20 derniers articles
**And** chaque item a : titre, description, lien, date de publication

**Given** le site est bilingue
**When** j'accède à `/fr/rss.xml` ou `/en/rss.xml`
**Then** je reçois le flux dans la langue correspondante

**Technical Scope:**
- Server route `server/routes/rss.xml.ts`
- Génération XML avec les articles Nuxt Content
- Versions séparées par langue

**FRs:** FR45
