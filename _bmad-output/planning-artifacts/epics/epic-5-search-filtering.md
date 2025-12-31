# Epic 5: Search & Filtering

**Objectif :** Un visiteur trouve facilement des articles par critères multiples.

## Story 5.1: Search Page Layout

As a visiteur,
I want to access a dedicated search page with filters and results,
So that I can browse and filter all articles.

**Acceptance Criteria:**

**Given** je navigue vers `/[lang]/articles`
**When** la page se charge
**Then** je vois un layout avec sidebar filtres (gauche) et grille résultats (droite)
**And** sur mobile, les filtres sont accessibles via un drawer/modal
**And** le nombre total de résultats est affiché
**And** les articles sont affichés en grille responsive

**Technical Scope:**
- Page `app/pages/articles/index.vue`
- Layout deux colonnes desktop, une colonne mobile
- Composant `SearchFilters.vue` avec Sheet drawer mobile
- Affichage du count total

**FRs:** FR27
**UX:** UX-10

---

## Story 5.2: MiniSearch Integration & Index Generation

As a visiteur,
I want full-text search powered by MiniSearch,
So that I can find articles by keywords quickly.

**Acceptance Criteria:**

**Given** le site est buildé
**When** le script postgenerate s'exécute
**Then** un fichier index JSON est généré dans `public/search-index.json`
**And** l'index contient tous les articles avec titre, description, contenu, métadonnées

**Given** je suis sur la page de recherche
**When** je tape dans le champ de recherche
**Then** les résultats sont filtrés en temps réel côté client
**And** la recherche supporte fuzzy matching et prefix search
**And** le boosting privilégie titre > description > contenu

**Technical Scope:**
- Script `scripts/generate-search-index.mjs`
- MiniSearch 7.x configuration
- Composable `useSearch()`
- Stemming FR optionnel (snowball-stemmers)

**ARCH:** ARCH-10, ARCH-17

---

## Story 5.3: Metadata Filters (Theme, Category, Tag, Level)

As a visiteur,
I want to filter articles by theme, category, tag, and level,
So that I can find content matching my interests and skill level.

**Acceptance Criteria:**

**Given** je suis sur la page de recherche
**When** je coche un filtre thème (ex: "IA")
**Then** seuls les articles avec ce thème sont affichés
**And** le count par option est affiché (ex: "IA (12)")

**Given** je sélectionne plusieurs options dans un même type
**When** les filtres sont appliqués
**Then** la logique OU est utilisée (IA OU UX)

**Given** je sélectionne des filtres de types différents
**When** les filtres sont appliqués
**Then** la logique ET est utilisée (thème:IA ET catégorie:Tutoriel)

**And** l'application des filtres est immédiate (pas de bouton "Appliquer")

**Technical Scope:**
- Composants Checkbox de shadcn-vue
- Logique de filtrage combinée
- Mise à jour réactive des résultats

**FRs:** FR28, FR29, FR30, FR31, FR34

---

## Story 5.4: Time & Date Filters

As a visiteur,
I want to filter articles by reading time and publication date,
So that I can find quick reads or recent content.

**Acceptance Criteria:**

**Given** je suis sur la page de recherche
**When** je sélectionne un filtre temps de lecture
**Then** je peux choisir parmi "< 5 min", "5-10 min", "10-15 min", "> 15 min"
**And** seuls les articles correspondants sont affichés
**And** la sélection est unique (un seul choix possible)

**Given** je sélectionne un filtre date
**When** je choisis une option
**Then** je peux choisir parmi "Cette semaine", "Ce mois", "Cette année", "Tout"
**And** les articles sont filtrés par date de publication

**Technical Scope:**
- Radio buttons ou Select pour sélection unique
- Calcul des plages de dates
- Intégration avec les autres filtres

**FRs:** FR32, FR33

---

## Story 5.5: URL Deep Linking & Filter Reset

As a visiteur,
I want the active filters to be reflected in the URL,
So that I can share or bookmark filtered views.

**Acceptance Criteria:**

**Given** j'applique des filtres
**When** les filtres sont actifs
**Then** l'URL est mise à jour (ex: `/[lang]/articles?theme=ia&category=tutoriel&level=intermediate`)
**And** l'URL est partageable et bookmarkable

**Given** je charge une page avec des paramètres de filtre dans l'URL
**When** la page se charge
**Then** les filtres sont restaurés depuis l'URL
**And** les résultats correspondent aux filtres

**Given** des filtres sont actifs
**When** je clique sur "Réinitialiser les filtres"
**Then** tous les filtres sont désactivés
**And** l'URL est nettoyée

**Technical Scope:**
- useRoute() et useRouter() pour sync URL
- Composable `useFilters()` avec URL sync bidirectionnel
- Bouton reset conditionnel

**FRs:** FR35

---

## Story 5.6: Pagination

As a visiteur,
I want to navigate between pages of results,
So that I can browse all articles without overwhelming the page.

**Acceptance Criteria:**

**Given** il y a plus de 12 articles correspondant aux filtres
**When** je suis sur la page de résultats
**Then** je vois une pagination avec "Précédent" / "Suivant" et numéros de page
**And** 12 articles par page sont affichés (configurable)
**And** le paramètre page est dans l'URL (`?page=2`)

**Given** je change de page
**When** la nouvelle page se charge
**Then** la page scroll automatiquement vers le haut
**And** les filtres actifs sont préservés

**Technical Scope:**
- Composant `Pagination.vue`
- Calcul du nombre de pages
- Scroll to top au changement
- Préservation des query params

**FRs:** FR36

---
