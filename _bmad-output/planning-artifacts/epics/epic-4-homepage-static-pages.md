# Epic 4: Homepage & Static Pages

**Objectif :** Un visiteur voit les articles récents dès l'arrivée et peut en apprendre plus sur l'auteur.

## Story 4.1: Featured Article Hero

As a visiteur,
I want to see the latest article prominently displayed on the homepage,
So that I immediately discover the most recent content.

**Acceptance Criteria:**

**Given** je visite la page d'accueil `/[lang]/`
**When** la page se charge
**Then** je vois le dernier article publié affiché en hero (pleine largeur)
**And** le hero affiche : titre, description, thème (badge), temps de lecture, date
**And** une image de couverture est affichée si disponible (sinon gradient/pattern)
**And** le hero est entièrement cliquable et redirige vers l'article

**Technical Scope:**
- Composant `FeaturedArticleCard.vue`
- Query Nuxt Content pour récupérer le dernier article
- Image de couverture avec fallback
- Layout full-width responsive

**FRs:** FR25

---

## Story 4.2: Article Grid

As a visiteur,
I want to see a grid of recent articles below the featured article,
So that I can browse more content quickly.

**Acceptance Criteria:**

**Given** je suis sur la page d'accueil
**When** je scroll sous le hero
**Then** je vois une grille d'articles (excluant le hero)
**And** sur desktop : 3 colonnes
**And** sur tablette : 2 colonnes
**And** sur mobile : 1 colonne
**And** chaque carte affiche : titre, description tronquée, thème, temps de lecture
**And** le nombre d'articles affichés est 6-9 (configurable)
**And** un lien "Voir tous les articles" redirige vers `/[lang]/articles`

**Technical Scope:**
- Composant `ArticleCard.vue`
- Grid CSS responsive
- Query Nuxt Content avec limit et skip (exclure le premier)
- Lien vers page de recherche

**FRs:** FR26

---

## Story 4.3: About Page

As a visiteur,
I want to access an "About" page,
So that I can learn more about the author and the blog's vision.

**Acceptance Criteria:**

**Given** je navigue vers `/[lang]/about` (EN) ou `/[lang]/a-propos` (FR)
**When** la page se charge
**Then** je vois la présentation de l'auteur
**And** je vois la vision du blog
**And** je vois les informations de contact
**And** le layout est cohérent avec les pages article
**And** la page est accessible depuis le footer ou la navigation

**Technical Scope:**
- Page `app/pages/about.vue` ou contenu dans `content/[lang]/about.md`
- Même layout que les articles
- Lien dans le footer (à créer si non existant)

**FRs:** FR50

---
