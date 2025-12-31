# Quand reconsidérer MiniSearch

## Garder MiniSearch quand :

- **Volume < 500 articles** — Index < 150KB, latence < 10ms
- **Boosting personnalisé requis** — Field, document, et term boost
- **Algorithmes de ranking custom** — Contrôle total sur le scoring
- **Index dynamique runtime** — Ajout/suppression sans rebuild
- **Edge functions planifiées** — Compatible Cloudflare Workers

## Migrer vers Pagefind quand :

- **Volume > 500 articles** — Index chunked plus efficace
- **Zero-configuration multilingue** — Détection auto via `<html lang="">`
- **Minimiser le bundle JS** — Pas de lib à charger initialement
- **Trafic search imprévisible** — Chunked loading scale mieux
- **Contenu change = rebuild anyway** — Pagefind s'intègre naturellement

## Matrice de décision

| Critère | MiniSearch | Pagefind |
|---------|------------|----------|
| **< 200 articles** | ✅ Recommandé | Overkill |
| **200-500 articles** | ✅ Recommandé | Alternative viable |
| **500-2000 articles** | ⚠️ + Web Worker | ✅ Recommandé |
| **> 2000 articles** | ❌ | ✅ Recommandé |
| **Custom ranking** | ✅ | ❌ |
| **Runtime updates** | ✅ | ❌ |
| **Setup time** | Moyen | Minimal |

**Pour sebc.dev (< 500 articles prévus)** : MiniSearch reste le choix optimal avec latence < 10ms et contrôle total sur le boosting.

---
