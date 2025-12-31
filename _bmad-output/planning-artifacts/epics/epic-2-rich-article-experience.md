# Epic 2: Rich Article Experience

**Objectif :** Un visiteur bénéficie d'une expérience de lecture enrichie avec métadonnées, navigation interne et interaction code.

## Story 2.1: Article Metadata Badges

As a visiteur,
I want to see the article's theme, category, tags, and level as visual badges,
So that I can quickly understand the article's context and relevance.

**Acceptance Criteria:**

**Given** je suis sur une page article
**When** la page se charge
**Then** je vois un badge thème avec la couleur du pilier (IA: violet, Ingénierie: bleu, UX: rose)
**And** je vois un badge catégorie (Actualité/Tutoriel/Décryptage/Étude de cas/Retour d'expérience)
**And** je vois les tags associés sous forme de badges
**And** je vois un badge niveau (Tous niveaux/Débutant/Intermédiaire/Avancé)
**And** les badges sont positionnés dans le header de l'article

**Technical Scope:**
- Composant `Badge` de shadcn-vue
- Couleurs piliers définies (UX-2)
- Extraction des métadonnées depuis le frontmatter Nuxt Content
- Composant `ArticleHeader.vue`

**FRs:** FR2, FR3, FR4, FR5
**UX:** UX-2

---

## Story 2.2: Reading Time & Publication Date

As a visiteur,
I want to see the estimated reading time and publication date,
So that I can decide if I have time to read and know how recent the content is.

**Acceptance Criteria:**

**Given** je suis sur une page article
**When** la page se charge
**Then** je vois le temps de lecture estimé au format "X min de lecture" (FR) / "X min read" (EN)
**And** le calcul est basé sur 200 mots/minute, arrondi à la minute supérieure
**And** je vois la date de publication au format "15 janvier 2025" (FR) / "January 15, 2025" (EN)
**And** la date est localisée selon la langue active

**Technical Scope:**
- Composable `useReadingTime()`
- Calcul au build time via Nuxt Content transformer ou composable
- Formatage date avec `Intl.DateTimeFormat`
- Affichage dans `ArticleHeader.vue`

**FRs:** FR6, FR7

---

## Story 2.3: Table of Contents

As a visiteur,
I want to see a table of contents generated from article headings,
So that I can navigate quickly to specific sections.

**Acceptance Criteria:**

**Given** je suis sur une page article avec des titres H2 et H3
**When** la page se charge
**Then** je vois une table des matières générée automatiquement
**And** sur desktop, elle est affichée en sidebar sticky (visible pendant le scroll)
**And** sur mobile, elle est accessible via un bouton flottant ou collapse
**And** au clic sur un item, la page scroll smooth vers la section
**And** la section active est mise en évidence pendant la lecture

**Technical Scope:**
- Composant `TableOfContents.vue` avec IntersectionObserver
- Extraction des headings via Nuxt Content
- Sticky positioning sur desktop (sidebar 240px)
- Sheet/Modal sur mobile
- Scroll-behavior: smooth

**FRs:** FR8
**UX:** UX-7

---

## Story 2.4: Reading Progress Indicator

As a visiteur,
I want to see my reading progress as I scroll,
So that I know how much of the article remains.

**Acceptance Criteria:**

**Given** je suis sur une page article
**When** je scroll dans l'article
**Then** une barre de progression horizontale est visible en haut de la page
**And** la barre progresse de 0% (haut) à 100% (fin de l'article)
**And** la barre fait 3px de hauteur avec la couleur accent
**And** la barre est visible uniquement sur les pages article

**Technical Scope:**
- Composant `ReadingProgress.vue`
- Calcul basé sur scroll position vs article height
- Position fixed top, z-index élevé
- Couleur accent (#14B8A6)

**FRs:** FR9
**UX:** UX-8

---

## Story 2.5: Expandable "Approfondir" Sections

As a visiteur,
I want to expand optional "Approfondir" sections,
So that I can access additional details without cluttering the main content.

**Acceptance Criteria:**

**Given** je suis sur une page article avec des sections "Approfondir"
**When** je clique sur le titre d'une section
**Then** le contenu se déplie avec une animation smooth
**And** une icône chevron indique l'état (→ fermé, ↓ ouvert)
**And** les sections sont fermées par défaut

**Technical Scope:**
- Composant `ProseDetails.vue` (MDC component)
- HTML natif `<details><summary>`
- Animation CSS transition (height, opacity)
- Styling cohérent avec le design system

**FRs:** FR10
**UX:** UX-11

---

## Story 2.6: Code Blocks with Syntax Highlighting & Copy

As a visiteur développeur,
I want to see syntax-highlighted code blocks with copy functionality,
So that I can easily read and reuse code examples.

**Acceptance Criteria:**

**Given** je suis sur une page article avec des blocs de code
**When** la page se charge
**Then** le code est affiché avec coloration syntaxique (TypeScript, JavaScript, Vue, CSS, JSON, Bash, Markdown)
**And** un badge indique le langage en haut à gauche du bloc
**And** un bouton "Copier" est visible au survol ou en permanence
**And** au clic sur Copier, le code est copié dans le presse-papier
**And** un feedback "✓ Copié!" apparaît pendant 2 secondes
**And** les numéros de ligne sont optionnels (activables)
**And** certaines lignes peuvent être mises en évidence (highlight)

**Technical Scope:**
- Shiki pour syntax highlighting (intégré à Nuxt Content)
- Composant `CodeBlock.vue` personnalisé
- Clipboard API pour copie
- Props: `language`, `filename?`, `highlightLines?`, `showLineNumbers?`
- Thème de coloration cohérent avec le dark mode

**FRs:** FR11, FR12, FR13
**UX:** UX-6

---
