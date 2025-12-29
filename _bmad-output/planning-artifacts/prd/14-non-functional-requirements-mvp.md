# 14. Non-Functional Requirements (MVP)

Cette section définit les **exigences de qualité** du système — comment il doit performer, pas ce qu'il fait.

**Objectif Lighthouse : 100/100/100/100** (Performance, Accessibility, Best Practices, SEO)

## 14.1 Performance

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR1 | Largest Contentful Paint (LCP) | < 2.5s |
| NFR2 | First Input Delay (FID) | < 100ms |
| NFR3 | Cumulative Layout Shift (CLS) | < 0.1 |
| NFR4 | Time to First Byte (TTFB) | < 200ms |
| NFR5 | Score Lighthouse Performance | ≥ 95 (viser 100 si effort raisonnable) |

## 14.2 Accessibilité

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR6 | Conformité WCAG | Niveau AA (2.1) |
| NFR7 | Score Lighthouse Accessibility | 100 |
| NFR8 | Navigation clavier complète | 100% des fonctionnalités |
| NFR9 | Contraste des couleurs (mode sombre) | Ratio ≥ 4.5:1 (texte normal) |

## 14.3 SEO Technique

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR10 | Score Lighthouse SEO | 100 |
| NFR11 | Toutes les pages indexables | 100% (sauf explicitement exclues) |
| NFR12 | Temps de crawl par page | < 500ms |
| NFR13 | Validation Schema.org | 0 erreur, 0 warning |

## 14.4 Best Practices

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR14 | Score Lighthouse Best Practices | 100 |

## 14.5 Compatibilité Navigateurs

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR15 | Chrome (2 dernières versions) | Support complet |
| NFR16 | Firefox (2 dernières versions) | Support complet |
| NFR17 | Safari (2 dernières versions) | Support complet |
| NFR18 | Edge (2 dernières versions) | Support complet |
| NFR19 | Internet Explorer | Exclu |

## 14.6 Disponibilité

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR20 | Uptime mensuel | ≥ 99.5% |

## 14.7 Sécurité

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR21 | HTTPS obligatoire | 100% des pages |
| NFR22 | Headers de sécurité (CSP, X-Frame-Options, etc.) | Configurés |
| NFR23 | Score Mozilla Observatory | ≥ B+ |

## 14.8 Récapitulatif NFRs

| Catégorie | Nombre de NFRs |
|-----------|----------------|
| Performance | 5 |
| Accessibilité | 4 |
| SEO Technique | 4 |
| Best Practices | 1 |
| Compatibilité Navigateurs | 5 |
| Disponibilité | 1 |
| Sécurité | 3 |
| **TOTAL** | **23 NFRs** |

---
