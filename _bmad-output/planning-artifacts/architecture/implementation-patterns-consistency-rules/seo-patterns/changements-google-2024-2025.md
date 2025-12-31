# Changements Google 2024-2025

Récapitulatif des évolutions impactant les rich results Schema.org :

| Fonctionnalité | Statut | Date | Impact |
|----------------|--------|------|--------|
| **Sitelinks Search Box** | Supprimé | Nov 2024 | `SearchAction` n'affecte plus les SERPs |
| **FAQ Rich Results** | Limité | Août 2023 | Réservé aux sites gov/santé uniquement |
| **How-To Rich Results** | Supprimé | 2023 | Plus de rich snippets pour tutoriels |
| **Article author.url** | Ajouté | 2024 | Améliore la désambiguïsation des auteurs |

**Implications pour le projet :**
- Ne pas implémenter `SearchAction` (inutile)
- Ne pas implémenter `FAQPage` (blogs techniques non éligibles)
- Se concentrer sur `Article`, `BreadcrumbList`, `Person`, `WebSite`
