# Design Direction Decision

## Design Directions Explored

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

## Chosen Direction

**Direction A : "Technical Clarity"**

Approche visuelle sobre et professionnelle inspirée de Stripe Docs et Josh Comeau, optimisée pour la lecture technique long-form.

## Design Rationale

| Critère | Justification |
|---------|---------------|
| **Lisibilité** | Corps 18px, line-height 1.6, largeur 720px = optimal lecture |
| **Code First** | Blocs code proéminents, copy button visible |
| **Orientation** | ToC sticky, progress bar, structure claire |
| **Performance** | CSS minimal, pas de gradients = Lighthouse 100 |
| **Crédibilité** | Style professionnel = confiance technique |
| **Maintenabilité** | Simplicité = maintenable en solo |

## Implementation Approach

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
