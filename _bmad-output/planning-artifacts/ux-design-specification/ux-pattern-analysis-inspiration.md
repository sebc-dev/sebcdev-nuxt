# UX Pattern Analysis & Inspiration

## Inspiring Products Analysis

### Josh Comeau (joshwcomeau.com)
- **Excellence** : Visualisations interactives, ton pédagogique bienveillant, progression claire
- **Patterns clés** : ToC sticky avec highlight, sections "Digging Deeper" collapsibles, code avec copy button
- **Applicable** : Ton "Learning in Public", structure progressive, ToC comportement

### Stripe Documentation
- **Excellence** : Code snippets parfaits, navigation latérale efficace, language switcher
- **Patterns clés** : Copy button avec feedback, multi-language tabs, breadcrumbs contextuels
- **Applicable** : Copy button design, feedback animation, navigation hiérarchique

### Tailwind CSS Documentation
- **Excellence** : Recherche ultra-rapide, exemples copy-paste, structure catégorisée
- **Patterns clés** : DocSearch, sidebar collapsible, dark mode cohérent
- **Applicable** : Recherche (MVP+), sidebar filtres, dark-only design

### MDN Web Docs
- **Excellence** : Profondeur progressive, compatibilité navigateur, références croisées
- **Patterns clés** : Autorité + accessibilité, structure encyclopédique
- **Applicable** : Profondeur pour Chloé, accessibilité comme standard

## Transferable UX Patterns

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

## Anti-Patterns to Avoid

| Anti-Pattern | Problème | Notre Approche |
|--------------|----------|----------------|
| Code non-copable | Frustration, perte de temps | Copy button obligatoire sur chaque bloc |
| ToC cachée | Perte d'orientation | ToC visible en permanence (desktop) |
| Intro avant valeur | Time-to-value > 60s | TL;DR en haut, Answer-First |
| Popup newsletter | Interruption, méfiance | CTA discret fin d'article uniquement |
| Mode clair default | Pas le standard dev | Dark mode fixe, pas de toggle |
| Hover-only actions | Accessibilité, mobile | Actions toujours visibles |
| Jargon non-expliqué | Junior perdu | Définitions inline ou liens |

## Design Inspiration Strategy

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
