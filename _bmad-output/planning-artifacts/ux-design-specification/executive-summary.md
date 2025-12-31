# Executive Summary

## Project Vision

**sebc.dev** est un blog technique bilingue (FR/EN) documentant en temps réel le parcours d'un développeur autodidacte à l'intersection de trois piliers : IA (40%), Ingénierie Logicielle (30%), et UX (30%).

Le positionnement "R&D Engineering in Public" se différencie des blogs experts traditionnels par la transparence totale sur le processus d'apprentissage — partage des résultats validés ET des échecs documentés.

La thèse centrale : face à l'IA omniprésente fin 2025, les développeurs doivent maîtriser 3 domaines complémentaires plutôt que se spécialiser dans le code pur.

## Target Users

**Lucas — "Le Mid-Level Bloqué" (Primary)**
- Développeur Fullstack 27 ans, 4 ans d'expérience, startup SaaS
- Friction : 70% du temps en "glue code" IA, manque de vision architecturale
- Objectif UX : Time-to-value < 60 secondes, code copy-paste fonctionnel immédiatement
- Succès : "Ce mec gère les edge-cases que j'avais oubliés. C'est du code de prod."

**Chloé — "L'Apprentie Copilote" (Secondary)**
- Développeuse Junior 24 ans, bootcamp 2024, 10 mois de code
- Friction : "Hollow Senior" - code avec IA sans comprendre, syndrome imposteur violent
- Objectif UX : Pattern discovery < 3 minutes, comprendre le "pourquoi"
- Succès : "Je savais ce qu'il fallait faire! Je ne suis pas nulle!"

**Maxime — "L'Architecte Frankenstein" (Secondary)**
- Solopreneur 32 ans, ex-Senior Dev, 3 micro-SaaS + freelance
- Friction : Integration Hell, 15+ services, ~400€/mois gaspillés
- Objectif UX : Vue d'ensemble rapide, ROI clair, comparatifs coûts
- Succès : "Ce gars me fait gagner de l'argent."

## Key Design Challenges

1. **Pattern Onion Multi-Audience** — Servir 3 personas avec 1 article sans confusion ni dilution. Lucas lit TL;DR + Code, Chloé lit Pattern + Deep Dive, Maxime lit Hook + Comparatifs.

2. **Time-to-Value Extrême** — Lucas doit trouver une solution copy-paste en < 60 secondes. La structure Answer-First impose la réponse dans les 100 premiers mots.

3. **Révélation Progressive** — Chloé a besoin de profondeur pédagogique, mais sans submerger Lucas qui veut juste le code. Les sections `<details>` doivent être clairement signalées mais non-intrusives.

4. **Bilingue Sans Friction** — 100% des articles en FR et EN simultanément. Le switch de langue doit préserver la position de lecture et les filtres actifs.

5. **Performance Lighthouse 100** — Objectif non-négociable. Impact direct sur le design : lazy loading, optimisation images, bundle minimal.

## Design Opportunities

1. **Table des Matières Intelligente** — ToC sticky avec highlight section active ET temps de lecture par section. Permet à chaque persona de naviguer selon son objectif.

2. **Badges Cliquables Contextuels** — Chaque badge (thème, catégorie, tag, niveau) est un point d'entrée vers la recherche filtrée. Découverte par sérendipité.

3. **Métriques Comportementales** — Copy Rate (Lucas), Scroll Depth (Pattern Onion validation), Temps par section (Chloé engagement). Analytics comme outil de validation UX.

4. **Structure Answer-First GEO-Ready** — Optimisation pour les IA de recherche (Perplexity, ChatGPT). Premier paragraphe = réponse directe citée par les LLMs.
