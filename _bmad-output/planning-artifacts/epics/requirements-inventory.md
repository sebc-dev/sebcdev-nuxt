# Requirements Inventory

## Functional Requirements

**Lecture de Contenu (10 FRs)**
- FR1: Le visiteur peut lire un article complet sur une page dédiée
- FR2: Le visiteur peut voir le thème de l'article (IA / Ingénierie logicielle / UX)
- FR3: Le visiteur peut voir la catégorie de l'article (Actualité / Tutoriel / Décryptage / Étude de cas / Retour d'expérience)
- FR4: Le visiteur peut voir les tags associés à l'article
- FR5: Le visiteur peut voir le niveau de l'article (Tous niveaux / Débutant / Intermédiaire / Avancé)
- FR6: Le visiteur peut voir le temps de lecture estimé
- FR7: Le visiteur peut voir la date de publication
- FR8: Le visiteur peut voir une table des matières générée automatiquement depuis les titres
- FR9: Le visiteur peut voir sa progression de lecture dans l'article (indicateur visuel)
- FR10: Le visiteur peut déplier les sections "Approfondir" (details/summary)

**Interaction avec le Code (3 FRs)**
- FR11: Le visiteur peut voir les blocs de code avec coloration syntaxique
- FR12: Le visiteur peut copier un bloc de code en un clic
- FR13: Le visiteur peut voir le langage du bloc de code (badge)

**Navigation Globale - Header (7 FRs)**
- FR14: Le visiteur peut voir le logo et le nom du site dans le header
- FR15: Le visiteur peut naviguer vers l'accueil via le header
- FR16: Le visiteur peut naviguer vers la page de recherche via le lien "Articles"
- FR17: Le visiteur peut accéder aux thèmes via un dropdown dans le header
- FR18: Le visiteur peut accéder aux catégories via un dropdown dans le header
- FR19: Le visiteur peut accéder aux niveaux via un dropdown dans le header
- FR20: Le visiteur peut sélectionner la langue (FR/EN) via un dropdown à droite du header

**Navigation par Badges (4 FRs)**
- FR21: Le visiteur peut cliquer sur un badge thème pour accéder à la recherche filtrée par ce thème
- FR22: Le visiteur peut cliquer sur un badge catégorie pour accéder à la recherche filtrée par cette catégorie
- FR23: Le visiteur peut cliquer sur un badge tag pour accéder à la recherche filtrée par ce tag
- FR24: Le visiteur peut cliquer sur un badge niveau pour accéder à la recherche filtrée par ce niveau

**Page d'Accueil (2 FRs)**
- FR25: Le visiteur peut voir le dernier article en vedette (pleine largeur)
- FR26: Le visiteur peut voir une grille des articles suivants sous l'article en vedette

**Recherche & Filtrage (10 FRs)**
- FR27: Le visiteur peut accéder à une page de recherche dédiée
- FR28: Le visiteur peut filtrer les articles par thème (sidebar)
- FR29: Le visiteur peut filtrer les articles par catégorie (sidebar)
- FR30: Le visiteur peut filtrer les articles par tag (sidebar)
- FR31: Le visiteur peut filtrer les articles par niveau (sidebar)
- FR32: Le visiteur peut filtrer les articles par temps de lecture (sidebar)
- FR33: Le visiteur peut filtrer les articles par date de publication (sidebar)
- FR34: Le visiteur peut combiner plusieurs filtres simultanément
- FR35: Le visiteur peut voir les filtres actifs reflétés dans l'URL (deep linking)
- FR36: Le visiteur peut naviguer entre les pages de résultats via pagination

**Bilingue (4 FRs)**
- FR37: Le visiteur peut lire le contenu en français
- FR38: Le visiteur peut lire le contenu en anglais
- FR39: Le visiteur peut basculer entre FR et EN sur chaque page
- FR40: Le système détecte la langue préférée du navigateur pour la langue par défaut

**SEO & GEO (5 FRs)**
- FR41: Le système génère un fichier llms.txt accessible aux LLMs
- FR42: Le système génère des Schema Markup (TechArticle, FAQ) par article
- FR43: Le système génère un sitemap XML automatiquement
- FR44: Le système génère des meta tags Open Graph et Twitter Cards
- FR45: Le système génère un flux RSS des articles

**Apparence & Responsive (4 FRs)**
- FR46: Le site s'affiche uniquement en mode sombre (thème fixe, non modifiable)
- FR47: Le site est responsive et adapté à l'affichage desktop
- FR48: Le site est responsive et adapté à l'affichage tablette
- FR49: Le site est responsive et adapté à l'affichage mobile

**Pages Statiques (1 FR)**
- FR50: Le visiteur peut accéder à une page "À propos"

## NonFunctional Requirements

**Performance (5 NFRs)**
- NFR1: Largest Contentful Paint (LCP) < 2.5s
- NFR2: First Input Delay (FID) < 100ms
- NFR3: Cumulative Layout Shift (CLS) < 0.1
- NFR4: Time to First Byte (TTFB) < 200ms
- NFR5: Score Lighthouse Performance ≥ 95 (viser 100)

**Accessibilité (4 NFRs)**
- NFR6: Conformité WCAG Niveau AA (2.1)
- NFR7: Score Lighthouse Accessibility 100
- NFR8: Navigation clavier complète 100% des fonctionnalités
- NFR9: Contraste des couleurs (mode sombre) Ratio ≥ 4.5:1

**SEO Technique (4 NFRs)**
- NFR10: Score Lighthouse SEO 100
- NFR11: Toutes les pages indexables 100%
- NFR12: Temps de crawl par page < 500ms
- NFR13: Validation Schema.org 0 erreur, 0 warning

**Best Practices (1 NFR)**
- NFR14: Score Lighthouse Best Practices 100

**Compatibilité Navigateurs (5 NFRs)**
- NFR15: Chrome (2 dernières versions) - Support complet
- NFR16: Firefox (2 dernières versions) - Support complet
- NFR17: Safari (2 dernières versions) - Support complet
- NFR18: Edge (2 dernières versions) - Support complet
- NFR19: Internet Explorer - Exclu

**Disponibilité (1 NFR)**
- NFR20: Uptime mensuel ≥ 99.5%

**Sécurité (3 NFRs)**
- NFR21: HTTPS obligatoire 100% des pages
- NFR22: Headers de sécurité (CSP, X-Frame-Options, etc.) Configurés
- NFR23: Score Mozilla Observatory ≥ B+

## Additional Requirements

**Exigences Architecture - Infrastructure**
- ARCH-1: Initialisation modulaire du projet (pas de starter template existant)
- ARCH-2: Configuration Cloudflare D1 OBLIGATOIRE (runtime Worker sans accès filesystem)
- ARCH-3: Déploiement sur Cloudflare Pages (Edge global, 330+ DC)
- ARCH-4: Node.js 22 LTS (Active LTS "Jod")
- ARCH-5: pnpm 10.26+ comme package manager

**Exigences Architecture - Stack Technique**
- ARCH-6: Nuxt 4.2.2 avec structure `app/` (srcDir)
- ARCH-7: Nuxt Content 3.10.0+ avec validation Zod 4 / zod/mini
- ARCH-8: TailwindCSS 4.1.17 via @tailwindcss/vite (config CSS-native @theme)
- ARCH-9: shadcn-vue 2.4.3+ avec Reka UI 2.7.0 pour accessibilité
- ARCH-10: MiniSearch 7.x pour recherche client-side (~7KB, index JSON post-build)
- ARCH-11: @nuxtjs/i18n v10.2.1+ (strategy prefix_except_default, strictSeo)
- ARCH-12: @nuxtjs/sitemap v7.5+ avec asSitemapCollection() pour Content v3
- ARCH-13: nuxt-schema-org v5.0.10 pour Schema.org

**Exigences Architecture - Patterns d'Implémentation**
- ARCH-14: Dossier `shared/` pour code isomorphe (types, utils auto-importés)
- ARCH-15: Hydratation lazy native Nuxt 4 (hydrate-on-visible, hydrate-on-idle)
- ARCH-16: Mode sombre unique - tokens oklch définis directement
- ARCH-17: Script postgenerate pour génération index MiniSearch
- ARCH-18: Tests a11y en CI avec @axe-core/playwright

**Exigences UX - Design System**
- UX-1: Palette dark-only avec tokens oklch (--background: #0A0A0B)
- UX-2: Couleurs piliers distinctes (IA: Violet #8B5CF6, Ingénierie: Bleu #3B82F6, UX: Rose #EC4899)
- UX-3: Typography Satoshi Variable (sans-serif) + JetBrains Mono (code)
- UX-4: Spacing scale base 4px
- UX-5: Breakpoints: Mobile < 768px, Tablet 768-1023px, Desktop ≥ 1024px

**Exigences UX - Composants Custom**
- UX-6: CodeBlock avec Shiki, copy button, highlight lines
- UX-7: TableOfContents sticky avec IntersectionObserver
- UX-8: ReadingProgress fixé en haut (3px, accent color)
- UX-9: ArticleCard avec états hover (border accent)
- UX-10: SearchFilters avec Sheet drawer mobile, URL sync
- UX-11: ProseDetails pour sections "Approfondir" avec animation CSS

**Exigences UX - Accessibilité**
- UX-12: Contraste texte minimum 7.2:1 (dépasse WCAG AA)
- UX-13: Focus visible 2px ring accent
- UX-14: Touch targets 48px mobile (dépasse 44px minimum)
- UX-15: Support prefers-reduced-motion
- UX-16: ARIA labels sur tous les éléments interactifs

## FR Coverage Map

| FR | Epic | Description |
|----|------|-------------|
| FR1 | Epic 1 | Article complet sur page dédiée |
| FR46 | Epic 1 | Mode sombre uniquement |
| FR47 | Epic 1 | Responsive desktop |
| FR48 | Epic 1 | Responsive tablette |
| FR49 | Epic 1 | Responsive mobile |
| FR2 | Epic 2 | Badge thème article |
| FR3 | Epic 2 | Badge catégorie article |
| FR4 | Epic 2 | Tags associés |
| FR5 | Epic 2 | Badge niveau article |
| FR6 | Epic 2 | Temps de lecture estimé |
| FR7 | Epic 2 | Date de publication |
| FR8 | Epic 2 | Table des matières auto |
| FR9 | Epic 2 | Progression de lecture |
| FR10 | Epic 2 | Sections Approfondir |
| FR11 | Epic 2 | Coloration syntaxique code |
| FR12 | Epic 2 | Copie code en un clic |
| FR13 | Epic 2 | Badge langage code |
| FR14 | Epic 3 | Logo et nom dans header |
| FR15 | Epic 3 | Navigation vers accueil |
| FR16 | Epic 3 | Lien vers page recherche |
| FR17 | Epic 3 | Dropdown thèmes |
| FR18 | Epic 3 | Dropdown catégories |
| FR19 | Epic 3 | Dropdown niveaux |
| FR20 | Epic 3 | Sélecteur langue FR/EN |
| FR21 | Epic 3 | Badge thème cliquable |
| FR22 | Epic 3 | Badge catégorie cliquable |
| FR23 | Epic 3 | Badge tag cliquable |
| FR24 | Epic 3 | Badge niveau cliquable |
| FR25 | Epic 4 | Article en vedette hero |
| FR26 | Epic 4 | Grille articles accueil |
| FR50 | Epic 4 | Page À propos |
| FR27 | Epic 5 | Page recherche dédiée |
| FR28 | Epic 5 | Filtre par thème |
| FR29 | Epic 5 | Filtre par catégorie |
| FR30 | Epic 5 | Filtre par tag |
| FR31 | Epic 5 | Filtre par niveau |
| FR32 | Epic 5 | Filtre par temps lecture |
| FR33 | Epic 5 | Filtre par date |
| FR34 | Epic 5 | Combinaison filtres |
| FR35 | Epic 5 | Deep linking URL |
| FR36 | Epic 5 | Pagination résultats |
| FR37 | Epic 6 | Contenu en français |
| FR38 | Epic 6 | Contenu en anglais |
| FR39 | Epic 6 | Bascule FR/EN |
| FR40 | Epic 6 | Détection langue navigateur |
| FR41 | Epic 7 | Fichier llms.txt |
| FR42 | Epic 7 | Schema Markup articles |
| FR43 | Epic 7 | Sitemap XML auto |
| FR44 | Epic 7 | Meta tags OG/Twitter |
| FR45 | Epic 7 | Flux RSS |
