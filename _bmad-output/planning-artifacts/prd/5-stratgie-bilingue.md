# 5. Stratégie Bilingue

## 5.1 Décision : Bilingue Dès le MVP

**100% des articles publiés en FR et EN simultanément**

| Principe | Description |
|----------|-------------|
| FR-first | Rédaction originale en français (langue maternelle, authenticité) |
| Assistance IA rédaction | L'IA assiste la rédaction, ne génère pas d'articles entiers |
| Traduction IA | Agent IA traduit l'article FR vers EN |
| Relecture humaine | 15-30 min de relecture/ajustement par article traduit |

## 5.2 Workflow Publication

```
1. Rédaction FR (assistée IA) ──────────────────────┐
2. Publication article FR                            │ Simultané
3. Traduction IA (FR → EN)                          │
4. Relecture rapide (15-30 min)                     │
5. Publication article EN ──────────────────────────┘
```

**Impact temps** : +15-30 min/article (négligeable sur 6h de rédaction)

## 5.3 Justification Stratégique

| Avantage | Impact |
|----------|--------|
| Double audience dès J1 | Trafic FR + EN, SEO international |
| GEO maximisé | LLMs citent sources bilingues plus facilement |
| Authenticité préservée | Voix FR originale, pas de traduction mécanique |
| Coût marginal faible | 15-30 min vs rédaction native EN (2-3h) |

## 5.4 Qualité Traduction

| Type de contenu | Approche |
|-----------------|----------|
| **Code/Technique** | Traduction IA directe (termes techniques universels) |
| **Explications** | Traduction IA + relecture légère |
| **Opinion/Ton personnel** | Traduction IA + ajustement ton (préserver voix) |

**Outil prévu** : Claude/GPT avec prompt spécialisé "tech blog translation"

---
