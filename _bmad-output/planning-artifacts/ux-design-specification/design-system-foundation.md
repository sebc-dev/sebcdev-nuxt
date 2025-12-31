# Design System Foundation

## Design System Choice

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

## Rationale for Selection

| Critère | Décision | Justification |
|---------|----------|---------------|
| **Accessibilité** | shadcn-vue + Reka UI | Radix primitives = WCAG AA natif |
| **Performance** | shadcn-vue | Tree-shakable, pas de runtime CSS-in-JS |
| **Temps Dev (Solo)** | shadcn-vue | Copy-paste ready, moins de code custom |
| **Dark Mode** | Tailwind `dark:` | Système natif, cohérent |
| **Customisation** | shadcn-vue | Code dans le projet, modifiable à volonté |
| **Stack Nuxt** | shadcn-vue | Compatible Nuxt 4, Vue 3 Composition API |

## Implementation Approach

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

## Customization Strategy

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
