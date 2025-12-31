# Epic List

## Epic 1: Core Reading Experience

**Objectif :** Un visiteur peut lire un article complet sur une page dédiée avec un design responsive et sombre.

**Valeur livrée :**
- Setup complet Nuxt 4 + Cloudflare Pages + D1
- Rendu d'articles MDC
- Design dark-only responsive (mobile/tablet/desktop)

**FRs couverts :** FR1, FR46, FR47, FR48, FR49

**Requirements Architecture :** ARCH-1, ARCH-2, ARCH-3, ARCH-4, ARCH-5, ARCH-6, ARCH-7, ARCH-8, ARCH-9, ARCH-14, ARCH-15, ARCH-16

**Requirements UX :** UX-1, UX-3, UX-4, UX-5, UX-12, UX-13, UX-14, UX-15

---

## Epic 2: Rich Article Experience

**Objectif :** Un visiteur bénéficie d'une expérience de lecture enrichie avec métadonnées, navigation interne et interaction code.

**Valeur livrée :**
- Badges (thème, catégorie, tags, niveau)
- Date de publication, temps de lecture
- Table des matières sticky avec scroll smooth
- Barre de progression de lecture
- Sections "Approfondir" dépliables
- Blocs de code avec coloration Shiki et copie

**FRs couverts :** FR2, FR3, FR4, FR5, FR6, FR7, FR8, FR9, FR10, FR11, FR12, FR13

**Requirements UX :** UX-2, UX-6, UX-7, UX-8, UX-11, UX-16

---

## Epic 3: Navigation System

**Objectif :** Un visiteur peut naviguer intuitivement dans le site via le header et les badges cliquables.

**Valeur livrée :**
- Header avec logo et liens principaux
- Dropdowns pour thèmes, catégories, niveaux
- Sélecteur de langue FR/EN
- Badges cliquables redirigeant vers filtres

**FRs couverts :** FR14, FR15, FR16, FR17, FR18, FR19, FR20, FR21, FR22, FR23, FR24

**Requirements UX :** UX-9, UX-16

---

## Epic 4: Homepage & Static Pages

**Objectif :** Un visiteur voit les articles récents dès l'arrivée et peut en apprendre plus sur l'auteur.

**Valeur livrée :**
- Article en vedette (hero full-width)
- Grille d'articles responsive (3/2/1 colonnes)
- Page À propos

**FRs couverts :** FR25, FR26, FR50

---

## Epic 5: Search & Filtering

**Objectif :** Un visiteur trouve facilement des articles par critères multiples.

**Valeur livrée :**
- Page de recherche dédiée avec MiniSearch
- Filtres sidebar (thème, catégorie, tag, niveau, temps, date)
- Combinaison de filtres (ET/OU)
- Deep linking URL partageable
- Pagination

**FRs couverts :** FR27, FR28, FR29, FR30, FR31, FR32, FR33, FR34, FR35, FR36

**Requirements Architecture :** ARCH-10, ARCH-17

**Requirements UX :** UX-10

---

## Epic 6: Full Bilingual Support

**Objectif :** Un visiteur lit le contenu dans sa langue préférée (FR ou EN).

**Valeur livrée :**
- Contenu disponible en FR et EN
- Bascule de langue sur chaque page
- Détection automatique de la langue navigateur
- Structure URL `/fr/` et `/en/`

**FRs couverts :** FR37, FR38, FR39, FR40

**Requirements Architecture :** ARCH-11

---

## Epic 7: SEO & Discovery Optimization

**Objectif :** Le contenu est découvrable par les moteurs de recherche traditionnels et les LLMs.

**Valeur livrée :**
- Fichier llms.txt pour visibilité IA
- Schema.org (TechArticle, FAQPage)
- Sitemap XML multilingue avec hreflang
- Open Graph et Twitter Cards
- Flux RSS FR et EN

**FRs couverts :** FR41, FR42, FR43, FR44, FR45

**Requirements Architecture :** ARCH-12, ARCH-13

**Requirements NFR :** NFR10, NFR11, NFR12, NFR13

---
