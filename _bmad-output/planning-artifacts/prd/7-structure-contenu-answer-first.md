# 7. Structure Contenu Answer-First

## 7.1 Principe Answer-First

**Règle d'or**: La réponse à la question du titre doit apparaître dans les 100 premiers mots.

Pourquoi:
- LLMs extraient le début pour citations
- Lecteurs scannent avant de s'engager
- Réduit taux de rebond

## 7.2 Template Article Type

```markdown
---
title: "Comment [VERBE] [SUJET] avec [TECHNOLOGIE]"
description: "[RÉPONSE DIRECTE EN UNE PHRASE - 150 CARACTÈRES MAX]"
pillar: "ia" | "engineering" | "ux"
level: "beginner" | "intermediate" | "advanced"
readingTime: 12
tags: ["nuxt", "typescript", "streaming"]
publishedAt: 2025-01-15
updatedAt: 2025-01-15
---

# Comment [VERBE] [SUJET] avec [TECHNOLOGIE]

**TL;DR**: [Réponse en 2-3 phrases. Code minimal si applicable.]

# Le Problème

[1-2 paragraphes décrivant la friction que le lecteur ressent]

# La Solution Rapide

\```typescript
// Code copy-paste qui fonctionne immédiatement
\```

[Explication en 3-4 phrases de ce que fait le code]

# Comprendre le Pattern

## Étape 1: [Action]

[Explication + code commenté]

## Étape 2: [Action]

[Explication + code commenté]

## Étape 3: [Action]

[Explication + code commenté]

# Approfondir

<details>
<summary>🧠 Pourquoi ce choix d'architecture?</summary>

[Contenu Layer 3 Pattern Onion - concepts fondamentaux]

</details>

<details>
<summary>⚠️ Pièges courants à éviter</summary>

[Liste des erreurs fréquentes et comment les éviter]

</details>

# Comparaison des Approches

| Approche | Complexité | Performance | Use Case |
|----------|------------|-------------|----------|
| Simple | ⭐ | ⚡⚡ | MVP/Solo |
| Intermédiaire | ⭐⭐ | ⚡⚡⚡ | Équipe petite |
| Avancée | ⭐⭐⭐ | ⚡⚡⚡⚡ | Scale/Enterprise |

# FAQ

## [Question fréquente 1]?

[Réponse concise]

## [Question fréquente 2]?

[Réponse concise]

# Ressources

- [Lien documentation officielle]
- [Article lié interne]
- [Repo GitHub exemple]

---

*Mis à jour le [DATE] — [Changelog si modification majeure]*
```

## 7.3 Règles de Rédaction

### Titres (H1)

| ✅ Bon | ❌ Mauvais |
|--------|-----------|
| "Comment gérer le streaming IA dans Nuxt" | "Mes réflexions sur le streaming" |
| "Résoudre les erreurs d'hydration Nuxt" | "Un problème courant" |
| "3 patterns pour intégrer LLM et legacy" | "LLM et legacy" |

**Format**: `[Comment/Pourquoi/Quand] + [Verbe Action] + [Sujet Précis] + [Contexte Tech]`

### Premier Paragraphe

```markdown
<!-- ✅ BON: Réponse immédiate -->
Pour streamer les réponses LLM dans Nuxt sans erreur TypeScript,
utilisez le composable `useLLMStream` avec typage générique et
gestion d'erreur intégrée. Voici le code complet:

<!-- ❌ MAUVAIS: Introduction vague -->
L'intelligence artificielle transforme notre façon de développer.
Dans cet article, nous allons explorer les différentes approches
possibles pour intégrer des réponses en streaming...
```

### Blocs de Code

```typescript
// ✅ BON: Commentaires contextués, TypeScript strict
interface StreamResponse {
  content: string
  done: boolean
}

// Composable pour streaming LLM avec gestion d'erreur
export const useLLMStream = () => {
  const content = ref('')
  const error = ref<Error | null>(null)
  const isStreaming = ref(false)

  // ...
}
```

```javascript
// ❌ MAUVAIS: Pas de types, pas de contexte
const useLLMStream = () => {
  const content = ref('')
  // ...
}
```

### Longueur

| Section | Longueur Cible |
|---------|----------------|
| TL;DR | 50-100 mots |
| Le Problème | 100-200 mots |
| Solution Rapide | Code + 50 mots |
| Comprendre le Pattern | 500-800 mots |
| Approfondir (details) | 200-400 mots chacun |
| FAQ | 50-100 mots par question |
| **Total Article** | 1 500 - 2 500 mots |

---
