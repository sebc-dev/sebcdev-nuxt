# 13. Functional Requirements (MVP)

Cette section définit le **contrat de capacités** du produit. Chaque FR est une capacité testable que le système doit fournir.

## 13.1 Lecture de Contenu

| FR# | Requirement |
|-----|-------------|
| FR1 | Le visiteur peut lire un article complet sur une page dédiée |
| FR2 | Le visiteur peut voir le thème de l'article (IA / Ingénierie logicielle / UX) |
| FR3 | Le visiteur peut voir la catégorie de l'article (Actualité / Tutoriel / Décryptage / Étude de cas / Retour d'expérience) |
| FR4 | Le visiteur peut voir les tags associés à l'article |
| FR5 | Le visiteur peut voir le niveau de l'article (Tous niveaux / Débutant / Intermédiaire / Avancé) |
| FR6 | Le visiteur peut voir le temps de lecture estimé |
| FR7 | Le visiteur peut voir la date de publication |
| FR8 | Le visiteur peut voir une table des matières générée automatiquement depuis les titres |
| FR9 | Le visiteur peut voir sa progression de lecture dans l'article (indicateur visuel) |
| FR10 | Le visiteur peut déplier les sections "Approfondir" (details/summary) |

### Critères d'Acceptation — Lecture de Contenu

**FR1 — Article complet**
- L'article s'affiche sur une URL unique `/[lang]/articles/[slug]`
- Le contenu Markdown/MDC est rendu correctement (titres, paragraphes, listes, images, liens)
- Les images ont un attribut alt et sont lazy-loaded
- Le temps de chargement initial < 2s (LCP)

**FR2-5 — Métadonnées article**
- Thème, catégorie, niveau et tags sont affichés sous forme de badges cliquables
- Chaque badge utilise une couleur distinctive par type (thème: couleur pilier, catégorie: neutre, niveau: gradient)
- Les badges sont positionnés de manière cohérente (header article ou sidebar)

**FR6 — Temps de lecture**
- Calcul basé sur 200 mots/minute (standard FR)
- Arrondi à la minute supérieure
- Format affiché : "X min de lecture"

**FR7 — Date de publication**
- Format FR : "15 janvier 2025"
- Format EN : "January 15, 2025"
- Localisée selon la langue active

**FR8 — Table des matières (ToC)**
- Générée automatiquement depuis les titres H2 et H3
- Affichée en sidebar sticky sur desktop (visible pendant le scroll)
- Sur mobile : accessible via bouton flottant ou collapse en haut d'article
- Scroll smooth vers la section au clic
- Highlight de la section active pendant la lecture

**FR9 — Progression de lecture**
- Barre de progression horizontale fixée en haut de la page
- Progression basée sur le scroll (0% en haut, 100% à la fin de l'article)
- Visible uniquement sur les pages article

**FR10 — Sections Approfondir**
- Utilise `<details><summary>` HTML natif
- État fermé par défaut
- Animation smooth à l'ouverture/fermeture (CSS transition)
- Icône chevron indiquant l'état (→ ouvert, ↓ fermé)

## 13.2 Interaction avec le Code

| FR# | Requirement |
|-----|-------------|
| FR11 | Le visiteur peut voir les blocs de code avec coloration syntaxique |
| FR12 | Le visiteur peut copier un bloc de code en un clic |
| FR13 | Le visiteur peut voir le langage du bloc de code (badge) |

### Critères d'Acceptation — Interaction Code

**FR11 — Coloration syntaxique**
- Support des langages : TypeScript, JavaScript, Vue/HTML, CSS, JSON, Bash, Markdown
- Thème de coloration cohérent avec le design sombre du site
- Numéros de ligne optionnels (activables via prop)
- Highlight de lignes spécifiques (ex: `{3-5}` pour lignes 3 à 5)

**FR12 — Copie code**
- Bouton "Copier" visible au survol ou en permanence (coin supérieur droit)
- Au clic : copie dans le presse-papier
- Feedback visuel : icône change en "✓" ou texte "Copié!" pendant 2 secondes
- Copie le code brut (sans numéros de ligne ni formatage)

**FR13 — Badge langage**
- Badge affiché en haut à gauche du bloc de code
- Texte en minuscules (ex: "typescript", "vue", "bash")
- Style discret, ne pas distraire de la lecture

## 13.3 Navigation Globale (Header)

| FR# | Requirement |
|-----|-------------|
| FR14 | Le visiteur peut voir le logo et le nom du site dans le header |
| FR15 | Le visiteur peut naviguer vers l'accueil via le header |
| FR16 | Le visiteur peut naviguer vers la page de recherche via le lien "Articles" |
| FR17 | Le visiteur peut accéder aux thèmes via un dropdown dans le header |
| FR18 | Le visiteur peut accéder aux catégories via un dropdown dans le header |
| FR19 | Le visiteur peut accéder aux niveaux via un dropdown dans le header |
| FR20 | Le visiteur peut sélectionner la langue (FR/EN) via un dropdown à droite du header |

### Critères d'Acceptation — Navigation Header

**FR14-15 — Logo et accueil**
- Logo cliquable, redirige vers `/[lang]/`
- Logo visible sur toutes les pages
- Taille adaptée : desktop et mobile

**FR16 — Lien Articles**
- Lien texte "Articles" dans la navigation principale
- Redirige vers `/[lang]/articles` (page de recherche/listing)

**FR17-19 — Dropdowns navigation**
- Dropdowns pour Thèmes, Catégories, Niveaux
- Au clic : affiche liste des options disponibles
- Chaque option redirige vers `/[lang]/articles?[filter]=[value]`
- Fermeture au clic extérieur ou touche Escape
- Accessible au clavier (Tab, Enter, Escape)

**FR20 — Sélecteur de langue**
- Dropdown FR/EN positionné à droite du header
- Langue active affichée (ex: "FR" ou drapeau)
- Au changement : redirige vers la même page dans l'autre langue
- Préserve les filtres et paramètres d'URL

## 13.4 Navigation par Badges

| FR# | Requirement |
|-----|-------------|
| FR21 | Le visiteur peut cliquer sur un badge thème pour accéder à la recherche filtrée par ce thème |
| FR22 | Le visiteur peut cliquer sur un badge catégorie pour accéder à la recherche filtrée par cette catégorie |
| FR23 | Le visiteur peut cliquer sur un badge tag pour accéder à la recherche filtrée par ce tag |
| FR24 | Le visiteur peut cliquer sur un badge niveau pour accéder à la recherche filtrée par ce niveau |

### Critères d'Acceptation — Navigation Badges

**FR21-24 — Badges cliquables**
- Tous les badges (thème, catégorie, tag, niveau) sont cliquables
- Au clic : redirige vers `/[lang]/articles?[type]=[value]`
- Curseur pointer au survol
- État hover visuel (légère mise en évidence)

## 13.5 Page d'Accueil

| FR# | Requirement |
|-----|-------------|
| FR25 | Le visiteur peut voir le dernier article en vedette (pleine largeur) |
| FR26 | Le visiteur peut voir une grille des articles suivants sous l'article en vedette |

### Critères d'Acceptation — Page d'Accueil

**FR25 — Article en vedette**
- Le dernier article publié est affiché en hero (pleine largeur)
- Affiche : titre, description, thème, temps de lecture, date
- Image de couverture si disponible (fallback : gradient ou pattern)
- Lien cliquable vers l'article complet

**FR26 — Grille articles**
- Affiche les articles suivants (excluant le hero) en grille
- Desktop : 3 colonnes
- Tablette : 2 colonnes
- Mobile : 1 colonne
- Chaque carte affiche : titre, description tronquée, thème, temps de lecture
- Nombre d'articles affichés : 6-9 (configurable)
- Lien "Voir tous les articles" vers la page de recherche

## 13.6 Recherche & Filtrage

| FR# | Requirement |
|-----|-------------|
| FR27 | Le visiteur peut accéder à une page de recherche dédiée |
| FR28 | Le visiteur peut filtrer les articles par thème (sidebar) |
| FR29 | Le visiteur peut filtrer les articles par catégorie (sidebar) |
| FR30 | Le visiteur peut filtrer les articles par tag (sidebar) |
| FR31 | Le visiteur peut filtrer les articles par niveau (sidebar) |
| FR32 | Le visiteur peut filtrer les articles par temps de lecture (sidebar) |
| FR33 | Le visiteur peut filtrer les articles par date de publication (sidebar) |
| FR34 | Le visiteur peut combiner plusieurs filtres simultanément |
| FR35 | Le visiteur peut voir les filtres actifs reflétés dans l'URL (deep linking) |
| FR36 | Le visiteur peut naviguer entre les pages de résultats via pagination |

### Critères d'Acceptation — Recherche & Filtrage

**FR27 — Page de recherche**
- URL : `/[lang]/articles`
- Layout : sidebar filtres (gauche) + grille résultats (droite)
- Sur mobile : filtres accessibles via drawer/modal
- Affiche le nombre total de résultats

**FR28-31 — Filtres par métadonnées**
- Filtres checkbox pour : thème, catégorie, tag, niveau
- Affichage du count par option (ex: "IA (12)")
- Sélection multiple possible au sein d'un même type
- Application immédiate (pas de bouton "Appliquer")

**FR32 — Filtre temps de lecture**
- Options prédéfinies : "< 5 min", "5-10 min", "10-15 min", "> 15 min"
- Sélection unique

**FR33 — Filtre date de publication**
- Options prédéfinies : "Cette semaine", "Ce mois", "Cette année", "Tout"
- Sélection unique

**FR34 — Combinaison de filtres**
- Logique ET entre types de filtres différents (thème ET catégorie)
- Logique OU au sein d'un même type (IA OU UX)
- Bouton "Réinitialiser les filtres" visible si filtres actifs

**FR35 — Deep linking URL**
- Format : `/[lang]/articles?theme=ia&category=tutoriel&level=intermediate`
- URL partageable et bookmarkable
- Au chargement : restaure les filtres depuis l'URL

**FR36 — Pagination**
- 12 articles par page (configurable)
- Navigation : "Précédent" / "Suivant" + numéros de page
- Scroll to top au changement de page
- Paramètre URL : `?page=2`

## 13.7 Bilingue

| FR# | Requirement |
|-----|-------------|
| FR37 | Le visiteur peut lire le contenu en français |
| FR38 | Le visiteur peut lire le contenu en anglais |
| FR39 | Le visiteur peut basculer entre FR et EN sur chaque page |
| FR40 | Le système détecte la langue préférée du navigateur pour la langue par défaut |

### Critères d'Acceptation — Bilingue

**FR37-38 — Contenu multilingue**
- Chaque article existe en version FR et EN
- URL structure : `/fr/articles/[slug]` et `/en/articles/[slug]`
- Interface (navigation, labels, boutons) traduite dans chaque langue
- Si un article n'existe pas dans une langue, afficher message "Cet article n'est pas encore traduit"

**FR39 — Bascule de langue**
- Sélecteur de langue accessible depuis le header (toutes pages)
- Au clic : redirige vers l'équivalent dans l'autre langue
- Préserve la position dans l'article si possible

**FR40 — Détection langue navigateur**
- À la première visite sur `/`, détecter `Accept-Language` header
- Rediriger vers `/fr/` ou `/en/` selon la préférence
- Fallback : FR si langue non supportée
- Stocker le choix utilisateur (cookie ou localStorage) pour les visites suivantes

## 13.8 SEO & GEO

| FR# | Requirement |
|-----|-------------|
| FR41 | Le système génère un fichier llms.txt accessible aux LLMs |
| FR42 | Le système génère des Schema Markup (TechArticle, FAQ) par article |
| FR43 | Le système génère un sitemap XML automatiquement |
| FR44 | Le système génère des meta tags Open Graph et Twitter Cards |
| FR45 | Le système génère un flux RSS des articles |

### Critères d'Acceptation — SEO & GEO

**FR41 — llms.txt**
- Fichier accessible à `/llms.txt`
- Contenu : présentation du site, liste des articles avec descriptions, guidelines pour citations
- Format conforme à https://llmstxt.org/
- Mis à jour automatiquement à chaque build

**FR42 — Schema Markup**
- Chaque article inclut JSON-LD `TechArticle` avec : headline, author, datePublished, description
- Si FAQ présente dans l'article, inclure schema `FAQPage`
- Validation sans erreur sur https://validator.schema.org/

**FR43 — Sitemap XML**
- Généré automatiquement à `/sitemap.xml`
- Inclut toutes les pages publiques (accueil, articles, pages statiques)
- Versions FR et EN avec hreflang
- Lastmod basé sur la date de modification

**FR44 — Open Graph & Twitter Cards**
- Chaque page inclut : og:title, og:description, og:image, og:url, og:type
- Twitter Card : summary_large_image
- Image OG générée dynamiquement ou image de couverture de l'article

**FR45 — Flux RSS**
- Accessible à `/rss.xml` (ou `/feed.xml`)
- Versions séparées FR et EN : `/fr/rss.xml`, `/en/rss.xml`
- Inclut les 20 derniers articles avec : titre, description, lien, date

## 13.9 Apparence & Responsive

| FR# | Requirement |
|-----|-------------|
| FR46 | Le site s'affiche uniquement en mode sombre (thème fixe, non modifiable) |
| FR47 | Le site est responsive et adapté à l'affichage desktop |
| FR48 | Le site est responsive et adapté à l'affichage tablette |
| FR49 | Le site est responsive et adapté à l'affichage mobile |

### Critères d'Acceptation — Apparence & Responsive

**FR46 — Mode sombre**
- Palette de couleurs sombres uniquement (pas de toggle clair/sombre)
- Couleurs définies via CSS variables pour cohérence
- Contraste texte/fond conforme WCAG AA (ratio ≥ 4.5:1)

**FR47 — Desktop**
- Breakpoint : ≥ 1024px
- Layout full-width avec max-width container (1200-1400px)
- Sidebar ToC visible en permanence sur les articles
- Grilles à 3 colonnes pour les listings

**FR48 — Tablette**
- Breakpoint : 768px - 1023px
- Layout adapté avec sidebar collapsible
- Grilles à 2 colonnes pour les listings
- Navigation header complète

**FR49 — Mobile**
- Breakpoint : < 768px
- Navigation hamburger ou drawer
- ToC accessible via bouton flottant
- Grilles à 1 colonne
- Touch targets ≥ 44px (accessibilité)

## 13.10 Pages Statiques

| FR# | Requirement |
|-----|-------------|
| FR50 | Le visiteur peut accéder à une page "À propos" |

### Critères d'Acceptation — Pages Statiques

**FR50 — Page À propos**
- URL : `/[lang]/about` ou `/[lang]/a-propos`
- Contenu : présentation de l'auteur, vision du blog, contact
- Même layout que les articles (cohérence visuelle)
- Accessible depuis le footer ou la navigation

## 13.11 Récapitulatif MVP

| Domaine | Nombre de FRs |
|---------|---------------|
| Lecture de Contenu | 10 |
| Interaction Code | 3 |
| Navigation Globale (Header) | 7 |
| Navigation Badges | 4 |
| Page d'Accueil | 2 |
| Recherche & Filtrage | 10 |
| Bilingue | 4 |
| SEO & GEO | 5 |
| Apparence & Responsive | 4 |
| Pages Statiques | 1 |
| **TOTAL MVP** | **50 FRs** |

---
