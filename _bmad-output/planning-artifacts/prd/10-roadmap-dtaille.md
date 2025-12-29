# 10. Roadmap Détaillée

## 10.1 Vue d'Ensemble

```
M0          M6          M9          M12         M18         M24
│           │           │           │           │           │
▼           ▼           ▼           ▼           ▼           ▼
┌───────────┬───────────┬───────────┬───────────┬───────────┐
│    MVP    │   GROWTH  │  PRODUCTS │  SCALING  │ AUTHORITY │
│           │           │           │           │           │
│ • 17 art. │ • 18 art. │ • Starter │ • Premium │ • Communty│
│ • 500 UV  │ • 2k UV   │   Kit 49€ │   79-149€ │ • Discord │
│ • 50 NL   │ • 150 NL  │ • NL auto │ • Cours?  │ • Sponsors│
└───────────┴───────────┴───────────┴───────────┴───────────┘
```

## 10.2 M6 — Fin MVP

### Livrables Techniques

| Livrable | Description | Statut |
|----------|-------------|--------|
| Blog Nuxt fonctionnel | SSR, SEO, bilingue | ✅ Requis |
| 17 articles publiés | 6 IA, 5 Ing., 4 UX, 3 Cross | ✅ Requis |
| ToC Interactive | Sidebar sticky, progress bar | ✅ Requis |
| Code Blocks | Syntax + Copy + Badge | ✅ Requis |
| Analytics | Plausible avec events custom | ✅ Requis |
| llms.txt | Fichier contexte LLMs | ✅ Requis |
| Schema Markup | TechArticle, FAQ | ✅ Requis |

### Métriques Cibles

| Métrique | Cible M6 |
|----------|----------|
| Visiteurs Uniques/mois | 500 |
| Temps lecture moyen | > 3 min |
| Scroll Depth | > 60% |
| Copy Rate | > 10% |
| AI Referral Traffic | 10% |

> **Note** : La newsletter est prévue comme première itération post-MVP. Les objectifs d'abonnés seront définis lors de cette phase.

## 10.3 M9 — Automation

### Livrables

| Livrable | Description | Investissement |
|----------|-------------|----------------|
| Newsletter Automation | Séquence bienvenue, weekly digest | 2 jours |
| Cross-posting | Dev.to, Medium, Hashnode | 1 jour/article |
| Template Preparation | Nuxt AI Starter Kit (structure) | 5 jours |
| Documentation Starter | README, docs, exemples | 3 jours |

### Métriques Cibles

| Métrique | Cible M9 |
|----------|----------|
| Visiteurs Uniques/mois | 2 000 |
| Newsletter abonnés | 150 |
| Taux ouverture NL | > 45% |
| Articles total | 26 |

## 10.4 M12 — Premier Produit

### Livrables

| Livrable | Description | Prix | Cible Ventes |
|----------|-------------|------|--------------|
| Nuxt AI Starter Kit | Template complet IA + Arch + UX | 49€ | 10/mois |
| Documentation Kit | Docs complètes + tutoriels vidéo | Inclus | - |
| Support | GitHub Issues + Discord gratuit | Inclus | - |

### Contenu du Starter Kit

```
nuxt-ai-starter/
├── README.md
├── nuxt.config.ts
├──
├── composables/
│   ├── useLLMStream.ts
│   ├── useRAG.ts
│   └── useAuth.ts
├──
├── server/
│   ├── api/
│   │   ├── chat.ts
│   │   └── embed.ts
│   └── services/
│       ├── llm.service.ts
│       └── vector.service.ts
├──
├── components/
│   ├── ChatInterface.vue
│   ├── StreamingText.vue
│   └── design-system/
├──
└── docs/
    ├── architecture.md
    ├── deployment.md
    └── customization.md
```

### Métriques Cibles

| Métrique | Cible M12 |
|----------|----------|
| Visiteurs Uniques/mois | 5 000 |
| Newsletter abonnés | 500 |
| MRR Produits | 100€ |
| Inbound Consulting | 2-3/mois |
| Backlinks Qualité | 20 |

## 10.5 M18 — Expansion Produits

### Nouveaux Livrables

| Livrable | Description | Prix | Cible |
|----------|-------------|------|-------|
| Nuxt Enterprise Template | DDD/Hexagonal complet | 79€ | 5/mois |
| Testing Boilerplate | Unit/Integration/E2E setup | 49€ | 5/mois |
| CI/CD Pipelines | GitHub Actions templates | 39€ | 5/mois |
| Bundle Complet | Tous les templates | 149€ | 3/mois |

### Exploration

| Exploration | Décision M18 | Investment |
|-------------|--------------|------------|
| Cours Vidéo | Go/No-Go basé demande | 2-3 mois si Go |
| SaaS Outil Dev | Prototype si demande | 1 mois prototype |
| Consulting Structuré | Offre packagée | 1 semaine setup |

### Métriques Cibles

| Métrique | Cible M18 |
|----------|----------|
| Visiteurs Uniques/mois | 8 000 |
| Newsletter abonnés | 800 |
| MRR Produits | 300€ |
| MRR Total (+ Consulting) | 800€ |

## 10.6 M24 — Autorité Établie

### Livrables

| Livrable | Description | Prix/Modèle |
|----------|-------------|-------------|
| Communauté Discord | 3 channels (#ia, #arch, #ux) | 5€/mois |
| Newsletter Premium | Deep-dives hebdo exclusifs | 5€/mois |
| Sponsors Blog | 1-2 sponsors techniques alignés | 500-1000€/mois |
| Cours Vidéo (optionnel) | "IA + Clean Code + UX" bundle | 150€ |

### Métriques Cibles Vision

| Métrique | Cible M24 |
|----------|----------|
| Visiteurs Uniques/mois | 10 000+ |
| Newsletter abonnés | 1 000+ |
| MRR Total | 1 000€ |
| Reconnaissance | Référence francophone Nuxt/IA/UX |
| Invitations Podcasts | 3+/an |
| Citations Nuxt Weekly | 3+/an |

---
