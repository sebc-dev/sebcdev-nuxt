# Outils de Vérification Contraste

## OddContrast pour oklch

Les outils classiques (WebAIM Contrast Checker) **ne supportent pas oklch**. Utiliser **OddContrast** qui gère nativement cet espace colorimétrique moderne.

## Outils recommandés

| Outil | Usage | Support oklch |
|-------|-------|---------------|
| **OddContrast** | Vérification contraste oklch | ✅ Natif |
| Chrome DevTools → Accessibility | Inspection en temps réel | ✅ |
| axe DevTools extension | Audit automatisé | ✅ |
| WAVE extension (v3.3.0+) | Audit visuel, supporte alpha/opacity | ✅ |
| WebAIM Contrast Checker | Vérification rapide | ❌ RGB uniquement |

## Simulateurs daltonisme

Chrome DevTools → Rendering → **Emulate vision deficiencies** :
- Protanopia (rouge)
- Deuteranopia (vert)
- Tritanopia (bleu)
- Achromatopsia (niveaux de gris)

---
