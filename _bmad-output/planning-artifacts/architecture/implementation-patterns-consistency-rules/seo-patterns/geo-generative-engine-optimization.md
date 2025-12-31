# GEO (Generative Engine Optimization)

## Ratio optimal de contenu

La densité factuelle détermine les chances de citation par les moteurs IA. Ratio recommandé :

| Métrique | Ratio optimal | Exemple (3000 mots) |
|----------|---------------|---------------------|
| Statistiques | 1 / 150-200 mots | 15-20 stats sourcées |
| Réponses directes | 6 / 1000 mots | 18 answer capsules |
| Citations externes | 5-8 / article | Sources .edu, .gov, académiques |

**Pattern quotable fact :**

```markdown
❌ Inefficace : "Notre solution améliore considérablement les performances."

✅ GEO-optimisé : "Notre solution réduit le temps de réponse de 4 minutes
à 45 secondes, augmentant l'efficacité de 82% (Source, 2025)."
```

## Answer Capsules

L'answer capsule est une réponse autonome placée après un H2, conçue pour extraction verbatim par les LLM.

| Contexte | Longueur | Liens |
|----------|----------|-------|
| Réponse principale (après H1) | 40-60 mots | **Aucun** |
| Answer capsule avec preuves | 80-120 mots | **Aucun** |
| Paragraphe de développement | 150-200 mots | Oui (sous la capsule) |

**Découverte critique** : **91%** des capsules citées par les IA ne contiennent aucun lien. Les liens suggèrent que la réponse autoritaire est ailleurs.

```markdown
# Comment optimiser les Core Web Vitals ?

Les Core Web Vitals s'optimisent en trois axes : LCP sous 2.5s via
lazy loading images et preload fonts, INP sous 200ms via code splitting
et hydratation différée, CLS à 0 via dimensions explicites sur les médias.

Paragraphe de développement avec [liens vers sources](url) et détails
techniques additionnels placés ici, sous la capsule.
```

## H2 formulés en questions

Les H2 en format question correspondent aux requêtes utilisateurs vers les assistants IA :

| À éviter | Préférer |
|----------|----------|
| "Conseils performance" | "Comment améliorer les performances ?" |
| "Démarrage" | "Quelles sont les étapes pour commencer ?" |
| "Nos solutions" | "Quelle solution choisir pour [cas d'usage] ?" |
| "Configuration" | "Comment configurer [fonctionnalité] ?" |

## Paramètres structurels Markdown

Pour Nuxt Content 3, respectez ces limites pour un parsing optimal par les LLM :

| Élément | Limite | Raison |
|---------|--------|--------|
| Paragraphes | Max 120 mots (idéal 40-60) | Extraction autonome |
| Phrases | 15-20 mots max | Parsing propre |
| H1 | 1 unique par page | Titre dans frontmatter |
| H2 | Questions alignées intention utilisateur | Correspondance requêtes IA |
| H3 | Détails de support, sous-réponses | Hiérarchie claire |

## TL;DR / Quick Answer (BLUF)

Le principe BLUF (Bottom Line Up Front) place la réponse clé en introduction :

```markdown
---
title: "Comment implémenter le dark mode dans Nuxt 4 ?"
---

# TL;DR

Le dark mode dans Nuxt 4 s'implémente via @nuxtjs/color-mode avec
la classe .dark de Tailwind. Configuration en 3 étapes : installer
le module, définir classSuffix vide, ajouter le script anti-FOUC
dans app.head. Temps d'implémentation : 15 minutes.

# Étape 1 : Installation du module

[Contenu détaillé...]
```

## Éléments bio auteur (E-E-A-T)

Les IA recherchent des preuves de crédentials avant de citer. Bio auteur requise :

1. **Nom complet** - Réel et vérifiable
2. **Crédentials** - Diplômes, certifications
3. **Titre et affiliation** - Poste actuel
4. **Expérience chiffrée** - Années, domaines
5. **Résultat concret** - Cas d'étude, accomplissement
6. **Liens profils** - LinkedIn, GitHub, site personnel

**Pattern page auteur :**

```markdown
<!-- content/auteurs/sebastien-c.md -->
---
name: "Sébastien C."
title: "Lead Developer"
image: "/images/authors/sebastien.jpg"
credentials: "10 ans d'expérience Vue.js/Nuxt"
social:
  github: "https://github.com/sebcdev"
  linkedin: "https://linkedin.com/in/sebcdev"
---

Développeur full-stack spécialisé Vue.js et Nuxt depuis 2015.
Contributeur open source avec 50+ PR merged sur l'écosystème Nuxt.
Auteur de 3 modules Nuxt totalisant 10K+ téléchargements mensuels.
```

## Différences entre plateformes IA

Chaque moteur génératif a des préférences distinctes :

| Critère | ChatGPT Search | Perplexity | Google AI Overviews |
|---------|---------------|------------|---------------------|
| **Longueur préférée** | 2500-3000 mots | Modérée, haute densité | Variable |
| **Poids fraîcheur** | Faible | **Très élevé** (90 jours) | Modéré |
| **Ton** | Encyclopédique | Conversationnel | Autoritaire |
| **Index source** | **Bing** (73% corrélation) | Temps réel | Index Google |
| **Signal fort** | Structure Wikipedia | Exemples communautaires | SEO traditionnel |

**Implications pratiques :**
- **Perplexity** : Contenu >9 mois = -35% citations. Mettre à jour `updatedAt` régulièrement
- **ChatGPT** : Optimiser pour Bing optimise aussi ChatGPT Search
- **Google AI** : 92% des citations viennent du top 10 SEO existant

## Métriques GEO vs SEO

| SEO traditionnel | Équivalent GEO |
|------------------|----------------|
| Position 1-10 | Inclusion dans la citation (cité ou non) |
| Taux de clic (CTR) | Fréquence et proéminence de citation |
| Trafic organique | Trafic référé IA (`utm_source=chatgpt.com`) |
| Profil de backlinks | Vélocité des mentions de marque |

**Tracking pratique :**
- Google Analytics : Trafic référent `perplexity.ai`, `chatgpt.com`
- ChatGPT ajoute automatiquement `utm_source=chatgpt.com`
- Logs serveur : Activité GPTBot, PerplexityBot, ClaudeBot

## Composant MDC ::faq-item

Pattern pour FAQ sémantique dans Nuxt Content 3 (note : FAQ rich results limités aux sites gov/santé, mais utile pour structure) :

```vue
<!-- components/content/FaqItem.vue -->
<script setup lang="ts">
defineProps<{
  question: string
}>()
</script>

<template>
  <details class="border-b border-gray-200 py-4">
    <summary class="cursor-pointer font-medium text-lg">
      {{ question }}
    </summary>
    <div class="mt-2 text-gray-600">
      <slot />
    </div>
  </details>
</template>
```

**Usage dans Markdown :**

```markdown
# FAQ

::faq-item{question="Comment installer nuxt-schema-org ?"}
Exécutez `npx nuxt module add schema-org` dans votre projet Nuxt.
Le module s'ajoute automatiquement à nuxt.config.ts et génère
les schemas JSON-LD au build time.
::

::faq-item{question="Le module supporte-t-il Nuxt 4 ?"}
Oui, nuxt-schema-org v5.0.10 est pleinement compatible avec
Nuxt 4 et compatibilityDate. Aucune configuration spéciale requise.
::
```
