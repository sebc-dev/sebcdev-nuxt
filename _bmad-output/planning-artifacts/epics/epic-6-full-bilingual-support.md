# Epic 6: Full Bilingual Support

**Objectif :** Un visiteur lit le contenu dans sa langue préférée (FR ou EN).

## Story 6.1: i18n Configuration & URL Structure

As a visiteur,
I want the site to support French and English with proper URL structure,
So that I can access content in my preferred language.

**Acceptance Criteria:**

**Given** le module @nuxtjs/i18n est configuré
**When** je navigue sur le site
**Then** les URLs suivent le pattern `/fr/...` et `/en/...`
**And** la stratégie `prefix_except_default` est utilisée (FR par défaut)
**And** les balises hreflang sont générées automatiquement

**Technical Scope:**
- Configuration @nuxtjs/i18n v10.2.1+ dans nuxt.config.ts
- Strategy: prefix_except_default
- Default locale: fr
- strictSeo experimental activé
- useLocaleHead() pour hreflang

**FRs:** FR37, FR38 (partiel)
**ARCH:** ARCH-11

---

## Story 6.2: Bilingual Content Structure

As a visiteur,
I want each article to exist in both French and English,
So that I can read content in my language.

**Acceptance Criteria:**

**Given** un article existe dans `content/fr/articles/[slug].md`
**When** la version anglaise est créée dans `content/en/articles/[slug].md`
**Then** les deux versions sont accessibles via leurs URLs respectives
**And** l'interface (navigation, labels, boutons) est traduite dans chaque langue

**Given** un article n'existe pas dans une langue
**When** je tente d'y accéder
**Then** un message "Cet article n'est pas encore traduit" / "This article is not yet translated" s'affiche

**Technical Scope:**
- Structure content/ avec dossiers `fr/` et `en/`
- Fichiers de traduction pour l'interface (i18n locales)
- Gestion du fallback pour articles non traduits

**FRs:** FR37, FR38

---

## Story 6.3: Language Switching with State Preservation

As a visiteur,
I want to switch languages while preserving my current page and filters,
So that I can seamlessly continue browsing in another language.

**Acceptance Criteria:**

**Given** je suis sur une page article `/fr/articles/mon-article`
**When** je clique sur "EN" dans le sélecteur de langue
**Then** je suis redirigé vers `/en/articles/mon-article`
**And** ma position dans l'article est préservée si possible

**Given** je suis sur la page de recherche avec des filtres actifs
**When** je change de langue
**Then** les filtres sont préservés dans l'URL
**And** les labels des filtres sont traduits

**Technical Scope:**
- useSwitchLocalePath() avec préservation query params
- Scroll position restoration
- Traduction des valeurs de filtres si nécessaire

**FRs:** FR39

---

## Story 6.4: Browser Language Detection

As a visiteur,
I want the site to detect my browser language preference,
So that I see content in my preferred language by default.

**Acceptance Criteria:**

**Given** je visite le site pour la première fois sur `/`
**When** mon navigateur envoie `Accept-Language: en`
**Then** je suis redirigé vers `/en/`

**Given** mon navigateur envoie `Accept-Language: fr`
**When** je visite `/`
**Then** je suis redirigé vers `/fr/`

**Given** mon navigateur envoie une langue non supportée (ex: `de`)
**When** je visite `/`
**Then** je suis redirigé vers `/fr/` (fallback)

**Given** j'ai déjà choisi une langue manuellement
**When** je reviens sur le site
**Then** ma préférence est respectée (cookie ou localStorage)

**Technical Scope:**
- detectBrowserLanguage configuration dans i18n
- Cookie pour stocker la préférence utilisateur
- Fallback vers FR

**FRs:** FR40

---
