---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
inputDocuments:
  - _bmad-output/planning-artifacts/product-brief-sebcdev-nuxt.md
  - _bmad-output/planning-artifacts/prd/1-executive-summary.md
  - _bmad-output/planning-artifacts/prd/2-personas-dtaills.md
  - _bmad-output/planning-artifacts/prd/7-structure-contenu-answer-first.md
  - _bmad-output/planning-artifacts/prd/13-functional-requirements-mvp.md
  - _bmad-output/planning-artifacts/prd/14-non-functional-requirements-mvp.md
  - docs/old/UX_UI_Spec.md
---

# UX Design Specification — sebc.dev

**Auteur:** Negus
**Date:** 31 décembre 2025

---

## Executive Summary

### Project Vision

**sebc.dev** est un blog technique bilingue (FR/EN) documentant en temps réel le parcours d'un développeur autodidacte à l'intersection de trois piliers : IA (40%), Ingénierie Logicielle (30%), et UX (30%).

Le positionnement "R&D Engineering in Public" se différencie des blogs experts traditionnels par la transparence totale sur le processus d'apprentissage — partage des résultats validés ET des échecs documentés.

La thèse centrale : face à l'IA omniprésente fin 2025, les développeurs doivent maîtriser 3 domaines complémentaires plutôt que se spécialiser dans le code pur.

### Target Users

**Lucas — "Le Mid-Level Bloqué" (Primary)**
- Développeur Fullstack 27 ans, 4 ans d'expérience, startup SaaS
- Friction : 70% du temps en "glue code" IA, manque de vision architecturale
- Objectif UX : Time-to-value < 60 secondes, code copy-paste fonctionnel immédiatement
- Succès : "Ce mec gère les edge-cases que j'avais oubliés. C'est du code de prod."

**Chloé — "L'Apprentie Copilote" (Secondary)**
- Développeuse Junior 24 ans, bootcamp 2024, 10 mois de code
- Friction : "Hollow Senior" - code avec IA sans comprendre, syndrome imposteur violent
- Objectif UX : Pattern discovery < 3 minutes, comprendre le "pourquoi"
- Succès : "Je savais ce qu'il fallait faire! Je ne suis pas nulle!"

**Maxime — "L'Architecte Frankenstein" (Secondary)**
- Solopreneur 32 ans, ex-Senior Dev, 3 micro-SaaS + freelance
- Friction : Integration Hell, 15+ services, ~400€/mois gaspillés
- Objectif UX : Vue d'ensemble rapide, ROI clair, comparatifs coûts
- Succès : "Ce gars me fait gagner de l'argent."

### Key Design Challenges

1. **Pattern Onion Multi-Audience** — Servir 3 personas avec 1 article sans confusion ni dilution. Lucas lit TL;DR + Code, Chloé lit Pattern + Deep Dive, Maxime lit Hook + Comparatifs.

2. **Time-to-Value Extrême** — Lucas doit trouver une solution copy-paste en < 60 secondes. La structure Answer-First impose la réponse dans les 100 premiers mots.

3. **Révélation Progressive** — Chloé a besoin de profondeur pédagogique, mais sans submerger Lucas qui veut juste le code. Les sections `<details>` doivent être clairement signalées mais non-intrusives.

4. **Bilingue Sans Friction** — 100% des articles en FR et EN simultanément. Le switch de langue doit préserver la position de lecture et les filtres actifs.

5. **Performance Lighthouse 100** — Objectif non-négociable. Impact direct sur le design : lazy loading, optimisation images, bundle minimal.

### Design Opportunities

1. **Table des Matières Intelligente** — ToC sticky avec highlight section active ET temps de lecture par section. Permet à chaque persona de naviguer selon son objectif.

2. **Badges Cliquables Contextuels** — Chaque badge (thème, catégorie, tag, niveau) est un point d'entrée vers la recherche filtrée. Découverte par sérendipité.

3. **Métriques Comportementales** — Copy Rate (Lucas), Scroll Depth (Pattern Onion validation), Temps par section (Chloé engagement). Analytics comme outil de validation UX.

4. **Structure Answer-First GEO-Ready** — Optimisation pour les IA de recherche (Perplexity, ChatGPT). Premier paragraphe = réponse directe citée par les LLMs.

## Core User Experience

### Defining Experience

L'expérience centrale de sebc.dev est la **consommation efficace d'un article technique selon l'objectif de chaque persona**.

Le même article sert 3 parcours distincts :
- **Lucas (Pragmatique)** : TL;DR → Code → Copy → Termine (< 60s)
- **Chloé (Apprenante)** : TL;DR → Pattern → Details → Comprend (< 3min)
- **Maxime (Stratégique)** : Hook → Comparatifs → Décide → Termine (< 90s)

L'architecture "Pattern Onion" permet ces 3 parcours sur le même contenu sans compromis ni dilution.

### Platform Strategy

| Aspect | Décision |
|--------|----------|
| **Plateforme** | Web responsive |
| **Device prioritaire** | Desktop-first (développeurs codent sur desktop) |
| **Input primaire** | Clavier + souris |
| **Touch** | Support secondaire (lecture mobile) |
| **Offline** | Non requis MVP |

**Breakpoints :**
- Desktop : ≥ 1024px (ToC sticky visible, grilles 3 colonnes)
- Tablette : 768px - 1023px (ToC drawer, grilles 2 colonnes)
- Mobile : < 768px (ToC modal, grilles 1 colonne, touch targets 44px)

### Effortless Interactions

| Interaction | Design |
|-------------|--------|
| **Copier du code** | Bouton "Copy" visible en permanence, feedback "Copié!" 2s |
| **Naviguer article** | ToC sticky desktop, highlight section active, scroll smooth |
| **Trouver la réponse** | TL;DR en haut, structure Answer-First, réponse < 100 mots |
| **Filtrer articles** | Sidebar filtres, application immédiate, URL partageable |
| **Changer de langue** | Switch FR/EN préservant position et filtres |
| **Voir progression** | Barre horizontale sticky top + highlight ToC |

### Critical Success Moments

1. **"Ça compile du premier coup"** — Lucas colle le code TypeScript et ça fonctionne. Edge-cases gérés. Confiance établie.

2. **"Je comprends enfin pourquoi"** — Chloé lit la section Pattern et a son moment eureka. Syndrome imposteur diminué.

3. **"Ce gars me fait gagner du temps/argent"** — Maxime trouve le tableau comparatif en 30 secondes. Efficacité respectée.

4. **"C'est instantané"** — Page charge < 2.5s (LCP). L'utilisateur ne perçoit pas d'attente.

5. **"Je peux naviguer au clavier"** — Accessibilité complète. Aucun utilisateur exclu.

### Experience Principles

1. **Answer-First, Always**
   - La valeur est accessible dans les 60 premières secondes
   - TL;DR en haut, code copable immédiatement
   - Pas de friction avant la réponse

2. **Progressive Depth, Not Gates**
   - La profondeur est disponible, jamais obligatoire
   - Sections `<details>` pour approfondir sans forcer
   - Chaque persona contrôle sa profondeur de lecture

3. **Navigation as Orientation**
   - ToC, badges, filtres = outils d'orientation
   - L'utilisateur sait toujours où il est
   - L'utilisateur sait toujours où aller ensuite

4. **Performance is UX**
   - Lighthouse 100/100/100/100 = promesse UX
   - Chaque milliseconde de latence érode la confiance
   - Performance perçue > performance mesurée

5. **Code is Content**
   - Blocs de code = éléments de premier ordre
   - Copy button, syntax highlighting, language badge obligatoires
   - TypeScript strict, commentaires contextuels

## Desired Emotional Response

### Primary Emotional Goals

| Persona | Émotion Primaire | Ce qui la Déclenche |
|---------|------------------|---------------------|
| **Lucas** | **Soulagement** | Code copy-paste qui fonctionne du premier coup |
| **Chloé** | **Validation** | Comprendre le "pourquoi" et pouvoir l'expliquer |
| **Maxime** | **Respect** | Contenu qui va droit au but, valeur immédiate |

**Émotion Unificatrice :** "Ce blog respecte mon temps ET mon intelligence."

### Emotional Journey Mapping

**Découverte → Première Impression → Lecture → Action → Après**

| Étape | Durée | Émotion Cible | Design |
|-------|-------|---------------|--------|
| **Découverte** | 0-5s | Espoir / Curiosité | Titre clair, structure visible |
| **Première impression** | 5-15s | Reconnaissance / Sécurité | TL;DR visible, code apparent |
| **Lecture active** | 15s-3min | Clarté / Compréhension | Answer-First, progression logique |
| **Action principale** | Variable | Satisfaction / Eureka | Code fonctionnel, "Ah!" moments |
| **Après visite** | N/A | Confiance / Empowerment | Envie de revenir, de partager |

### Micro-Emotions

**Émotions à Cultiver :**
- **Confiance** — Le code fonctionne, l'auteur sait de quoi il parle
- **Orientation** — L'utilisateur sait toujours où il est et où aller
- **Compétence** — L'utilisateur se sent capable, pas dépassé
- **Efficacité** — Pas de temps perdu, valeur immédiate
- **Inclusion** — Accessibilité complète, personne n'est exclu
- **Authenticité** — Ton pair-à-pair, vulnérabilité assumée

**Émotions à Éviter :**
- **Frustration** — Code qui ne marche pas
- **Confusion** — Structure floue, jargon non-expliqué
- **Syndrome imposteur** — Contenu condescendant
- **Méfiance** — Promesses non tenues
- **Impatience** — Page lente, valeur cachée
- **Exclusion** — Accessibilité négligée

### Design Implications

| Objectif Émotionnel | Implication Design |
|---------------------|-------------------|
| Soulagement (Lucas) | Blocs de code avec bouton Copy, TypeScript strict, edge-cases gérés |
| Confiance | Commentaires explicatifs dans le code, pas de "laissé en exercice" |
| Validation (Chloé) | Sections `<details>` "Pourquoi ce choix?", ton bienveillant |
| Empowerment | Structure progressive simple → complexe, pas de sauts |
| Respect (Maxime) | Tableaux comparatifs, chiffres concrets, Answer-First |
| Authenticité | Mentions d'échecs, ton "Learning in Public", pas de posture infaillible |
| Orientation | ToC sticky, highlight section active, barre progression |
| Inclusion | WCAG AA, navigation clavier complète, contraste suffisant |

### Emotional Design Principles

1. **Délivrer Avant de Promettre**
   - La valeur est prouvée, pas annoncée
   - Le code fonctionne avant d'être expliqué
   - Les résultats précèdent les méthodes

2. **Respecter l'Intelligence**
   - Ton pair-à-pair, jamais condescendant
   - Expliquer le "pourquoi", pas juste le "comment"
   - Vulnérabilité assumée ("J'ai d'abord essayé X, ça n'a pas marché")

3. **Réduire l'Anxiété**
   - Structure prévisible et cohérente
   - Toujours savoir où on est (ToC, progression)
   - Pas de surprises négatives (page lente, code cassé)

4. **Créer des Moments "Eureka"**
   - Sections "Approfondir" pour les curieux
   - Connexions entre concepts
   - "Ah! C'est pour ça!" intégré dans la structure

5. **Inviter au Retour**
   - Expérience positive = envie de revenir
   - Contenu qui se partage naturellement
   - Confiance construite sur plusieurs visites

## UX Pattern Analysis & Inspiration

### Inspiring Products Analysis

#### Josh Comeau (joshwcomeau.com)
- **Excellence** : Visualisations interactives, ton pédagogique bienveillant, progression claire
- **Patterns clés** : ToC sticky avec highlight, sections "Digging Deeper" collapsibles, code avec copy button
- **Applicable** : Ton "Learning in Public", structure progressive, ToC comportement

#### Stripe Documentation
- **Excellence** : Code snippets parfaits, navigation latérale efficace, language switcher
- **Patterns clés** : Copy button avec feedback, multi-language tabs, breadcrumbs contextuels
- **Applicable** : Copy button design, feedback animation, navigation hiérarchique

#### Tailwind CSS Documentation
- **Excellence** : Recherche ultra-rapide, exemples copy-paste, structure catégorisée
- **Patterns clés** : DocSearch, sidebar collapsible, dark mode cohérent
- **Applicable** : Recherche (MVP+), sidebar filtres, dark-only design

#### MDN Web Docs
- **Excellence** : Profondeur progressive, compatibilité navigateur, références croisées
- **Patterns clés** : Autorité + accessibilité, structure encyclopédique
- **Applicable** : Profondeur pour Chloé, accessibilité comme standard

### Transferable UX Patterns

**Navigation & Orientation**
| Pattern | Application |
|---------|-------------|
| ToC Sticky + Highlight | Sidebar droite desktop, highlight section active au scroll |
| Barre Progression | Horizontale sticky top, 0-100% basée sur scroll |
| Breadcrumbs | Accueil > Articles > [Titre article] |
| Filtres Sidebar | Thème, catégorie, niveau, tags — application immédiate |

**Code & Interaction**
| Pattern | Application |
|---------|-------------|
| Copy Button Visible | Toujours visible (pas hover-only), coin supérieur droit |
| Feedback "Copié!" | Icône → checkmark, texte "Copié!" pendant 2s |
| Language Badge | Badge discret coin supérieur gauche ("typescript", "vue") |
| Highlight Lignes | Support `{3-5}` pour focus attention sur lignes spécifiques |

**Révélation Progressive**
| Pattern | Application |
|---------|-------------|
| `<details>` Natif | Sections "Pourquoi ce choix?", "Approfondir" |
| Smooth Animation | CSS transition à l'ouverture/fermeture |
| État Fermé Default | Ne pas submerger Lucas avec la profondeur Chloé |

### Anti-Patterns to Avoid

| Anti-Pattern | Problème | Notre Approche |
|--------------|----------|----------------|
| Code non-copable | Frustration, perte de temps | Copy button obligatoire sur chaque bloc |
| ToC cachée | Perte d'orientation | ToC visible en permanence (desktop) |
| Intro avant valeur | Time-to-value > 60s | TL;DR en haut, Answer-First |
| Popup newsletter | Interruption, méfiance | CTA discret fin d'article uniquement |
| Mode clair default | Pas le standard dev | Dark mode fixe, pas de toggle |
| Hover-only actions | Accessibilité, mobile | Actions toujours visibles |
| Jargon non-expliqué | Junior perdu | Définitions inline ou liens |

### Design Inspiration Strategy

**Adopter Directement :**
- ToC sticky avec highlight section active (Josh Comeau)
- Copy button visible avec feedback animation (Stripe)
- Barre de progression lecture (Medium, Josh Comeau)
- Dark mode unique (Tailwind, standard dev)
- Sections `<details>` pour profondeur (MDN, Josh Comeau)

**Adapter pour sebc.dev :**
- Recherche : MiniSearch local pour MVP, Algolia considérer post-MVP
- Language switch : Toggle FR/EN global (pas tabs par bloc comme Stripe)
- Visualisations : Réserver pour articles IA complexes, pas systématique

**Éviter Explicitement :**
- Popup/modal newsletter intrusif
- Hover-only pour actions importantes
- Infinite scroll (pagination pour référence)
- Mode clair ou toggle theme

## Design System Foundation

### Design System Choice

**Choix Principal : shadcn-vue**
- Port Vue de shadcn/ui basé sur Reka UI (primitives accessibles)
- Composants copy-paste = ownership complet du code
- Styling Tailwind CSS natif avec support dark mode
- Performance optimale (tree-shakable, minimal bundle)

**Fondation : Reka UI (ex-Radix Vue)**
- Primitives accessibles WCAG AA pour composants complexes
- Base pour Dialog, Sheet, Tooltip, Collapsible
- Unstyled = contrôle total du styling Tailwind

**Styling : Tailwind CSS 4**
- Utility-first pour rapidité de développement
- Dark mode via classe `dark:` (dark-only pour sebc.dev)
- Design tokens via CSS variables
- Responsive breakpoints cohérents

### Rationale for Selection

| Critère | Décision | Justification |
|---------|----------|---------------|
| **Accessibilité** | shadcn-vue + Reka UI | Radix primitives = WCAG AA natif |
| **Performance** | shadcn-vue | Tree-shakable, pas de runtime CSS-in-JS |
| **Temps Dev (Solo)** | shadcn-vue | Copy-paste ready, moins de code custom |
| **Dark Mode** | Tailwind `dark:` | Système natif, cohérent |
| **Customisation** | shadcn-vue | Code dans le projet, modifiable à volonté |
| **Stack Nuxt** | shadcn-vue | Compatible Nuxt 4, Vue 3 Composition API |

### Implementation Approach

**Phase 1 — Fondations (MVP)**
```
src/
├── components/
│   ├── ui/           # shadcn-vue components
│   │   ├── button/
│   │   ├── card/
│   │   ├── badge/
│   │   ├── input/
│   │   └── progress/
│   ├── layout/       # Layout components
│   │   ├── Header.vue
│   │   ├── Footer.vue
│   │   └── Container.vue
│   └── article/      # Article-specific
│       ├── CodeBlock.vue
│       ├── TableOfContents.vue
│       └── ReadingProgress.vue
```

**Phase 2 — Recherche & Filtrage**
- Select, Sheet (mobile drawer), Pagination
- ArticleCard avec hover states et badges

**Phase 3 — Polish**
- Animations CSS transitions
- Tooltip pour métadonnées enrichies
- Fine-tuning responsive

### Customization Strategy

**Design Tokens (CSS Variables)**
```css
:root {
  /* Couleurs Dark Mode (seules utilisées) */
  --background: #1F1F1F;        /* Fond primaire */
  --background-secondary: #2E2E2E; /* Cartes, panneaux */
  --foreground: #FAFAFA;        /* Texte principal */
  --foreground-muted: #A6A6A6;  /* Texte secondaire */
  --accent: #14B8A6;            /* Teal - liens, boutons, actifs */
  --destructive: #F56565;       /* Erreurs */
  --success: #48BB78;           /* Confirmations */

  /* Rayons de bordure */
  --radius: 0.375rem;           /* 6px */

  /* Typographie */
  --font-sans: 'Satoshi Variable', ui-sans-serif, system-ui, sans-serif;
  --font-mono: 'JetBrains Mono Variable', ui-monospace, monospace;
}
```

**Composants Custom Prévus**
| Composant | Raison | Base |
|-----------|--------|------|
| CodeBlock | Copy button, language badge, line highlight | Shiki + custom |
| TableOfContents | Sticky, highlight active, temps/section | Custom + Tailwind |
| ReadingProgress | Barre horizontale sticky | Progress shadcn + custom |
| ArticleCard | Design spécifique sebc.dev | Card shadcn + custom |
| FeaturedArticleCard | Hero homepage | Card shadcn + custom |
| SearchFilters | Sidebar filtres combinables | Select/Sheet shadcn + custom |

**Composants shadcn-vue Utilisés Directement**
- Button (variantes: default, secondary, ghost, link)
- Badge (thème, catégorie, niveau, tags)
- Input (recherche)
- Select (dropdowns filtres)
- Sheet (panneau mobile)
- Dialog (ToC mobile)
- Progress (base pour progression)
- Tooltip (métadonnées enrichies)

## Defining User Experience

### The Defining Experience

**"Trouver du code qui marche, le copier, avancer."**

L'expérience que les utilisateurs décriront à leurs collègues :
> "J'ai trouvé ce blog, le code TypeScript marchait du premier coup. J'ai gagné 3 heures."

Cette expérience définissante guide toutes les décisions UX — chaque élément de design doit réduire le temps entre "j'ai un problème" et "j'ai une solution qui fonctionne".

### User Mental Model

**Parcours Type :**
```
[Problème code] → [Recherche] → [Trouve sebc.dev] → [Scanne TL;DR] → [Copie code] → [Retourne coder]
```

**Attentes par Persona :**

| Persona | Attente Principale | Frustration à Éviter |
|---------|-------------------|---------------------|
| **Lucas** | "Solution qui marche en 2 min" | Code qui ne compile pas |
| **Chloé** | "Comprendre, pas juste copier" | Se sentir dépassée |
| **Maxime** | "ROI clair, pas de bullshit" | Perte de temps |

**Modèle Mental Commun :**
- Le code doit être testable immédiatement
- La structure doit permettre de scanner rapidement
- La profondeur doit être accessible mais optionnelle

### Success Criteria

| Critère | Mesure | Seuil |
|---------|--------|-------|
| **Time-to-value** | Temps jusqu'au copy | < 60s |
| **Code fonctionnel** | Compile du premier coup | 100% |
| **Orientation** | Utilisateur sait où il est | ToC + progression |
| **Compréhension** | Peut expliquer le code | Commentaires inline |
| **Retour** | Visite récurrente | > 15% |

**Signaux de Succès :**
1. Lucas copie, compile, soulagement
2. Chloé comprend, eureka, empowerment
3. Maxime décide, efficacité, respect

### Pattern Analysis

**Patterns Établis Adoptés :**
- Copy button visible (Stripe, GitHub)
- ToC sticky avec highlight (Josh Comeau)
- Barre progression (Medium)
- Answer-First structure (Stack Overflow)
- Badges cliquables (Dev.to)

**Pattern Distinctif : "Pattern Onion"**

Architecture de contenu à couches permettant 3 parcours sur 1 article :

| Couche | Cible | Contenu |
|--------|-------|---------|
| **Hook** | Tous | Titre + TL;DR |
| **Quick Win** | Lucas, Maxime | Code fonctionnel |
| **Pattern** | Chloé, Lucas | Explication structurée |
| **Deep Dive** | Chloé | Sections `<details>` |
| **Comparatifs** | Maxime | Tableaux ROI |

### Experience Mechanics

**1. Initiation (0-5s)**
| Trigger | Système |
|---------|---------|
| Arrive depuis recherche | Page charge < 2.5s (LCP) |
| Première impression | TL;DR visible above-the-fold |
| Orientation | ToC visible, structure claire |

**2. Interaction (5s-3min)**
| Action | Feedback |
|--------|----------|
| Scroll vers code | Code block visible, copy button apparent |
| Clique Copy | "Copié!" 2s, icône checkmark |
| Clique ToC | Smooth scroll, section highlight |
| Ouvre details | Animation slide, chevron rotation |

**3. Completion**
| Persona | Signal Fin | Next Action |
|---------|------------|-------------|
| Lucas | Code copié | Retourne IDE |
| Chloé | Pattern compris | Bookmark/Related |
| Maxime | Décision prise | Applique |

## Visual Design Foundation

### Color System

**Mode : Dark-Only**

#### Core Palette
| Token | Value | Usage |
|-------|-------|-------|
| `--background` | `#0A0A0B` | Fond principal |
| `--background-secondary` | `#141415` | Cartes, panneaux |
| `--background-tertiary` | `#1E1E20` | Hover states |
| `--foreground` | `#FAFAFA` | Texte principal |
| `--foreground-muted` | `#A1A1AA` | Texte secondaire |
| `--border` | `#27272A` | Bordures |

#### Accent & Functional
| Token | Value | Usage |
|-------|-------|-------|
| `--accent` | `#14B8A6` | Liens, focus, CTA |
| `--success` | `#22C55E` | Confirmations |
| `--warning` | `#EAB308` | Avertissements |
| `--error` | `#EF4444` | Erreurs |

#### Pillar Colors
| Pilier | Couleur |
|--------|---------|
| IA | `#8B5CF6` (Violet) |
| Ingénierie | `#3B82F6` (Bleu) |
| UX | `#EC4899` (Rose) |

### Typography System

#### Font Families
- **Sans-serif** : Satoshi Variable (moderne, géométrique, variable font)
- **Monospace** : JetBrains Mono (code, ligatures)

#### Type Scale
| Token | Size | Line Height | Usage |
|-------|------|-------------|-------|
| `--text-base` | 16px | 1.6 | Corps |
| `--text-lg` | 18px | 1.6 | Lead |
| `--text-2xl` | 24px | 1.4 | H3 |
| `--text-3xl` | 30px | 1.3 | H2 |
| `--text-4xl` | 36px | 1.2 | H1 |
| `--text-5xl` | 48px | 1.1 | Hero |

### Spacing & Layout Foundation

#### Spacing Scale (Base 4px)
| Token | Value |
|-------|-------|
| `--space-1` | 4px |
| `--space-2` | 8px |
| `--space-4` | 16px |
| `--space-6` | 24px |
| `--space-8` | 32px |
| `--space-12` | 48px |

#### Breakpoints
| Name | Width | Columns |
|------|-------|---------|
| Mobile | < 768px | 1 |
| Tablet | 768-1023px | 2 |
| Desktop | ≥ 1024px | 3 |

#### Article Layout (Desktop)
- Content max-width : 720px
- ToC sidebar : 240px
- Gap : 48px
- Container max : 1200px

#### Border Radius
| Token | Value |
|-------|-------|
| `--radius-sm` | 4px |
| `--radius` | 6px |
| `--radius-md` | 8px |
| `--radius-lg` | 12px |

### Accessibility Considerations

| Requirement | Standard | Implementation |
|-------------|----------|----------------|
| Text contrast | WCAG AA 4.5:1 | 7.2:1 minimum |
| Focus visible | WCAG 2.1 | 2px ring accent |
| Touch targets | 44px min | 48px mobile |
| Reduced motion | `prefers-reduced-motion` | Supported |
| Screen readers | ARIA labels | All interactive elements |

## Design Direction Decision

### Design Directions Explored

**Direction A : "Technical Clarity"** (Stripe Docs + Josh Comeau)
- Layout content-centered avec ToC sidebar
- Densité aérée, respiration généreuse
- Accents teal subtils, couleurs piliers sur badges
- Typographie grande (18px corps), hiérarchie claire

**Direction B : "Dense & Efficient"** (Tailwind Docs + MDN)
- Layout compact, navigation sidebar gauche
- Haute densité d'information
- Code inline avec texte

**Direction C : "Modern Gradient"** (Nuxt.com + Vercel)
- Gradients et animations CSS
- Impressionnant visuellement
- Plus complexe à implémenter

### Chosen Direction

**Direction A : "Technical Clarity"**

Approche visuelle sobre et professionnelle inspirée de Stripe Docs et Josh Comeau, optimisée pour la lecture technique long-form.

### Design Rationale

| Critère | Justification |
|---------|---------------|
| **Lisibilité** | Corps 18px, line-height 1.6, largeur 720px = optimal lecture |
| **Code First** | Blocs code proéminents, copy button visible |
| **Orientation** | ToC sticky, progress bar, structure claire |
| **Performance** | CSS minimal, pas de gradients = Lighthouse 100 |
| **Crédibilité** | Style professionnel = confiance technique |
| **Maintenabilité** | Simplicité = maintenable en solo |

### Implementation Approach

**Layout Principal**
```
Header (sticky, 64px)
├── Logo (gauche)
├── Navigation (centre) : Articles, Thèmes▼, Catégories▼
└── Actions (droite) : Search, Lang Switch

Progress Bar (fixed top, 3px)

Main Content (max 1200px, centré)
├── Article Content (720px)
│   ├── Badges (thème, catégorie, niveau)
│   ├── H1 Title
│   ├── TL;DR Box
│   ├── Code Blocks
│   └── Sections avec H2/H3
└── ToC Sidebar (240px, sticky)
    ├── Section links
    └── Active highlight

Footer (padding 48px)
```

**Composants Clés**
| Composant | Style |
|-----------|-------|
| Header | `bg-background/80 backdrop-blur border-b` |
| Code Block | `bg-background-secondary border rounded-lg` |
| Badge | `bg-transparent border text-sm rounded-full px-3 py-1` |
| ToC Link | `text-foreground-muted hover:text-accent` |
| ToC Active | `text-accent border-l-2 border-accent` |
| Card | `bg-background-secondary border hover:border-accent/50` |

## User Journey Flows

### Journey 1: Quick Win Reading (Lucas)

**Goal:** Find working code and copy it in under 60 seconds.

**Flow:**
1. Arrive from search → Page loads < 2.5s
2. See title + TL;DR → Scan for relevance (5-10s)
3. Scroll to code block → Identify solution (10-20s)
4. Click Copy → Feedback "Copié!" (1s)
5. Paste in IDE → Code compiles
6. Success → Relief + possible bookmark

**Critical UX Elements:**
- LCP < 2.5s
- TL;DR above fold
- Copy button always visible
- Code with inline comments

### Journey 2: Deep Dive Learning (Chloé)

**Goal:** Understand the "why" behind a pattern.

**Flow:**
1. Read TL;DR → Get context
2. Encounter code → Check understanding
3. If confused → Read inline comments
4. Need more depth → Open `<details>` sections
5. "Eureka" moment → Pattern understood
6. Continue to "Approfondir" → Build complete understanding
7. Success → Can explain to colleague

**Critical UX Elements:**
- Progressive disclosure via `<details>`
- "Why this choice?" sections
- Non-intimidating structure
- Supportive tone

### Journey 3: Discovery & Filtering

**Goal:** Find relevant article in catalog.

**Flow:**
1. Enter site (homepage, /articles, or badge click)
2. See sidebar filters
3. Select filters (theme, level, category)
4. Results update instantly (no reload)
5. URL reflects filters (shareable)
6. Find relevant article → Click card
7. Navigate to article

**Critical UX Elements:**
- Instant filter updates
- URL synchronization
- Filter counts displayed
- Reset button when active

### Journey 4: Bilingual Navigation

**Goal:** Switch language without friction.

**Flow:**
1. Click language switch (FR/EN)
2. System checks article availability
3. If translated → Load translated version
4. If not → Show message + option to stay
5. Preserve filters and scroll position
6. Continue reading in new language

**Critical UX Elements:**
- Graceful handling of untranslated content
- Filter preservation
- Position preservation when possible

### Journey Patterns

**Immediate Feedback Pattern**
| Action | Feedback | Duration |
|--------|----------|----------|
| Copy | "Copié!" | 2s |
| Filter | Count update | Instant |
| Details open | Chevron rotate | 200ms |

**Progressive Disclosure Pattern**
| Level | Content | Trigger |
|-------|---------|---------|
| 1 | TL;DR | Default visible |
| 2 | Code + explanation | Scroll |
| 3 | "Why?" details | Explicit click |
| 4 | FAQ, resources | End of article |

**Continuous Orientation Pattern**
- Progress bar (article position)
- ToC highlight (active section)
- Breadcrumbs (navigation path)
- URL sync (app state)

### Flow Optimization Principles

1. **Minimize Steps to Value** — Lucas gets code in < 60s
2. **Reduce Cognitive Load** — One decision at a time
3. **Provide Clear Progress** — Always know where you are
4. **Handle Errors Gracefully** — No dead ends
5. **Create Eureka Moments** — Strategic "Ah!" opportunities

## Component Strategy

### Design System Components

**shadcn-vue Components (Direct Use)**
- Button, Badge, Input, Select, Checkbox
- Sheet, Dialog, Tooltip, Separator
- Skeleton, Collapsible, Progress (base)

**Components to Extend**
- Card → ArticleCard, FeaturedArticleCard
- Progress → ReadingProgress

### Custom Components

#### CodeBlock
- **Purpose:** Syntax-highlighted code with copy button
- **Props:** `code`, `language`, `filename?`, `highlightLines?`
- **States:** Default, Hover, Copied (2s feedback)
- **Tech:** Shiki for syntax highlighting

#### TableOfContents
- **Purpose:** Sticky article navigation with active highlight
- **Props:** `headings[]`, `activeId`
- **Behavior:** IntersectionObserver for active detection
- **Position:** Sticky sidebar (desktop), modal (mobile)

#### ReadingProgress
- **Purpose:** Visual reading progress indicator
- **Position:** Fixed top, 3px height, accent color
- **Behavior:** Scroll-based calculation

#### ArticleCard
- **Purpose:** Article preview in listings
- **Content:** Badges, title, description excerpt, meta
- **States:** Default, Hover (border accent)

#### ArticleHeader
- **Purpose:** Article page header with metadata
- **Content:** Badges row, H1, reading time, date

#### SearchFilters
- **Purpose:** Combinable filter sidebar
- **Behavior:** Instant update, URL sync
- **Mobile:** Sheet drawer

#### ProseDetails
- **Purpose:** Styled `<details>` for "Approfondir" sections
- **Animation:** CSS transition on open/close

### Component Implementation Strategy

**Principles:**
1. Use shadcn-vue tokens for all custom components
2. Ensure WCAG AA accessibility on all components
3. Support keyboard navigation throughout
4. Optimize for Lighthouse 100 (minimal JS)

**File Structure:**
```
components/
├── ui/              # shadcn-vue (installed)
├── layout/          # Header, Footer, Container
├── article/         # CodeBlock, ToC, Progress, Header
├── cards/           # ArticleCard, FeaturedCard
└── search/          # Filters, Pagination
```

### Implementation Roadmap

**Phase 1 — Core (MVP Blocking)**
- CodeBlock, TableOfContents, ReadingProgress
- ArticleCard, ArticleHeader

**Phase 2 — MVP Complete**
- FeaturedArticleCard, SearchFilters
- LanguageSwitch, Pagination, ProseDetails

**Phase 3 — Polish**
- Skeleton loaders, Toast, Breadcrumbs
