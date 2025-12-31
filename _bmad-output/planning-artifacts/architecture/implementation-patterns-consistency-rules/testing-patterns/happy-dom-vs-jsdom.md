# happy-dom vs jsdom

| Critère | happy-dom | jsdom |
|---------|-----------|-------|
| **Vitesse** | ~3x plus rapide | Plus lent |
| **Compatibilité** | 95% des cas | 100% |
| **Recommandation Nuxt** | ✅ Par défaut | Fallback si problèmes |

**Quand utiliser jsdom :**
- Tests nécessitant `canvas` ou `SVG` complexes
- Composants avec `MutationObserver` avancés
- Si happy-dom cause des erreurs spécifiques
- **Tests d'accessibilité avec vitest-axe** (obligatoire)

**Basculer par fichier avec directive :**

```typescript
// @vitest-environment jsdom
import { axe, toHaveNoViolations } from 'vitest-axe'
import { expect } from 'vitest'

expect.extend(toHaveNoViolations)

describe('Dialog Accessibility', () => {
  it('pas de violations WCAG', async () => {
    const { container } = render(Dialog, { props: { open: true } })
    const results = await axe(container)
    expect(results).toHaveNoViolations()
  })
})
```

---
