# 6. Métriques GEO

## 6.1 Qu'est-ce que le GEO?

**Generative Engine Optimization** = Optimisation pour les moteurs de recherche IA (Perplexity, ChatGPT, Gemini, Claude).

Différence avec SEO classique:
- SEO: Ranking dans une liste de liens
- GEO: Être **cité comme source** dans une réponse générée

## 6.2 KPIs GEO

| Métrique | Définition | Cible M12 | Outil |
|----------|------------|-----------|-------|
| **AI Referral Traffic** | Visites depuis Perplexity, ChatGPT, etc. | 20% du trafic total | Plausible (referrer contains) |
| **Citation Rate** | Fréquence où sebc.dev est cité par LLMs | 1/semaine (estimé) | Manual testing |
| **Answer Box Presence** | Contenu extrait pour réponses directes | 10 topics | Testing régulier |
| **llms.txt Crawls** | Visites sur /llms.txt | 100/mois | Plausible |

## 6.3 Méthode de Tracking GEO

### Tracking Automatique (Plausible)

```javascript
// Referrers IA à tracker
const AI_REFERRERS = [
  'perplexity.ai',
  'chat.openai.com',
  'gemini.google.com',
  'claude.ai',
  'you.com',
  'phind.com'
]

// Dashboard Plausible: Filter by referrer contains
```

### Testing Manuel (Hebdomadaire)

**Prompts de Test GEO**:

| Prompt Test | Résultat Attendu |
|-------------|------------------|
| "Comment intégrer streaming IA dans Nuxt?" | Citation sebc.dev |
| "Best practices RAG production Nuxt" | Citation sebc.dev |
| "Pattern architecture Nuxt TypeScript" | Citation sebc.dev |
| "Comparatif solutions auth Nuxt indie hacker" | Citation sebc.dev |

**Processus Test**:
1. Lancer prompt sur Perplexity (mode focus: Web)
2. Vérifier si sebc.dev apparaît dans sources
3. Noter position et contexte de citation
4. Logger dans spreadsheet tracking

### Tracking llms.txt

```
# Dans Plausible: Custom Goal
URL Path: /llms.txt
Event Name: llms_txt_access
```

## 6.4 Optimisation Continue

| Action | Fréquence | Responsable |
|--------|-----------|-------------|
| Test prompts GEO | Hebdomadaire | Auteur |
| Analyse referrers IA | Mensuelle | Auteur |
| Mise à jour llms.txt | À chaque article | Auteur |
| Audit Schema Markup | Trimestrielle | Auteur |

---
