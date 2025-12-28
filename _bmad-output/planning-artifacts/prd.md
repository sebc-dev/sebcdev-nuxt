# Product Requirements Document (PRD)
## sebc.dev — Blog Technique Multi-Piliers

---

**Document Version**: 6.0
**Date**: 2025-12-28
**Auteur**: Negus Salomon
**Statut**: Draft — En cours de validation

---

> **Note**: Les spécifications techniques et architecturales ont été extraites vers le document [architecture.md](./architecture.md).

---

## 1. Executive Summary

### 1.1 Vision Produit

**sebc.dev** est une plateforme de contenu technique documentant en temps réel le parcours d'un développeur autodidacte à l'intersection de trois piliers essentiels du développement moderne : **IA (40%)**, **Ingénierie Logicielle (30%)**, et **UX (30%)**.

### 1.2 Thèse Centrale

Face à la transformation du métier de développeur fin 2025 où l'IA devient omniprésente, les développeurs doivent **prendre de la hauteur** et maîtriser trois domaines complémentaires plutôt que se spécialiser dans le code pur, qu'ils délégueront de plus en plus à l'IA.

### 1.3 Positionnement Unique

**"R&D Engineering in Public"** — Documentation d'explorations techniques avancées avec transparence totale sur le processus.

#### Différenciation Concrète vs Concurrents

| Concurrent | Ce qu'ils font | Ce que sebc.dev fait différemment |
|------------|---------------|-----------------------------------|
| **Grafikart** (FR) | Tutoriels vidéo classiques, approche pédagogique | Articles écrits optimisés GEO + focus intersection IA/Arch/UX |
| **Josh Comeau** (EN) | Visualisations CSS/React, très polish | Stack Nuxt/Vue (niche moins saturée) + contenu FR natif |
| **ChatGPT/Perplexity** | Réponses génériques instantanées | Patterns battle-tested en production, contexte Nuxt spécifique |
| **Dev.to/Medium** | Contenu généraliste, qualité variable | Curation stricte 3 piliers, Pattern Onion multi-audience |
| **Blogs Vercel/Nuxt** | Documentation officielle | Retours terrain réels, échecs documentés, approche Learning in Public |

**Ma promesse différenciante** : "Pas un consultant qui théorise, mais un ingénieur qui teste les frontières en production et partage les résultats validés — y compris les échecs."

### 1.4 Audiences Cibles

| Segment | Priorité | Besoin Principal |
|---------|----------|------------------|
| Mid-Level Developers (Lucas) | P1 | Évoluer vers rôles architecturaux |
| Junior Developers (Chloé) | P2 | Construire fondamentaux solides |
| Indie Hackers (Maxime) | P2 | Jongler technique/produit/design |

### 1.5 Contraintes Auteur & Budget Temps

#### Disponibilité Brute Calculée

| Jour | Créneau Matin | Créneau Soir | Créneau Jour | Total/Jour |
|------|---------------|--------------|--------------|------------|
| **Lundi** | 2h (05h-07h) | 1h | - | 3h |
| **Mardi** | 2h (05h-07h) | 1h | - | 3h |
| **Mercredi** | 3h30 (05h-08h30) | 1h | - | 4h30 |
| **Jeudi** | 2h (05h-07h) | 1h | - | 3h |
| **Vendredi** | 2h (05h-07h) | 1h | - | 3h |
| **Samedi** | 3h30 (05h-08h30) | 1h | 1h | 5h30 |
| **Dimanche** | 3h30 (05h-08h30) | 1h | 1h | 5h30 |
| **TOTAL** | 18h30 | 7h | 2h | **27h30/sem** |

#### Allocation par Phase

**Phase 0 (→ Février)** : Build Mode

| Activité | Heures/sem | % |
|----------|------------|---|
| Développement Blog | ~22h | 80% |
| Veille / Learning | ~5h30 | 20% |

**Phase 1 (Février →)** : Run Mode

| Activité | Heures/sem | % |
|----------|------------|---|
| Blog (Rédaction + News + Tech) | ~11h | 40% |
| Freelance PME/TPME | ~16h30 | 60% |

#### Routine Hebdomadaire Phase 0 (Build)

| Créneau | Lun | Mar | Mer | Jeu | Ven | Sam | Dim |
|---------|-----|-----|-----|-----|-----|-----|-----|
| **Matin (Deep Work)** | 🛠️ Dev | 🛠️ Dev | 🛠️ Dev | 🛠️ Dev | 🛠️ Dev | 🛠️ Dev | 🛠️ Dev |
| **Soir (1h)** | 🧠 Learn | 🛠️ Dev | 🧠 Learn | 🧠 Learn | 🍻 OFF | 🛠️ Dev | 📅 Planif |
| **Journée WE** | - | - | - | - | - | 🛠️ Dev | 🛠️ Dev |

#### Routine Hebdomadaire Phase 1 (Run)

| Créneau | Lun | Mar | Mer | Jeu | Ven | Sam | Dim |
|---------|-----|-----|-----|-----|-----|-----|-----|
| **Matin (Deep Work)** | ✍️ Blog | ✍️ Blog | 🛠️ Freelance | 🛠️ Freelance | 🛠️ Freelance | 🛠️ Freelance | ✍️ Blog |
| **Soir (1h)** | 📰 News | 🔧 Blog Tech | 📰 News | 🧠 Learn | 🍻 OFF | 📰 News | 📅 Planif |
| **Journée WE** | - | - | - | - | - | 🛠️ Freelance | 🔧 Blog Tech |

### 1.6 Phases du Projet

Le développement de sebc.dev se découpe en deux phases distinctes :

| Phase | Période | Objectif | Répartition Temps |
|-------|---------|----------|-------------------|
| **Phase 0 — Build** | Décembre 2024 → Février 2025 | Construction technique du blog | 100% Dev |
| **Phase 1 — Run** | Février 2025 → | Écriture + Freelance clients | 40% Blog / 60% Freelance |

#### Phase 0 : Build Mode

Durant cette phase, **aucune rédaction d'article** n'est prévue. L'intégralité du temps disponible est consacrée à :

- Développement du site (frontend + CMS)
- Infrastructure et déploiement
- SEO technique et performance
- Veille technique ciblée

**Critère de sortie Phase 0** : Blog fonctionnel avec système de publication Articles + News, déployé en production, prêt à recevoir du contenu.

#### Phase 1 : Run Mode

Activation du calendrier éditorial (1 article/semaine + news) et démarrage de l'activité Freelance PME/TPME en parallèle.

### 1.7 Hypothèses & Risques

#### Hypothèses Critiques à Valider

| Hypothèse | Risque si fausse | Validation M3 |
|-----------|------------------|---------------|
| **H1**: Les devs cherchent du contenu intersection IA×Arch×UX | Pas de traction, 0 visiteur | 5+ commentaires/partages organiques |
| **H2**: Le marché FR est sous-servi sur ces sujets | Trafic uniquement EN | 60%+ trafic FR |
| **H3**: Le Pattern Onion fonctionne (3 audiences, 1 article) | Temps lecture très variable, confusion | Scroll depth >50% tous segments |
| **H4**: Le format "Learning in Public" attire | Perçu comme amateur | Feedback qualitatif positif (DM/comments) |
| **H5**: 8h/semaine rédaction suffisent pour 2 articles qualité | Burnout ou qualité dégradée | Respecter deadlines sans stress |

#### Risques Techniques

| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|--------|------------|
| **Nuxt 4 breaking changes** | Moyenne | Élevé | Rester sur Nuxt 3.x stable jusqu'à release officielle Nuxt 4. Migrer M6+ |
| **Dépendance module non maintenu** | Faible | Moyen | Utiliser uniquement modules Nuxt officiels + actifs |
| **Cloudflare Pages limitations** | Faible | Faible | Plan B: Vercel ou VPS (déjà maîtrisé) |
| **Complexité i18n sous-estimée** | Moyenne | Moyen | L'implémentation bilingue (routing, hreflang SEO, switch langue, gestion contenu) demande un effort dev significatif. Prévoir 20-30% du temps MVP pour i18n. Utiliser @nuxtjs/i18n dès le départ. |

#### **Risque de Dépassement Phase 0 :**
   - *Risque :* Le développement technique s'éternise, repoussant indéfiniment l'écriture.
   - *Mitigation :* Date butoir fixée à fin Février. Le blog doit être "good enough" pour publier, pas parfait. Définir un MVP technique strict et s'y tenir. Les améliorations itératives viendront après.

#### Risques Business

| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|--------|------------|
| **Audience trop niche** | Moyenne | Élevé | Valider traction M3, pivot vers stack React/Next si échec |
| **Saturation contenu IA** | Élevée | Moyen | Différenciation par intersection 3 piliers, pas IA pure |
| **Pas de distribution organique** | Moyenne | Élevé | Plan distribution détaillé §4 |

### 1.8 Assumptions & Constraints

#### Assumptions (Hypothèses de Travail)

| ID | Assumption | Impact si Fausse |
|----|------------|------------------|
| **A1** | Le contenu sera géré via fichiers Markdown/MDC dans le repo (pas de CMS headless) | Refonte workflow publication |
| **A2** | L'auteur (Negus) est le seul contributeur pendant Phase 0-1 | Multi-auteur à prévoir si collaboration |
| **A3** | Le déploiement cible Cloudflare Pages avec SSR hybride | Ajuster si autre hébergeur |
| **A4** | Les traductions EN sont faites manuellement (assistées IA) après rédaction FR | Automatisation à prévoir si volume augmente |
| **A5** | Pas de commentaires ni d'authentification utilisateur au MVP | Feature post-MVP si demande |
| **A6** | Analytics via Plausible (privacy-first, pas de cookies) | RGPD simplifié, pas de bannière cookie |
| **A7** | Le format llms.txt suit la spécification de https://llmstxt.org/ | Adapter si standard évolue |

#### Constraints (Contraintes Non-Négociables)

| Contrainte | Justification |
|------------|---------------|
| **Budget 0€ outils** | Side-project, utiliser services gratuits ou open-source |
| **27h30/semaine max** | Contrainte vie personnelle, non-négociable |
| **Solo dev** | Pas de budget pour freelance/contribution externe |
| **Date butoir Phase 0 : Fin Février 2025** | Éviter perfectionnisme, forcer le ship |
| **FR-first, EN second** | Authenticité voix, marché FR sous-servi prioritaire |

### 1.9 Objectifs Business M12

- 5 000-10 000 Visiteurs Uniques/mois
- 500 abonnés newsletter (taux d'ouverture >40%)
- 2-3 demandes inbound consulting/mois
- 500-1 000€/mois revenus passifs

---

## 2. Personas Détaillés

### 2.1 Lucas — "Le Mid-Level Bloqué" (Primary Persona)

#### Fiche d'Identité

| Attribut | Valeur |
|----------|--------|
| **Âge** | 27 ans |
| **Poste** | Développeur Fullstack (Vue.js/Nuxt & Node.js) |
| **Entreprise** | EcoLogistics, startup SaaS B2B (Série A) |
| **Équipe** | 8 personnes tech |
| **Expérience** | 4 ans (bootcamp + alternance → Mid-Level depuis 1 an) |
| **Autonomie** | 80% des tâches, manque vision architecturale macro |

#### Journée Type

```
09h30 — Stand-up
├── Annonce blocage sur intégration chatbot avec base SQL legacy
├── Frustration visible, pas de solution claire en vue

10h00-11h30 — Deep Work (La Lutte)
├── Tente connexion API LLM avec backend Node.js 2022
├── LLM renvoie données que front Nuxt n'arrive pas à typer
├── Chaos TypeScript : types incompatibles, erreurs cryptiques
├── Console.log partout, `any` utilisé avec culpabilité

11h30-12h00 — Recherche Solution (Point de Contact)
├── Requête: "Nuxt AI stream integration legacy API best practices"
├── Tombe sur article sebc.dev via Perplexity/Google
├── Identifie pattern applicable en 30 secondes

14h00-15h00 — Implémentation
├── Copie-colle composable trouvé sur blog
├── Adapte en 15 minutes
├── Ça marche du premier coup

16h00 — Code Review
└── Soumet PR, code plus propre que celui du Senior voisin
```

#### Frictions Quotidiennes

| Friction | Impact | Workaround Actuel |
|----------|--------|-------------------|
| 70% du temps = "glue code" | Productivité divisée par 3 | Regex fragiles, `any` TypeScript |
| Réponses LLM mal formatées | Crashes UI imprévisibles | Console.log partout |
| Manque vision architecturale | Bloqué sur évolution Senior | Imite sans comprendre |
| Legacy vs Moderne | Tension constante | Évite sujets architecture |

#### Vision de Succès

**Le Déclic**: "C'est exactement ce dont j'avais besoin : pas de théorie fumeuse sur le futur de l'IA, mais un bout de code Nuxt utilisable maintenant."

**Mesures Concrètes**:
- Tâche de 4h pliée en 1h
- Application ne crashe plus quand IA répond bizarrement
- Comprend le "pourquoi" et peut l'expliquer en Code Review
- Progression visible vers poste Senior

#### Triggers de Recherche

- Erreurs TypeScript avec LLM
- Intégration IA sur legacy
- Patterns streaming Nuxt
- "Best practices" production IA

---

### 2.2 Chloé — "L'Apprentie Copilote" (Junior Developer)

#### Fiche d'Identité

| Attribut | Valeur |
|----------|--------|
| **Âge** | 24 ans |
| **Formation** | Reconversion, Bootcamp 2024 + auto-formation |
| **Expérience** | 10 mois de code effectif |
| **Poste** | Développeuse Junior (Alternance/Premier CDD) |
| **Entreprise** | Agence Web Digitale |
| **Contexte** | Rythme soutenu, tuteur très occupé |

#### Journée Type

```
09h00 — Arrivée Bureau
├── Brief client avec nouvelle feature IA à intégrer
├── Anxiété : jamais fait ça seule

09h30-12h00 — Développement (Vibe Coding)
├── Cursor ouvert en permanence
├── Prompt: "Génère-moi un composant Nuxt qui fait X"
├── Code généré semble fonctionner, juge au "feeling"
├── Bug silencieux apparaît (hydration mismatch)

12h00-14h00 — Déjeuner + Stress
├── Repaste erreur dans IA
├── IA hallucine correction créant autre bug
├── Spirale d'incompréhension

14h00-17h00 — Blocage
├── Impossible de débugger code qu'elle n'a pas écrit
├── Tuteur indisponible
├── Syndrome de l'imposteur violent

19h00-21h00 — Formation Perso (Soir/Week-end)
├── Cherche ressources pour comprendre
├── Trouve sebc.dev via Discord Vue.js France
├── Lit article Pattern Onion, comprend enfin le "pourquoi"
└── Moment eureka : "Je ne suis pas nulle, je manquais d'explications!"
```

#### Frictions Quotidiennes

| Friction | Impact | Workaround Actuel |
|----------|--------|-------------------|
| IA donne solution finale | Saute étape d'apprentissage | Aucun (subit) |
| Code semble Senior mais fragile | Châteaux de cartes | Évite modifications |
| Mur du débogage | Blocage total sans tuteur | Re-prompt IA en boucle |
| Syndrome imposteur | Anxiété Code Reviews | Minimise ses contributions |

#### Vision de Succès

**Le Déclic**: "Ah! Donc l'IA utilisait shallowRef ici pour optimiser perf, pas juste par hasard! Je comprends enfin la différence."

**Mesures Concrètes**:
- Refactorise code IA sans redemander
- Explique le "pourquoi" lors stand-up
- Corrige bug avant que senior ne le voie
- Passe de "Passagère" à "Pilote"

#### Triggers de Recherche

- "Comprendre X en profondeur"
- "Pourquoi ça marche"
- Concepts fondamentaux (closures, reactivity, lifecycle)
- Tutoriels pas-à-pas bienveillants

---

### 2.3 Maxime — "L'Architecte Frankenstein" (Indie Hacker/Freelance)

#### Fiche d'Identité

| Attribut | Valeur |
|----------|--------|
| **Âge** | 32 ans |
| **Background** | Ancien Senior Dev agence (tout quitté il y a 2 ans) |
| **Statut** | Solopreneur / Freelance mi-temps |
| **Projets** | 3 micro-SaaS (1 à 2k€ MRR, 1 zombie, 1 en lancement) |
| **Revenus** | 2 jours freelance/semaine pour sécurité financière |
| **Stack** | 15+ services (chaos Frankenstein) |

#### Journée Type

```
07h00 — Réveil Anxieux
├── Check emails/Slack clients freelance
├── 3 alertes Sentry ignorées (notification fatigue)
├── Terreur que SaaS principal casse pendant mission

08h00-12h00 — Mission Freelance
├── Client A demande feature urgente
├── Contexte switch violent

12h00-13h00 — Pause (Veille)
├── Scroll X (Twitter), newsletters
├── Voit article sebc.dev: "Comment j'ai économisé 200$/mois de SaaS"
├── Bookmark immédiat

14h00-18h00 — Projets Perso (Integration Hell)
├── Client SaaS se plaint: "L'IA ne répond pas"
├── Debug: Vercel timeout? OpenAI down? Supabase lent? Clé API?
├── Pas de Dashboard Central
├── Login sur 4 services pour tracer une requête

19h00-22h00 — Maintenance (Cycle sans fin)
├── Webhooks échouent entre services
├── Formats JSON différents entre APIs
└── Plus de temps à faire communiquer outils qu'à coder valeur
```

#### Stack Actuelle (Chaos)

| Catégorie | Services | Coût Mensuel |
|-----------|----------|--------------|
| Backend | Supabase, Firebase, Node.js/Railway | ~80€ |
| IA | OpenAI, Pinecone, LangChain | ~100€ |
| Ops | Vercel, Resend, Stripe, Sentry, Axiom, Crisp | ~200€ |
| **Total** | **15+ services** | **~400€** |

#### Frictions Quotidiennes

| Friction | Impact | Workaround Actuel |
|----------|--------|-------------------|
| Integration Hell | 70% temps sur "glu" | Webhooks fragiles |
| 15 onglets monitoring | Zero vue d'ensemble | Emails d'alerte (spam) |
| Notification fatigue | Ignore alertes critiques | Espère que ça marche |
| Stack redondante | 400€/mois gaspillés | Accepte le coût |

#### Vision de Succès

**Le Déclic**: "Ce gars me fait gagner de l'argent."

**Mesures Concrètes**:
- Résilie 3 abonnements SaaS inutiles
- Dashboard "Maison" montrant état santé global (Vert/Rouge)
- Solution DIY implémentée en une après-midi
- Sérénité retrouvée

#### Triggers de Recherche

- "Simplifier stack"
- "DIY alternative to [SaaS]"
- "Pattern monolith modulaire"
- "Économiser" + techno

---

### 2.4 Validation Personas

#### Source Actuelle des Personas

| Persona | Source | Niveau de Confiance |
|---------|--------|---------------------|
| Lucas | Auto-biographie (moi il y a 3 ans) + observations collègues | ⭐⭐⭐ Moyen |
| Chloé | Observations juniors en mission + forums/Discord | ⭐⭐ Faible |
| Maxime | Projection future + suivi créateurs indie (Twitter/IH) | ⭐⭐ Faible |

**Verdict** : Personas basés sur intuition + observation indirecte. Pas d'interviews formelles. **Risque de biais de confirmation.**

#### Plan de Validation M1-M3

| Action | Timing | Objectif | Critère Succès |
|--------|--------|----------|----------------|
| **5 interviews utilisateurs** | M1-M2 | Valider frictions | 3/5 confirment frictions identifiées |
| **Sondage Discord Vue.js FR** | M1 | Quantifier besoins | 30+ réponses, patterns clairs |
| **Analyse commentaires** | M1-M3 | Feedback organique | Commentaires mentionnent frictions prédites |
| **Heatmaps premiers articles** | M2-M3 | Comportement réel | Scroll/Copy patterns alignés personas |

#### Questions Interviews (5 entretiens 30min)

**Cibles** : 2 mid-level, 2 juniors, 1 indie hacker (recrutement via LinkedIn/Discord)

1. "Quand tu bloques sur un problème technique, que fais-tu en premier?"
2. "Décris ta dernière galère avec l'IA/LLM dans ton code."
3. "C'est quoi un article technique que tu as trouvé vraiment utile récemment? Pourquoi?"
4. "Comment tu te formes actuellement? Qu'est-ce qui te frustre?"
5. "Si je te disais 'IA + Architecture + UX', ça t'évoque quoi?"

#### Pivot Personas si Validation Échoue

Si les interviews révèlent des frictions totalement différentes :
- M3 : Reformuler personas
- M4 : Ajuster calendrier éditorial
- Documenter publiquement le pivot ("Learning in Public")

---

## 3. Analyse Concurrentielle

### 3.1 Paysage Francophone

| Concurrent | Forces | Faiblesses | Menace |
|------------|--------|------------|--------|
| **Grafikart** | Énorme audience FR, format vidéo maîtrisé, 15+ ans d'historique | Peu de contenu IA avancé, format long, pas optimisé GEO | ⭐⭐⭐ Élevée (audience captive) |
| **Alsacreations** | Référence historique CSS/HTML, communauté fidèle | Contenu vieillissant, peu d'IA, forums moins actifs | ⭐⭐ Moyenne |
| **Human Coders** | Formation pro, crédibilité entreprises | Payant, pas de contenu libre, pas indie-friendly | ⭐ Faible (segment différent) |
| **WesBos** (mais EN traduit FR) | Cours très polish, bonne réputation | Anglophone, React-centric, pas Vue/Nuxt | ⭐ Faible |

### 3.2 Paysage Anglophone

| Concurrent | Forces | Faiblesses | Menace |
|------------|--------|------------|--------|
| **Josh Comeau** | Visualisations exceptionnelles, pédagogie | React only, pas de contenu IA, EN uniquement | ⭐⭐ Moyenne (référence qualité) |
| **Kent C. Dodds** | Testing, patterns React, influence | React only, contenu moins récent sur IA | ⭐ Faible |
| **Theo Browne** | YouTube viral, opinions tranchées | Drama > contenu actionable, pas de profondeur | ⭐ Faible |
| **Fireship** | Format ultra-concis, viral | Trop superficiel, pas de code production | ⭐ Faible |
| **Leerob (Vercel)** | Insider Next.js, crédibilité | Next.js only, corporate voice | ⭐⭐ Moyenne |

### 3.3 Positionnement Stratégique

```
                    Contenu Profond
                         ▲
                         │
    [Human Coders]       │       [sebc.dev] ← CIBLE
    [Alsacreations]      │       [Josh Comeau]
                         │
    ◄────────────────────┼────────────────────►
    Francophone          │          Anglophone
                         │
    [Grafikart]          │       [Fireship]
    [Dev.to FR]          │       [Theo Browne]
                         │
                         ▼
                  Contenu Superficiel
```

**Espace Libre** : Contenu profond + francophone + intersection IA/Arch/UX

### 3.4 Avantages Concurrentiels Défendables

| Avantage | Durabilité | Difficulté Copie |
|----------|------------|------------------|
| Premier sur niche Nuxt×IA×UX FR | 12-18 mois | Facile |
| Pattern Onion (3 audiences, 1 article) | Long terme | Moyenne (demande effort) |
| Learning in Public authentique | Long terme | Difficile (demande vulnérabilité) |
| Optimisation GEO native | 6-12 mois | Moyenne |
| Expertise pratique battle-tested | Long terme | Très difficile |

---

## 4. Stratégie Distribution M1-M6

### 4.1 Objectif

**500 visiteurs uniques/mois à M6** — Comment y arriver?

### 4.2 Canaux par Phase

#### M1-M2 : Fondations (0 → 100 UV)

| Canal | Action Spécifique | Fréquence | Effort |
|-------|-------------------|-----------|--------|
| **Discord Vue.js FR** | Répondre aux questions + lien article pertinent | Quotidien 15min | ⭐⭐ |
| **Discord Nuxt** | Idem, focus questions IA/streaming | Quotidien 15min | ⭐⭐ |
| **LinkedIn** | Post technique + lien article | 2x/semaine | ⭐⭐ |
| **BlueSky** | Threads techniques FR | 2x/semaine | ⭐ |
| **GitHub** | Repos exemples liés aux articles | Par article | ⭐⭐⭐ |

**Principe** : **Apporter de la valeur d'abord**, puis lien naturel vers article. Jamais de spam.

#### M3-M4 : Croissance (100 → 300 UV)

| Canal | Action Spécifique | Fréquence | Effort |
|-------|-------------------|-----------|--------|
| **Nuxt Weekly Newsletter** | Soumettre articles pertinents | Chaque article | ⭐ |
| **Vue.js News** | Idem | Chaque article | ⭐ |
| **Cross-posting Dev.to** | Version résumée + lien | Chaque article | ⭐⭐ |
| **Reddit r/vuejs, r/webdev** | Partage si vraiment pertinent | 1x/mois | ⭐ |
| **Guest posts blogs FR** | Proposition 1-2 blogs | M3-M4 | ⭐⭐⭐ |

#### M5-M6 : Traction (300 → 500 UV)

| Canal | Action Spécifique | Fréquence | Effort |
|-------|-------------------|-----------|--------|
| **SEO Organique** | Articles M1-M4 commencent à ranker | Passif | ⭐ |
| **GEO (Perplexity, ChatGPT)** | Citations si contenu de qualité | Passif | ⭐ |
| **Newsletter propre** | Lancement avec premiers abonnés | M5 | ⭐⭐ |
| **Podcast FR (invité)** | Pitcher 2-3 podcasts dev FR | M5-M6 | ⭐⭐⭐ |

### 4.3 Métriques Distribution

| Métrique | M2 | M4 | M6 |
|----------|----|----|-----|
| Followers LinkedIn | +100 | +250 | +400 |
| Followers BlueSky | +50 | +150 | +300 |
| Membres Discord (serveur propre) | 0 | 0 | 50 |
| Abonnés newsletter | 0 | 20 | 100 |
| Backlinks qualité | 2 | 5 | 10 |

### 4.4 Contenu Spécial Distribution

| Type | Objectif | Timing |
|------|----------|--------|
| **Thread viral potentiel** | "10 erreurs que je faisais avec l'IA dans Nuxt" | M2 |
| **Repo GitHub starter** | nuxt-ai-starter (open source) | M3 |
| **Cheatsheet PDF gratuit** | "LLM Integration Nuxt" (lead magnet) | M4 |
| **Guest post Grafikart/autre** | Audience empruntée | M4-M5 |

### 4.5 Anti-Patterns à Éviter

| ❌ À Éviter | ✅ À Faire |
|------------|-----------|
| Spammer les Discords avec liens | Répondre sincèrement, lien en bonus si pertinent |
| Poster le même contenu partout | Adapter le format par plateforme |
| Demander des partages | Créer du contenu qu'on veut partager |
| Ignorer les commentaires | Répondre à tout pendant M1-M6 |
| Automatiser trop tôt | Authenticité > scale |

---

## 5. Stratégie Bilingue

### 5.1 Décision : Bilingue Dès le MVP

**100% des articles publiés en FR et EN simultanément**

| Principe | Description |
|----------|-------------|
| FR-first | Rédaction originale en français (langue maternelle, authenticité) |
| Assistance IA rédaction | L'IA assiste la rédaction, ne génère pas d'articles entiers |
| Traduction IA | Agent IA traduit l'article FR vers EN |
| Relecture humaine | 15-30 min de relecture/ajustement par article traduit |

### 5.2 Workflow Publication

```
1. Rédaction FR (assistée IA) ──────────────────────┐
2. Publication article FR                            │ Simultané
3. Traduction IA (FR → EN)                          │
4. Relecture rapide (15-30 min)                     │
5. Publication article EN ──────────────────────────┘
```

**Impact temps** : +15-30 min/article (négligeable sur 6h de rédaction)

### 5.3 Justification Stratégique

| Avantage | Impact |
|----------|--------|
| Double audience dès J1 | Trafic FR + EN, SEO international |
| GEO maximisé | LLMs citent sources bilingues plus facilement |
| Authenticité préservée | Voix FR originale, pas de traduction mécanique |
| Coût marginal faible | 15-30 min vs rédaction native EN (2-3h) |

### 5.4 Qualité Traduction

| Type de contenu | Approche |
|-----------------|----------|
| **Code/Technique** | Traduction IA directe (termes techniques universels) |
| **Explications** | Traduction IA + relecture légère |
| **Opinion/Ton personnel** | Traduction IA + ajustement ton (préserver voix) |

**Outil prévu** : Claude/GPT avec prompt spécialisé "tech blog translation"

---

## 6. Métriques GEO

### 6.1 Qu'est-ce que le GEO?

**Generative Engine Optimization** = Optimisation pour les moteurs de recherche IA (Perplexity, ChatGPT, Gemini, Claude).

Différence avec SEO classique:
- SEO: Ranking dans une liste de liens
- GEO: Être **cité comme source** dans une réponse générée

### 6.2 KPIs GEO

| Métrique | Définition | Cible M12 | Outil |
|----------|------------|-----------|-------|
| **AI Referral Traffic** | Visites depuis Perplexity, ChatGPT, etc. | 20% du trafic total | Plausible (referrer contains) |
| **Citation Rate** | Fréquence où sebc.dev est cité par LLMs | 1/semaine (estimé) | Manual testing |
| **Answer Box Presence** | Contenu extrait pour réponses directes | 10 topics | Testing régulier |
| **llms.txt Crawls** | Visites sur /llms.txt | 100/mois | Plausible |

### 6.3 Méthode de Tracking GEO

#### Tracking Automatique (Plausible)

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

#### Testing Manuel (Hebdomadaire)

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

#### Tracking llms.txt

```
# Dans Plausible: Custom Goal
URL Path: /llms.txt
Event Name: llms_txt_access
```

### 6.4 Optimisation Continue

| Action | Fréquence | Responsable |
|--------|-----------|-------------|
| Test prompts GEO | Hebdomadaire | Auteur |
| Analyse referrers IA | Mensuelle | Auteur |
| Mise à jour llms.txt | À chaque article | Auteur |
| Audit Schema Markup | Trimestrielle | Auteur |

---

## 7. Structure Contenu Answer-First

### 7.1 Principe Answer-First

**Règle d'or**: La réponse à la question du titre doit apparaître dans les 100 premiers mots.

Pourquoi:
- LLMs extraient le début pour citations
- Lecteurs scannent avant de s'engager
- Réduit taux de rebond

### 7.2 Template Article Type

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

## Le Problème

[1-2 paragraphes décrivant la friction que le lecteur ressent]

## La Solution Rapide

\```typescript
// Code copy-paste qui fonctionne immédiatement
\```

[Explication en 3-4 phrases de ce que fait le code]

## Comprendre le Pattern

### Étape 1: [Action]

[Explication + code commenté]

### Étape 2: [Action]

[Explication + code commenté]

### Étape 3: [Action]

[Explication + code commenté]

## Approfondir

<details>
<summary>🧠 Pourquoi ce choix d'architecture?</summary>

[Contenu Layer 3 Pattern Onion - concepts fondamentaux]

</details>

<details>
<summary>⚠️ Pièges courants à éviter</summary>

[Liste des erreurs fréquentes et comment les éviter]

</details>

## Comparaison des Approches

| Approche | Complexité | Performance | Use Case |
|----------|------------|-------------|----------|
| Simple | ⭐ | ⚡⚡ | MVP/Solo |
| Intermédiaire | ⭐⭐ | ⚡⚡⚡ | Équipe petite |
| Avancée | ⭐⭐⭐ | ⚡⚡⚡⚡ | Scale/Enterprise |

## FAQ

### [Question fréquente 1]?

[Réponse concise]

### [Question fréquente 2]?

[Réponse concise]

## Ressources

- [Lien documentation officielle]
- [Article lié interne]
- [Repo GitHub exemple]

---

*Mis à jour le [DATE] — [Changelog si modification majeure]*
```

### 7.3 Règles de Rédaction

#### Titres (H1)

| ✅ Bon | ❌ Mauvais |
|--------|-----------|
| "Comment gérer le streaming IA dans Nuxt" | "Mes réflexions sur le streaming" |
| "Résoudre les erreurs d'hydration Nuxt" | "Un problème courant" |
| "3 patterns pour intégrer LLM et legacy" | "LLM et legacy" |

**Format**: `[Comment/Pourquoi/Quand] + [Verbe Action] + [Sujet Précis] + [Contexte Tech]`

#### Premier Paragraphe

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

#### Blocs de Code

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

#### Longueur

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

## 8. Calendrier Éditorial & Cadence

> **Note** : Cette section s'active uniquement en Phase 1 (à partir de Février 2025).

#### Le Rythme A/B

- **Semaine A (Article Pilier)** : Article de fond (1500+ mots), intersection IA/Arch/UX. Créneaux Lundi + Mardi matin + Dimanche matin.
- **Semaine B (Article Tactique)** : Quick Win, snippet, étude de cas courte. Plus rapide à produire.

#### Format "News" (Brèves)

- **Fréquence** : 2 à 3 fois par semaine (Lundi, Mercredi, Samedi soir).
- **Format** : TIL, analyse release, ou lien commenté.
- **Temps max** : 30 min par brève.

#### Décomposition Temps Blog Phase 1 (~11h/sem)

| Tâche | Temps | Quand |
|-------|-------|-------|
| Rédaction Article Hebdo | 6h | Lun + Mar + Dim matin |
| News/Brèves (2-3/sem) | 1h30 | Soirs Lun + Mer + Sam |
| Veille/Curation | 1h30 | Soirs + Dim |
| Technique (SEO, infra) | 1h | Mar soir + Dim |
| Planification | 1h | Dim soir |

---

## 9. Points de Décision

### 9.1 Checkpoint M3

| Signal | Vert | Orange | Rouge |
|--------|------|--------|-------|
| Visiteurs Uniques | >150 | 50-150 | <50 |
| Temps lecture moyen | >3 min | 2-3 min | <2 min |
| Validation personas (interviews) | 3/5 confirment | 2/5 confirment | <2/5 confirment |
| Burnout ressenti | Non | Léger | Oui |

**Décision M3** :
- 🟢 Vert : Continuer M4-M6
- 🟠 Orange : Ajuster scope ou fréquence, revalider personas
- 🔴 Rouge : Pivot (autre niche, autre format, ou pause)

### 9.2 Décision M6: Validation MVP

#### SCALE UP si:

| Critère | Seuil | Action |
|---------|-------|--------|
| Visiteurs Uniques | > 500/mois | Augmenter fréquence publication |
| Équilibre piliers | Chaque > 20% trafic | Maintenir répartition |
| Newsletter | > 50 abonnés | Lancer automation |
| Engagement | > 60% scroll depth | Explorer nouveaux formats |
| Inbound | 1+ demande consulting | Structurer offre |

#### PIVOT si:

| Signal | Action |
|--------|--------|
| 1 pilier < 15% trafic | Revoir contenu/distribution |
| 1 pilier > 70% trafic | Risque mono-thématique, rééquilibrer |
| Copy Rate < 5% | Code pas assez pratique |
| Temps lecture < 2 min | Contenu pas assez engageant |

#### STOP si:

| Signal | Action |
|--------|--------|
| < 200 UV/mois M6 | Échec positionnement |
| 0 engagement social | Audience pas connectée |
| Pression "expert" insupportable | Format Learning in Public ne marche pas |

### 9.3 Décision M12: Direction Long-terme

#### SCALE UP AGRESSIF si:

| Critère | Seuil | Action |
|---------|-------|--------|
| Visiteurs Uniques | > 5 000/mois | Lancer produits digitaux |
| Newsletter | > 500 abonnés | Newsletter premium |
| Inbound | > 3/mois | Structurer consulting |
| Reconnaissance | Cité par Nuxt Weekly | Chercher sponsors |

#### DIVERSIFICATION si:

| Signal | Action |
|--------|--------|
| Plafond trafic | Ajouter React/Next (même approche) |
| Demande cours | Lancer formation vidéo |
| Demande communauté | Discord payant |

#### MAINTENANCE MODE si:

| Signal | Action |
|--------|--------|
| 2-5k UV stables | Continuer rythme actuel |
| Revenus passifs suffisants | Réduire fréquence |
| Opportunités externes | Prioriser consulting |

---

## 10. Roadmap Détaillée

### 10.1 Vue d'Ensemble

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

### 10.2 M6 — Fin MVP

#### Livrables Techniques

| Livrable | Description | Statut |
|----------|-------------|--------|
| Blog Nuxt fonctionnel | SSR, SEO, bilingue | ✅ Requis |
| 17 articles publiés | 6 IA, 5 Ing., 4 UX, 3 Cross | ✅ Requis |
| ToC Interactive | Sidebar sticky, progress bar | ✅ Requis |
| Code Blocks | Syntax + Copy + Badge | ✅ Requis |
| Analytics | Plausible avec events custom | ✅ Requis |
| llms.txt | Fichier contexte LLMs | ✅ Requis |
| Schema Markup | TechArticle, FAQ | ✅ Requis |

#### Métriques Cibles

| Métrique | Cible M6 |
|----------|----------|
| Visiteurs Uniques/mois | 500 |
| Temps lecture moyen | > 3 min |
| Scroll Depth | > 60% |
| Copy Rate | > 10% |
| AI Referral Traffic | 10% |

> **Note** : La newsletter est prévue comme première itération post-MVP. Les objectifs d'abonnés seront définis lors de cette phase.

### 10.3 M9 — Automation

#### Livrables

| Livrable | Description | Investissement |
|----------|-------------|----------------|
| Newsletter Automation | Séquence bienvenue, weekly digest | 2 jours |
| Cross-posting | Dev.to, Medium, Hashnode | 1 jour/article |
| Template Preparation | Nuxt AI Starter Kit (structure) | 5 jours |
| Documentation Starter | README, docs, exemples | 3 jours |

#### Métriques Cibles

| Métrique | Cible M9 |
|----------|----------|
| Visiteurs Uniques/mois | 2 000 |
| Newsletter abonnés | 150 |
| Taux ouverture NL | > 45% |
| Articles total | 26 |

### 10.4 M12 — Premier Produit

#### Livrables

| Livrable | Description | Prix | Cible Ventes |
|----------|-------------|------|--------------|
| Nuxt AI Starter Kit | Template complet IA + Arch + UX | 49€ | 10/mois |
| Documentation Kit | Docs complètes + tutoriels vidéo | Inclus | - |
| Support | GitHub Issues + Discord gratuit | Inclus | - |

#### Contenu du Starter Kit

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

#### Métriques Cibles

| Métrique | Cible M12 |
|----------|----------|
| Visiteurs Uniques/mois | 5 000 |
| Newsletter abonnés | 500 |
| MRR Produits | 100€ |
| Inbound Consulting | 2-3/mois |
| Backlinks Qualité | 20 |

### 10.5 M18 — Expansion Produits

#### Nouveaux Livrables

| Livrable | Description | Prix | Cible |
|----------|-------------|------|-------|
| Nuxt Enterprise Template | DDD/Hexagonal complet | 79€ | 5/mois |
| Testing Boilerplate | Unit/Integration/E2E setup | 49€ | 5/mois |
| CI/CD Pipelines | GitHub Actions templates | 39€ | 5/mois |
| Bundle Complet | Tous les templates | 149€ | 3/mois |

#### Exploration

| Exploration | Décision M18 | Investment |
|-------------|--------------|------------|
| Cours Vidéo | Go/No-Go basé demande | 2-3 mois si Go |
| SaaS Outil Dev | Prototype si demande | 1 mois prototype |
| Consulting Structuré | Offre packagée | 1 semaine setup |

#### Métriques Cibles

| Métrique | Cible M18 |
|----------|----------|
| Visiteurs Uniques/mois | 8 000 |
| Newsletter abonnés | 800 |
| MRR Produits | 300€ |
| MRR Total (+ Consulting) | 800€ |

### 10.6 M24 — Autorité Établie

#### Livrables

| Livrable | Description | Prix/Modèle |
|----------|-------------|-------------|
| Communauté Discord | 3 channels (#ia, #arch, #ux) | 5€/mois |
| Newsletter Premium | Deep-dives hebdo exclusifs | 5€/mois |
| Sponsors Blog | 1-2 sponsors techniques alignés | 500-1000€/mois |
| Cours Vidéo (optionnel) | "IA + Clean Code + UX" bundle | 150€ |

#### Métriques Cibles Vision

| Métrique | Cible M24 |
|----------|----------|
| Visiteurs Uniques/mois | 10 000+ |
| Newsletter abonnés | 1 000+ |
| MRR Total | 1 000€ |
| Reconnaissance | Référence francophone Nuxt/IA/UX |
| Invitations Podcasts | 3+/an |
| Citations Nuxt Weekly | 3+/an |

---

## 11. Monétisation

### 11.1 Répartition Cible Revenus

| Source | Part M12 | Part M24 |
|--------|----------|----------|
| Consulting/Freelance (TJM+) | 70% | 40% |
| Produits Digitaux | 20% | 35% |
| Sponsoring | 0% | 15% |
| Communauté/Newsletter Premium | 10% | 10% |

### 11.2 Produits Digitaux par Segment

#### Pour Indie Hackers (Maxime) — Quick Ship

| Produit | Description | Prix | Cible/mois |
|---------|-------------|------|------------|
| Nuxt Indie Starter | 1 fichier auth + logs + payments | 29€ | 15 |
| RAG 1-File Pattern Collection | Copy-paste ready patterns | 19€ | 20 |
| Deployment Scripts Bundle | Cloudflare/Railway/Fly configs | 15€ | 10 |

**Positionnement**: "Ship maintenant, scale plus tard"

#### Pour Teams (Lucas) — Production-Ready

| Produit | Description | Prix | Cible/mois |
|---------|-------------|------|------------|
| Nuxt Enterprise Architecture | DDD/Hexagonal complet | 79€ | 5 |
| Testing Strategy Boilerplate | Unit/Integration/E2E setup | 49€ | 8 |
| CI/CD Pipeline Templates | GitHub Actions complet | 39€ | 8 |

**Positionnement**: "Architecture battle-tested pour équipes"

#### Cross-Segment

| Produit | Description | Prix | Cible/mois |
|---------|-------------|------|------------|
| Scaling Guide: Indie → Enterprise | Migration path documenté | 59€ | 5 |
| Bundle Complet | Tous les templates | 149€ | 3 |

### 11.3 Pricing Strategy

#### Principes

1. **Pas de SaaS**: Produits one-time, pas d'abonnement (sauf communauté)
2. **Valeur > Prix**: Chaque produit économise > 10x son prix en temps
3. **Upgrades naturels**: Indie Starter → Enterprise quand scale
4. **Mises à jour incluses**: Lifetime updates pour versions majeures

#### Comparaison Marché

| Concurrent | Produit Similaire | Prix | Notre Prix | Différenciation |
|------------|-------------------|------|------------|-----------------|
| Shipfast | SaaS Boilerplate | $199 | 79€ | Focus Nuxt + IA |
| Divjoy | React Starter | $149 | 49€ | Architecture + UX |
| Tailwind UI | Components | $299 | Inclus | Accessibilité native |

### 11.4 Sponsoring (M12+)

#### Critères d'Acceptation

| Critère | Requis |
|---------|--------|
| Alignement technique | Outils Nuxt/Vue/IA |
| Qualité produit | Utilisé personnellement |
| Non-intrusif | Pas de popups, pas d'AdSense |
| Transparence | Clairement identifié "Sponsor" |

#### Sponsors Cibles

| Catégorie | Exemples | Fourchette/mois |
|-----------|----------|-----------------|
| Hébergement | Vercel, Cloudflare, Railway | 300-500€ |
| Monitoring | Sentry, Axiom | 200-400€ |
| IA | OpenAI, Anthropic | 500-1000€ |
| DevTools | GitHub, Linear | 200-400€ |

#### Formats Sponsoring

| Format | Description | Prix Indicatif |
|--------|-------------|----------------|
| Article Sponsorisé | 1 article/mois, 100% éditorial | 500-1000€ |
| Mention Newsletter | Section "Powered by" | 200-300€/mois |
| Sidebar Badge | Logo dans sidebar blog | 100-200€/mois |

---

## 12. Expansion Post-MVP

### 12.1 Nouvelles Audiences (M12+)

| Audience | Timing | Contenu Adapté |
|----------|--------|----------------|
| **PMs** | M12+ | "Comprendre intersection IA/Arch/UX pour mieux spec" |
| **UX Designers** | M15+ | "Coder ses propres prototypes IA" |
| **CTOs** | M18+ | "Décisions architecture pour produits IA-first" |
| **Juniors Code IA** | M12+ | "Sortir du Vibe Coding" (série) |
| **Seniors Legacy** | M15+ | "Moderniser sans tout casser" |

### 12.2 Expansion Technologique (M18+)

| Technologie | Timing | Approche |
|-------------|--------|----------|
| **React/Next.js** | M18+ | Mêmes patterns, autre framework |
| **SvelteKit** | M24+ | Si demande significative |
| **Mobile (React Native)** | M24+ | Extension naturelle |
| **Backend (Rust/Go)** | M24+ | Selon exploration personnelle |

**Principe**: Toujours garder l'approche 3 piliers (IA × Ingénierie × UX)

### 12.3 Sujets par Pilier — Vision Complète

#### IA (40%)

| Sujet | Niveau | Priorité |
|-------|--------|----------|
| RAG Production | Intermédiaire | P0 |
| Agents Autonomes | Avancé | P1 |
| LLM Tooling | Intermédiaire | P0 |
| Prompting Avancé | Débutant-Intermédiaire | P0 |
| Context Engineering | Intermédiaire | P1 |
| Observabilité IA | Avancé | P2 |
| Edge AI (Cloudflare Workers) | Intermédiaire | P1 |
| Fine-tuning (Post-M18) | Avancé | P3 |

#### Ingénierie Logicielle (30%)

| Sujet | Niveau | Priorité |
|-------|--------|----------|
| Architecture Hexagonale | Intermédiaire | P0 |
| DDD | Avancé | P1 |
| CQRS | Avancé | P2 |
| Clean Code | Débutant-Intermédiaire | P0 |
| SOLID | Intermédiaire | P0 |
| Design Patterns | Intermédiaire | P1 |
| Testing Strategies | Intermédiaire | P0 |
| CI/CD | Intermédiaire | P1 |
| Java/Spring (Backend) | Avancé | P3 |

#### UX (30%)

| Sujet | Niveau | Priorité |
|-------|--------|----------|
| Design Systems | Intermédiaire | P0 |
| Accessibilité (WCAG) | Débutant-Intermédiaire | P0 |
| UX Writing | Débutant | P1 |
| Micro-interactions | Intermédiaire | P1 |
| IA dans Workflows UX | Avancé | P2 |
| Prototyping IA | Intermédiaire | P2 |
| Research Methods | Intermédiaire | P3 |

### 12.4 Articles Cross-Piliers (Valeur Unique)

| Article | Piliers | Priorité |
|---------|---------|----------|
| "Architecture RAG avec UX First" | IA + UX + Ingénierie | P0 |
| "Design System pour Apps IA" | UX + IA | P1 |
| "Clean Code pour Composants Accessibles" | Ingénierie + UX | P1 |
| "De Junior à Tech Lead: Maîtriser les 3 Piliers" | Meta | P2 |
| "Observabilité IA: Du Backend à l'UX" | IA + Ingénierie + UX | P2 |
| "Full Stack Holistique" | 3 piliers intégrés | P1 |

---

## 13. Functional Requirements (MVP)

Cette section définit le **contrat de capacités** du produit. Chaque FR est une capacité testable que le système doit fournir.

### 13.1 Lecture de Contenu

| FR# | Requirement |
|-----|-------------|
| FR1 | Le visiteur peut lire un article complet sur une page dédiée |
| FR2 | Le visiteur peut voir le thème de l'article (IA / Ingénierie logicielle / UX) |
| FR3 | Le visiteur peut voir la catégorie de l'article (Actualité / Tutoriel / Décryptage / Étude de cas / Retour d'expérience) |
| FR4 | Le visiteur peut voir les tags associés à l'article |
| FR5 | Le visiteur peut voir le niveau de l'article (Tous niveaux / Débutant / Intermédiaire / Avancé) |
| FR6 | Le visiteur peut voir le temps de lecture estimé |
| FR7 | Le visiteur peut voir la date de publication |
| FR8 | Le visiteur peut voir une table des matières générée automatiquement depuis les titres |
| FR9 | Le visiteur peut voir sa progression de lecture dans l'article (indicateur visuel) |
| FR10 | Le visiteur peut déplier les sections "Approfondir" (details/summary) |

#### Critères d'Acceptation — Lecture de Contenu

**FR1 — Article complet**
- L'article s'affiche sur une URL unique `/[lang]/articles/[slug]`
- Le contenu Markdown/MDC est rendu correctement (titres, paragraphes, listes, images, liens)
- Les images ont un attribut alt et sont lazy-loaded
- Le temps de chargement initial < 2s (LCP)

**FR2-5 — Métadonnées article**
- Thème, catégorie, niveau et tags sont affichés sous forme de badges cliquables
- Chaque badge utilise une couleur distinctive par type (thème: couleur pilier, catégorie: neutre, niveau: gradient)
- Les badges sont positionnés de manière cohérente (header article ou sidebar)

**FR6 — Temps de lecture**
- Calcul basé sur 200 mots/minute (standard FR)
- Arrondi à la minute supérieure
- Format affiché : "X min de lecture"

**FR7 — Date de publication**
- Format FR : "15 janvier 2025"
- Format EN : "January 15, 2025"
- Localisée selon la langue active

**FR8 — Table des matières (ToC)**
- Générée automatiquement depuis les titres H2 et H3
- Affichée en sidebar sticky sur desktop (visible pendant le scroll)
- Sur mobile : accessible via bouton flottant ou collapse en haut d'article
- Scroll smooth vers la section au clic
- Highlight de la section active pendant la lecture

**FR9 — Progression de lecture**
- Barre de progression horizontale fixée en haut de la page
- Progression basée sur le scroll (0% en haut, 100% à la fin de l'article)
- Visible uniquement sur les pages article

**FR10 — Sections Approfondir**
- Utilise `<details><summary>` HTML natif
- État fermé par défaut
- Animation smooth à l'ouverture/fermeture (CSS transition)
- Icône chevron indiquant l'état (→ ouvert, ↓ fermé)

### 13.2 Interaction avec le Code

| FR# | Requirement |
|-----|-------------|
| FR11 | Le visiteur peut voir les blocs de code avec coloration syntaxique |
| FR12 | Le visiteur peut copier un bloc de code en un clic |
| FR13 | Le visiteur peut voir le langage du bloc de code (badge) |

#### Critères d'Acceptation — Interaction Code

**FR11 — Coloration syntaxique**
- Support des langages : TypeScript, JavaScript, Vue/HTML, CSS, JSON, Bash, Markdown
- Thème de coloration cohérent avec le design sombre du site
- Numéros de ligne optionnels (activables via prop)
- Highlight de lignes spécifiques (ex: `{3-5}` pour lignes 3 à 5)

**FR12 — Copie code**
- Bouton "Copier" visible au survol ou en permanence (coin supérieur droit)
- Au clic : copie dans le presse-papier
- Feedback visuel : icône change en "✓" ou texte "Copié!" pendant 2 secondes
- Copie le code brut (sans numéros de ligne ni formatage)

**FR13 — Badge langage**
- Badge affiché en haut à gauche du bloc de code
- Texte en minuscules (ex: "typescript", "vue", "bash")
- Style discret, ne pas distraire de la lecture

### 13.3 Navigation Globale (Header)

| FR# | Requirement |
|-----|-------------|
| FR14 | Le visiteur peut voir le logo et le nom du site dans le header |
| FR15 | Le visiteur peut naviguer vers l'accueil via le header |
| FR16 | Le visiteur peut naviguer vers la page de recherche via le lien "Articles" |
| FR17 | Le visiteur peut accéder aux thèmes via un dropdown dans le header |
| FR18 | Le visiteur peut accéder aux catégories via un dropdown dans le header |
| FR19 | Le visiteur peut accéder aux niveaux via un dropdown dans le header |
| FR20 | Le visiteur peut sélectionner la langue (FR/EN) via un dropdown à droite du header |

#### Critères d'Acceptation — Navigation Header

**FR14-15 — Logo et accueil**
- Logo cliquable, redirige vers `/[lang]/`
- Logo visible sur toutes les pages
- Taille adaptée : desktop et mobile

**FR16 — Lien Articles**
- Lien texte "Articles" dans la navigation principale
- Redirige vers `/[lang]/articles` (page de recherche/listing)

**FR17-19 — Dropdowns navigation**
- Dropdowns pour Thèmes, Catégories, Niveaux
- Au clic : affiche liste des options disponibles
- Chaque option redirige vers `/[lang]/articles?[filter]=[value]`
- Fermeture au clic extérieur ou touche Escape
- Accessible au clavier (Tab, Enter, Escape)

**FR20 — Sélecteur de langue**
- Dropdown FR/EN positionné à droite du header
- Langue active affichée (ex: "FR" ou drapeau)
- Au changement : redirige vers la même page dans l'autre langue
- Préserve les filtres et paramètres d'URL

### 13.4 Navigation par Badges

| FR# | Requirement |
|-----|-------------|
| FR21 | Le visiteur peut cliquer sur un badge thème pour accéder à la recherche filtrée par ce thème |
| FR22 | Le visiteur peut cliquer sur un badge catégorie pour accéder à la recherche filtrée par cette catégorie |
| FR23 | Le visiteur peut cliquer sur un badge tag pour accéder à la recherche filtrée par ce tag |
| FR24 | Le visiteur peut cliquer sur un badge niveau pour accéder à la recherche filtrée par ce niveau |

#### Critères d'Acceptation — Navigation Badges

**FR21-24 — Badges cliquables**
- Tous les badges (thème, catégorie, tag, niveau) sont cliquables
- Au clic : redirige vers `/[lang]/articles?[type]=[value]`
- Curseur pointer au survol
- État hover visuel (légère mise en évidence)

### 13.5 Page d'Accueil

| FR# | Requirement |
|-----|-------------|
| FR25 | Le visiteur peut voir le dernier article en vedette (pleine largeur) |
| FR26 | Le visiteur peut voir une grille des articles suivants sous l'article en vedette |

#### Critères d'Acceptation — Page d'Accueil

**FR25 — Article en vedette**
- Le dernier article publié est affiché en hero (pleine largeur)
- Affiche : titre, description, thème, temps de lecture, date
- Image de couverture si disponible (fallback : gradient ou pattern)
- Lien cliquable vers l'article complet

**FR26 — Grille articles**
- Affiche les articles suivants (excluant le hero) en grille
- Desktop : 3 colonnes
- Tablette : 2 colonnes
- Mobile : 1 colonne
- Chaque carte affiche : titre, description tronquée, thème, temps de lecture
- Nombre d'articles affichés : 6-9 (configurable)
- Lien "Voir tous les articles" vers la page de recherche

### 13.6 Recherche & Filtrage

| FR# | Requirement |
|-----|-------------|
| FR27 | Le visiteur peut accéder à une page de recherche dédiée |
| FR28 | Le visiteur peut filtrer les articles par thème (sidebar) |
| FR29 | Le visiteur peut filtrer les articles par catégorie (sidebar) |
| FR30 | Le visiteur peut filtrer les articles par tag (sidebar) |
| FR31 | Le visiteur peut filtrer les articles par niveau (sidebar) |
| FR32 | Le visiteur peut filtrer les articles par temps de lecture (sidebar) |
| FR33 | Le visiteur peut filtrer les articles par date de publication (sidebar) |
| FR34 | Le visiteur peut combiner plusieurs filtres simultanément |
| FR35 | Le visiteur peut voir les filtres actifs reflétés dans l'URL (deep linking) |
| FR36 | Le visiteur peut naviguer entre les pages de résultats via pagination |

#### Critères d'Acceptation — Recherche & Filtrage

**FR27 — Page de recherche**
- URL : `/[lang]/articles`
- Layout : sidebar filtres (gauche) + grille résultats (droite)
- Sur mobile : filtres accessibles via drawer/modal
- Affiche le nombre total de résultats

**FR28-31 — Filtres par métadonnées**
- Filtres checkbox pour : thème, catégorie, tag, niveau
- Affichage du count par option (ex: "IA (12)")
- Sélection multiple possible au sein d'un même type
- Application immédiate (pas de bouton "Appliquer")

**FR32 — Filtre temps de lecture**
- Options prédéfinies : "< 5 min", "5-10 min", "10-15 min", "> 15 min"
- Sélection unique

**FR33 — Filtre date de publication**
- Options prédéfinies : "Cette semaine", "Ce mois", "Cette année", "Tout"
- Sélection unique

**FR34 — Combinaison de filtres**
- Logique ET entre types de filtres différents (thème ET catégorie)
- Logique OU au sein d'un même type (IA OU UX)
- Bouton "Réinitialiser les filtres" visible si filtres actifs

**FR35 — Deep linking URL**
- Format : `/[lang]/articles?theme=ia&category=tutoriel&level=intermediate`
- URL partageable et bookmarkable
- Au chargement : restaure les filtres depuis l'URL

**FR36 — Pagination**
- 12 articles par page (configurable)
- Navigation : "Précédent" / "Suivant" + numéros de page
- Scroll to top au changement de page
- Paramètre URL : `?page=2`

### 13.7 Bilingue

| FR# | Requirement |
|-----|-------------|
| FR37 | Le visiteur peut lire le contenu en français |
| FR38 | Le visiteur peut lire le contenu en anglais |
| FR39 | Le visiteur peut basculer entre FR et EN sur chaque page |
| FR40 | Le système détecte la langue préférée du navigateur pour la langue par défaut |

#### Critères d'Acceptation — Bilingue

**FR37-38 — Contenu multilingue**
- Chaque article existe en version FR et EN
- URL structure : `/fr/articles/[slug]` et `/en/articles/[slug]`
- Interface (navigation, labels, boutons) traduite dans chaque langue
- Si un article n'existe pas dans une langue, afficher message "Cet article n'est pas encore traduit"

**FR39 — Bascule de langue**
- Sélecteur de langue accessible depuis le header (toutes pages)
- Au clic : redirige vers l'équivalent dans l'autre langue
- Préserve la position dans l'article si possible

**FR40 — Détection langue navigateur**
- À la première visite sur `/`, détecter `Accept-Language` header
- Rediriger vers `/fr/` ou `/en/` selon la préférence
- Fallback : FR si langue non supportée
- Stocker le choix utilisateur (cookie ou localStorage) pour les visites suivantes

### 13.8 SEO & GEO

| FR# | Requirement |
|-----|-------------|
| FR41 | Le système génère un fichier llms.txt accessible aux LLMs |
| FR42 | Le système génère des Schema Markup (TechArticle, FAQ) par article |
| FR43 | Le système génère un sitemap XML automatiquement |
| FR44 | Le système génère des meta tags Open Graph et Twitter Cards |
| FR45 | Le système génère un flux RSS des articles |

#### Critères d'Acceptation — SEO & GEO

**FR41 — llms.txt**
- Fichier accessible à `/llms.txt`
- Contenu : présentation du site, liste des articles avec descriptions, guidelines pour citations
- Format conforme à https://llmstxt.org/
- Mis à jour automatiquement à chaque build

**FR42 — Schema Markup**
- Chaque article inclut JSON-LD `TechArticle` avec : headline, author, datePublished, description
- Si FAQ présente dans l'article, inclure schema `FAQPage`
- Validation sans erreur sur https://validator.schema.org/

**FR43 — Sitemap XML**
- Généré automatiquement à `/sitemap.xml`
- Inclut toutes les pages publiques (accueil, articles, pages statiques)
- Versions FR et EN avec hreflang
- Lastmod basé sur la date de modification

**FR44 — Open Graph & Twitter Cards**
- Chaque page inclut : og:title, og:description, og:image, og:url, og:type
- Twitter Card : summary_large_image
- Image OG générée dynamiquement ou image de couverture de l'article

**FR45 — Flux RSS**
- Accessible à `/rss.xml` (ou `/feed.xml`)
- Versions séparées FR et EN : `/fr/rss.xml`, `/en/rss.xml`
- Inclut les 20 derniers articles avec : titre, description, lien, date

### 13.9 Apparence & Responsive

| FR# | Requirement |
|-----|-------------|
| FR46 | Le site s'affiche uniquement en mode sombre (thème fixe, non modifiable) |
| FR47 | Le site est responsive et adapté à l'affichage desktop |
| FR48 | Le site est responsive et adapté à l'affichage tablette |
| FR49 | Le site est responsive et adapté à l'affichage mobile |

#### Critères d'Acceptation — Apparence & Responsive

**FR46 — Mode sombre**
- Palette de couleurs sombres uniquement (pas de toggle clair/sombre)
- Couleurs définies via CSS variables pour cohérence
- Contraste texte/fond conforme WCAG AA (ratio ≥ 4.5:1)

**FR47 — Desktop**
- Breakpoint : ≥ 1024px
- Layout full-width avec max-width container (1200-1400px)
- Sidebar ToC visible en permanence sur les articles
- Grilles à 3 colonnes pour les listings

**FR48 — Tablette**
- Breakpoint : 768px - 1023px
- Layout adapté avec sidebar collapsible
- Grilles à 2 colonnes pour les listings
- Navigation header complète

**FR49 — Mobile**
- Breakpoint : < 768px
- Navigation hamburger ou drawer
- ToC accessible via bouton flottant
- Grilles à 1 colonne
- Touch targets ≥ 44px (accessibilité)

### 13.10 Pages Statiques

| FR# | Requirement |
|-----|-------------|
| FR50 | Le visiteur peut accéder à une page "À propos" |

#### Critères d'Acceptation — Pages Statiques

**FR50 — Page À propos**
- URL : `/[lang]/about` ou `/[lang]/a-propos`
- Contenu : présentation de l'auteur, vision du blog, contact
- Même layout que les articles (cohérence visuelle)
- Accessible depuis le footer ou la navigation

### 13.11 Récapitulatif MVP

| Domaine | Nombre de FRs |
|---------|---------------|
| Lecture de Contenu | 10 |
| Interaction Code | 3 |
| Navigation Globale (Header) | 7 |
| Navigation Badges | 4 |
| Page d'Accueil | 2 |
| Recherche & Filtrage | 10 |
| Bilingue | 4 |
| SEO & GEO | 5 |
| Apparence & Responsive | 4 |
| Pages Statiques | 1 |
| **TOTAL MVP** | **50 FRs** |

---

## 14. Non-Functional Requirements (MVP)

Cette section définit les **exigences de qualité** du système — comment il doit performer, pas ce qu'il fait.

**Objectif Lighthouse : 100/100/100/100** (Performance, Accessibility, Best Practices, SEO)

### 14.1 Performance

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR1 | Largest Contentful Paint (LCP) | < 2.5s |
| NFR2 | First Input Delay (FID) | < 100ms |
| NFR3 | Cumulative Layout Shift (CLS) | < 0.1 |
| NFR4 | Time to First Byte (TTFB) | < 200ms |
| NFR5 | Score Lighthouse Performance | ≥ 95 (viser 100 si effort raisonnable) |

### 14.2 Accessibilité

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR6 | Conformité WCAG | Niveau AA (2.1) |
| NFR7 | Score Lighthouse Accessibility | 100 |
| NFR8 | Navigation clavier complète | 100% des fonctionnalités |
| NFR9 | Contraste des couleurs (mode sombre) | Ratio ≥ 4.5:1 (texte normal) |

### 14.3 SEO Technique

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR10 | Score Lighthouse SEO | 100 |
| NFR11 | Toutes les pages indexables | 100% (sauf explicitement exclues) |
| NFR12 | Temps de crawl par page | < 500ms |
| NFR13 | Validation Schema.org | 0 erreur, 0 warning |

### 14.4 Best Practices

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR14 | Score Lighthouse Best Practices | 100 |

### 14.5 Compatibilité Navigateurs

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR15 | Chrome (2 dernières versions) | Support complet |
| NFR16 | Firefox (2 dernières versions) | Support complet |
| NFR17 | Safari (2 dernières versions) | Support complet |
| NFR18 | Edge (2 dernières versions) | Support complet |
| NFR19 | Internet Explorer | Exclu |

### 14.6 Disponibilité

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR20 | Uptime mensuel | ≥ 99.5% |

### 14.7 Sécurité

| NFR# | Requirement | Cible |
|------|-------------|-------|
| NFR21 | HTTPS obligatoire | 100% des pages |
| NFR22 | Headers de sécurité (CSP, X-Frame-Options, etc.) | Configurés |
| NFR23 | Score Mozilla Observatory | ≥ B+ |

### 14.8 Récapitulatif NFRs

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

## Annexes

### A. Documents Liés

| Document | Description |
|----------|-------------|
| [architecture.md](./architecture.md) | Spécifications techniques, stack, composants, SSR/GEO |

### B. Résumé des Modifications v3.0

| Modification | Détail |
|--------------|--------|
| Extraction architecture | Sections 7 et 9 déplacées vers architecture.md |
| Renumérotation sections | 8→7, 10→8, 11→9, 12→10, 13→11, 14→12 |
| Ajout référence | Lien vers architecture.md en en-tête |

### C. Prochaines Étapes

1. ☐ Valider budget temps avec planning réel semaine type
2. ☐ Recruter 5 personnes pour interviews personas (M1)
3. ☐ Finaliser dev blog MVP (avant M1 contenu)
4. ☐ Rédiger premier article pilote
5. ☐ Setup Plausible + events custom

### D. Glossaire

| Terme | Définition |
|-------|------------|
| **GEO** | Generative Engine Optimization — Optimisation pour moteurs de recherche IA |
| **Pattern Onion** | Architecture de contenu à niveaux de profondeur multiples |
| **Learning in Public** | Approche d'apprentissage transparente et documentée publiquement |
| **Glue Code** | Code d'intégration entre systèmes, souvent laborieux |
| **Hollow Senior** | Développeur avec productivité senior mais compréhension junior |
| **Vibe Coding** | Coder au feeling sans comprendre le code |
| **TJM** | Taux Journalier Moyen (consulting/freelance) |

### E. Références

- [Nuxt Documentation](https://nuxt.com/docs)
- [Tailwind CSS](https://tailwindcss.com)
- [Plausible Analytics](https://plausible.io)
- [Schema.org TechArticle](https://schema.org/TechArticle)
- [llms.txt Standard](https://llmstxt.org)

---

## Changelog

| Version | Date | Modifications |
|---------|------|---------------|
| 1.0 | 2025-12-28 | Création initiale |
| 2.0 | 2025-12-28 | Révision Claude v1 |
| 3.0 | 2025-12-28 | Extraction architecture vers document séparé, renumérotation sections |
| 4.0 | 2025-12-28 | Ajout Section 13 (50 Functional Requirements MVP) et Section 14 (Post-MVP Requirements) |
| 5.0 | 2025-12-28 | Ajout Section 14 (23 Non-Functional Requirements MVP), renumérotation Post-MVP → Section 15 |
| 6.0 | 2025-12-28 | Review PM : Ajout header Lucas (2.1), Section 1.8 Assumptions & Constraints, risque i18n, critères d'acceptation détaillés pour tous les FRs, cible Lighthouse 95+, newsletter reportée post-MVP, suppression Section 15 Post-MVP |
