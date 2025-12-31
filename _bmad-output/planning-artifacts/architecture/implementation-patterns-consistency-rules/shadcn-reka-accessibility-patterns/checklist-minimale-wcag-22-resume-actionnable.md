# Checklist Minimale WCAG 2.2 (Résumé Actionnable)

Critères **indispensables** avant toute mise en production :

| Critère | Exigence | Vérification |
|---------|----------|--------------|
| **Taille cibles** | ≥ 24×24 CSS pixels | `min-width: 24px; min-height: 24px` sur interactifs |
| **Focus visible** | Contraste 3:1 entre états | Double-ring ou outline 2px visible |
| **DialogTitle** | Toujours présent | Visible ou `<VisuallyHidden>` |
| **prefers-reduced-motion** | Respecté | CSS filet de sécurité global |
| **IDs ARIA** | Stables SSR/client | `useId()` de Vue 3.5+ |
| **Score Lighthouse** | ≥ 95% accessibilité | CI avec assertions |

## Commandes de vérification rapide

```bash
# Tests automatisés (détectent ~30% des problèmes)
pnpm test:a11y              # Vitest + axe-core
pnpm test:e2e:a11y          # Playwright + axe

# Lighthouse CI
pnpm lhci autorun

# ESLint accessibilité
pnpm lint --filter vuejs-accessibility
```

## Outils de tests manuels (70% restants)

| Outil | Usage |
|-------|-------|
| **NVDA** (Windows) | Test principal lecteur d'écran |
| **VoiceOver** (macOS) | Test secondaire |
| **Chrome DevTools > Accessibility** | Inspection ARIA temps réel |
| **axe DevTools extension** | Audit page complète |
| **WAVE extension** | Audit visuel (overlay sur la page) |

**Règle** : Les tests automatisés sont nécessaires mais insuffisants. Prévoir des tests manuels avec lecteurs d'écran avant chaque release majeure.
