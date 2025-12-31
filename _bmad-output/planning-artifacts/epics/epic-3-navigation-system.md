# Epic 3: Navigation System

**Objectif :** Un visiteur peut naviguer intuitivement dans le site via le header et les badges cliquables.

## Story 3.1: Header with Logo and Main Navigation

As a visiteur,
I want to see a header with the site logo and main navigation links,
So that I can easily navigate to key sections of the site.

**Acceptance Criteria:**

**Given** je suis sur n'importe quelle page du site
**When** la page se charge
**Then** je vois le logo du site cliquable (redirige vers `/[lang]/`)
**And** je vois un lien "Articles" qui redirige vers `/[lang]/articles`
**And** le header est visible sur toutes les pages
**And** le header s'adapte en taille selon le device (desktop/mobile)

**Technical Scope:**
- Composant `TheHeader.vue` dans `app/components/layout/`
- Logo SVG ou image optimisée
- Navigation responsive
- NuxtLink pour navigation interne

**FRs:** FR14, FR15, FR16

---

## Story 3.2: Navigation Dropdowns (Themes, Categories, Levels)

As a visiteur,
I want to access themes, categories, and levels via dropdown menus in the header,
So that I can quickly filter articles by these criteria.

**Acceptance Criteria:**

**Given** je suis sur n'importe quelle page
**When** je clique sur le dropdown "Thèmes"
**Then** je vois la liste des thèmes disponibles (IA, Ingénierie logicielle, UX)
**And** chaque option redirige vers `/[lang]/articles?theme=[value]`

**Given** je clique sur le dropdown "Catégories"
**When** le dropdown s'ouvre
**Then** je vois la liste des catégories (Actualité, Tutoriel, Décryptage, Étude de cas, Retour d'expérience)
**And** chaque option redirige vers `/[lang]/articles?category=[value]`

**Given** je clique sur le dropdown "Niveaux"
**When** le dropdown s'ouvre
**Then** je vois la liste des niveaux (Tous niveaux, Débutant, Intermédiaire, Avancé)
**And** chaque option redirige vers `/[lang]/articles?level=[value]`

**Given** un dropdown est ouvert
**When** je clique à l'extérieur ou appuie sur Escape
**Then** le dropdown se ferme
**And** la navigation au clavier (Tab, Enter, Escape) fonctionne

**Technical Scope:**
- Composants `DropdownMenu` de shadcn-vue (Reka UI)
- Accessibilité clavier native via Reka UI
- ARIA labels appropriés

**FRs:** FR17, FR18, FR19
**UX:** UX-16

---

## Story 3.3: Language Switcher

As a visiteur,
I want to switch between French and English via a dropdown,
So that I can read content in my preferred language.

**Acceptance Criteria:**

**Given** je suis sur n'importe quelle page
**When** je clique sur le sélecteur de langue (affiché à droite du header)
**Then** je vois les options "FR" et "EN"
**And** la langue active est mise en évidence
**And** au changement, je suis redirigé vers la même page dans l'autre langue
**And** les filtres et paramètres d'URL sont préservés

**Technical Scope:**
- Composant `LanguageSwitch.vue`
- Utilisation de `useSwitchLocalePath()` de @nuxtjs/i18n
- Dropdown ou boutons toggle
- Préservation des query params

**FRs:** FR20

---

## Story 3.4: Clickable Badge Navigation

As a visiteur,
I want to click on article badges to filter by that criterion,
So that I can discover related content quickly.

**Acceptance Criteria:**

**Given** je suis sur une page article ou une carte article
**When** je clique sur un badge thème (ex: "IA")
**Then** je suis redirigé vers `/[lang]/articles?theme=ia`

**Given** je clique sur un badge catégorie (ex: "Tutoriel")
**When** la redirection s'effectue
**Then** je suis sur `/[lang]/articles?category=tutoriel`

**Given** je clique sur un badge tag (ex: "Vue.js")
**When** la redirection s'effectue
**Then** je suis sur `/[lang]/articles?tag=vuejs`

**Given** je clique sur un badge niveau (ex: "Intermédiaire")
**When** la redirection s'effectue
**Then** je suis sur `/[lang]/articles?level=intermediate`

**And** le curseur est pointer au survol
**And** un état hover visuel indique que le badge est cliquable

**Technical Scope:**
- Mise à jour du composant Badge pour le rendre cliquable
- NuxtLink wrapping ou onClick handler
- Styles hover cohérents

**FRs:** FR21, FR22, FR23, FR24
**UX:** UX-9

---
