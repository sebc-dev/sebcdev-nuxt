---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments: []
date: 2025-12-27
author: Negus
---

# Product Brief: sebc.dev

## Executive Summary

**sebc.dev** est une plateforme de contenu technique qui documente en temps réel le parcours d'un développeur autodidacte naviguant l'intersection de trois piliers essentiels du développement moderne : **IA, Ingénierie Logicielle, et UX**.

Face à la transformation radicale du métier de développeur fin 2025, où l'IA devient un assistant omniprésent, la plateforme défend une thèse contre-intuitive : les développeurs doivent **prendre de la hauteur** et maîtriser trois domaines complémentaires plutôt que se spécialiser dans le code pur, qu'ils délégueront de plus en plus à l'IA.

La plateforme cible prioritairement les **développeurs mid-level** cherchant à évoluer vers des rôles plus architecturaux, puis s'étend naturellement aux **juniors** construisant leurs fondamentaux et aux **indie hackers** jonglant entre technique, produit et design.

L'avantage compétitif unique : documenter une **exploration technique rigoureuse** à l'intersection de trois disciplines, avec un positionnement "Learning in Public" assumé - pas un consultant qui théorise, mais un ingénieur R&D qui teste les frontières en production et partage les résultats validés en temps réel.

Le blog sert également un objectif personnel critique : **valider et documenter des patterns production-ready** à l'intersection de ces trois domaines. En tant qu'ingénieur autodidacte, explorer les frontières technologiques et systématiser ce qui fonctionne en production est le meilleur moyen d'ancrer une expertise solide. Cette démarche transparente devient la force du contenu : résultats d'exploration validés plutôt que théories déconnectées de la réalité terrain.

---

## Core Vision

### Problem Statement

L'industrie du développement logiciel fin 2025 traverse une **transformation structurelle du métier** : le code devient une commodity que l'IA peut générer, forçant les développeurs à évoluer ou disparaître. Trois gaps critiques émergent :

**1. Le Gap de Hauteur (Mid-level → Senior/Architect)**

Les développeurs mid-level excellents en implémentation se retrouvent coincés : l'IA peut coder aussi bien voire mieux qu'eux, mais ils manquent de vision architecturale, de compréhension UX, et d'expertise IA pour orchestrer ces outils. Ils sentent qu'ils doivent "monter en gamme" mais ne savent pas vers quoi.

**2. Le Gap de Fondamentaux (Juniors & Autodidactes)**

Une génération de développeurs formés rapidement (bootcamps, reconversions) ou autodidactes maîtrise les frameworks du moment mais manque de bases solides. Ils peuvent construire avec l'IA mais ne comprennent pas les principes sous-jacents :
- Patterns d'architecture logicielle (SOLID, DDD, Clean Architecture)
- Fondamentaux UX (accessibilité, design systems, workflows utilisateur)
- Principes IA (RAG, agents, prompting, context engineering)

**3. Le Gap de Vision Holistique (Tous niveaux)**

Les ressources existantes traitent IA, Ingénierie Logicielle, et UX comme des silos séparés. Aucune ne montre comment ces trois piliers s'interconnectent pour créer des produits modernes robustes. Les développeurs naviguent entre tutoriels IA déconnectés de la réalité architecturale, cours d'architecture ignorant l'IA, et ressources UX pensées pour designers.

### Problem Impact

**Si non résolu**, les conséquences structurelles sont dévastatrices:

- **Productivité**: Stagnation durable avec burnout cognitif massif. Les développeurs deviennent des gestionnaires d'attente passifs, changeant constamment de contexte (lancer tâche → attendre → vérifier → revenir). Taux de rotation élevé coûtant cher en perte de connaissances tribales.

- **Formation**: Une génération sacrifiée créant une dette de compétence de décennies. Les "Seniors Creux" avec 3-4 ans d'expérience ont le volume de production d'un senior via IA mais les compétences de débogage d'un junior, incapables de maintenir le code généré qu'ils ne comprennent pas.

- **Économie Indie**: Seuls les solopreneurs maîtrisant la complexité opérationnelle survivent. Coût financier direct de 50% des revenus perdus par churn dû à onboarding lent, plus coût temporel d'"Integration Hell" tuant les projets.

- **Bases de code**: Fragilité structurelle avec assemblages de "code magique" IA que personne ne comprend, créant une dette technique massive invisible nécessitant un "Archaeological Programming" futur.

### Why Existing Solutions Fall Short

Les ressources existantes traitent les symptômes superficiels mais ratent trois gaps structurels critiques :

**1. Silos Disciplinaires - Zéro Vision Holistique**

Les ressources actuelles fragmentent artificiellement les compétences modernes :

- **Tutoriels IA (LangChain, OpenAI docs, Vercel AI SDK)** : Se concentrent sur l'API et la syntaxe ("Comment appeler une fonction") mais ignorent totalement l'architecture logicielle et l'UX. Résultat : des développeurs capables de faire un chatbot qui marche techniquement mais avec une architecture fragile et une UX catastrophique.

- **Cours d'Architecture Logicielle (Clean Code, DDD, SOLID)** : Enseignent des patterns éprouvés mais dans un monde pré-IA. Zéro considération sur comment ces principes s'adaptent quand l'IA génère 70% du code, ou comment architecturer des systèmes avec agents autonomes. Les exemples utilisent des cas classiques (e-commerce, blog) jamais des apps IA-first.

- **Ressources UX (Nielsen Norman, Smashing Magazine, UX Collective)** : Pensées pour designers, pas pour développeurs. Supposent une séparation stricte dev/design et ignorent les contraintes techniques modernes (latence LLM, streaming, states complexes des agents IA). Les développeurs doivent traduire seuls ces principes en implémentation.

**Le gap critique** : Aucune ressource ne montre comment **orchestrer** ces trois piliers ensemble. Comment concevoir l'UX d'une app avec RAG ? Comment architecturer proprement un système multi-agents ? Comment rendre accessible une interface conversationnelle ? Les développeurs naviguent seuls à l'intersection.

**2. Posture "Expert Parfait" - Apprentissage Invisible**

Les ressources techniques mainstream maintiennent une façade d'expertise totale :

- **Documentation officielle** : Assume que vous comprenez déjà les fondamentaux. Zéro transparence sur les pièges, les impasses, le processus d'apprentissage réel. "Voici comment faire X" sans jamais "Voici pourquoi j'ai d'abord essayé Y et pourquoi ça n'a pas marché".

- **Tutoriels YouTube/Cours Udemy** : L'instructeur sait tout, ne se trompe jamais, produit du code parfait du premier coup. Cette perfection artificielle crée syndrome de l'imposteur chez les apprenants qui, eux, galèrent et se sentent incompétents.

- **Blogs tech existants** : Même les blogs "authentiques" montrent surtout les succès finaux. Les échecs sont édulcorés en "lessons learned" post-rationalisation. Le vrai processus chaotique d'apprentissage - les erreurs stupides, les incompréhensions fondamentales, les retours en arrière - reste invisible.

**Le gap critique** : Les autodidactes et mid-levels manquent de **modèles d'apprentissage réalistes**. Ils ne voient que les résultats, jamais le processus. Cela les prive de validation ("c'est normal de galérer") et de stratégies concrètes pour progresser.

**3. Solutions Mono-Niveau - Aucune Progression Claire**

Les ressources s'adressent soit aux juniors, soit aux seniors, jamais aux deux :

- **Tutoriels débutants** : "Votre premier chatbot en 10 minutes" - utiles pour commencer mais laissent sur le bord de la route. Comment passer du prototype jouet à l'architecture production ? Mystère.

- **Articles experts** : Assument que vous maîtrisez déjà SOLID, DDD, les patterns avancés. Impossible pour un junior d'y entrer, décourageant pour un mid-level qui sent qu'il manque des bases.

- **Aucun chemin de progression** : Nulle part on ne voit la trajectoire complète Junior → Mid → Senior documentée en continu sur les 3 piliers. Les développeurs doivent deviner comment monter en compétence.

**Le gap critique** : Les développeurs en transition (bootcamp → mid-level, mid-level → senior) n'ont pas de **roadmap d'apprentissage holistique** montrant comment progresser simultanément en IA, Architecture, et UX.

**4. "Teaching" vs "Learning in Public" - Direction du Flux**

Les ressources existantes adoptent un modèle top-down :

- **Flux descendant** : Expert → Apprenant. L'auteur possède le savoir, le distribue. Aucune co-construction, aucune vulnérabilité.

- **Contenu figé** : Une fois publié, un article reste statique. Pas d'évolution visible de la compréhension de l'auteur. Donne l'illusion que "une fois qu'on sait, on sait pour toujours".

- **Communautés passives** : Les lecteurs consomment, posent des questions, mais ne participent pas à l'apprentissage de l'auteur.

**Le gap critique** : Le modèle "Learning in Public" - où l'auteur apprend **avec** son audience, documente ses erreurs en temps réel, et fait évoluer sa compréhension visiblement - est quasi-inexistant dans l'écosystème francophone tech.

---

**Synthèse : Ce Qui Manque**

Aucune ressource ne combine :
- ✅ **Vision holistique** : IA × Ingénierie × UX intégrés
- ✅ **Authenticité totale** : Échecs, processus, zones d'ombre documentés
- ✅ **Progression continue** : Du junior au senior sur les 3 piliers
- ✅ **Learning in Public** : Apprentissage partagé, vulnérabilité assumée
- ✅ **Francophone moderne** : Adapté au contexte et à la culture d'apprentissage francophone

sebc.dev comble ce vide avec un positionnement unique : **le développeur autodidacte qui consolide ses fondamentaux en public, à l'intersection de trois disciplines, et invite son audience à apprendre avec lui.**

### Proposed Solution

**sebc.dev** : Le **"Blog comme Laboratoire d'Apprentissage Multi-Piliers"** — Une plateforme documentant en temps réel l'apprentissage et la pratique à l'intersection de l'IA, de l'Ingénierie Logicielle, et de l'UX.

**Philosophie "R&D Engineering in Public" :**
- Pas de posture théorique - documentation d'explorations techniques avancées validées en production
- Partage des impasses explorées et des patterns qui en émergent (pour que vous n'ayez pas à les tester)
- Systématisation des apprentissages via écriture technique rigoureuse
- Transparence totale sur le processus d'exploration comme garantie d'authenticité

**Architecture Trois Piliers Égaux :**

**Pilier IA (40% du contenu)** :
- RAG, agents autonomes, prompting avancé
- Intégration IA dans applications Nuxt
- Context engineering et observabilité
- *Focus : Comment orchestrer l'IA, pas juste l'utiliser*

**Pilier Ingénierie Logicielle (30% du contenu)** :
- Patterns d'architecture (Hexagonale, DDD, CQRS)
- Clean Code, SOLID, principes fondamentaux
- Testing, CI/CD, bonnes pratiques
- *Focus : Fondamentaux solides pour déléguer le code à l'IA*

**Pilier UX (30% du contenu)** :
- Design Systems et accessibilité
- Workflows utilisateur et micro-interactions
- UX pour développeurs
- *Focus : Comprendre l'utilisateur pour mieux architecturer*

**Format révolutionnaire "Pattern Onion"** identique au brief original, mais appliqué aux **trois piliers** avec des articles cross-piliers montrant les interconnexions.

**Différenciateur clé** : Documentation **EN DIRECT** d'un parcours d'apprentissage authentique avec :
- Transparence sur les zones d'ombre et d'apprentissage actif
- Métriques d'évolution personnelle avant/après
- Échecs documentés et lessons learned
- Consolidation visible des fondamentaux

### Key Differentiators

**1. Unfair Advantage — Le Praticien en Temps Réel**

Parcours unique impossible à répliquer : Mid-level (3-6 ans) avec expertise IA forte, vivant **maintenant** les frictions documentées dans le rapport fin 2025. Tension quotidienne legacy (environnement professionnel contraint) vs moderne (side projects expérimentaux) offrant une perspective biface rare. Learning in public authentique avec métriques traçables.

**2. Authenticité "Learning in Public" - L'Autodidacte Transparent**

Positionnement unique impossible à copier : documenter un parcours d'apprentissage **en temps réel** avec vulnérabilité assumée. Pas un expert qui enseigne du haut de sa tour d'ivoire, mais un développeur mid-level autodidacte qui :
- Consolide ses fondamentaux **en public** par l'écriture
- Partage son processus d'apprentissage, pas juste les résultats
- Documente ses impasses et incompréhensions
- Apprend en enseignant, enseigne ce qu'il apprend

Cette authenticité crée une connexion unique avec l'audience : juniors et mid-levels se reconnaissent dans le parcours, trouvent validation de leurs propres gaps, et apprennent aux côtés de l'auteur plutôt que passivement.

Le format "Learning in Public" devient un avantage stratégique :
- Crédibilité par vulnérabilité (vs posture d'expertise artificielle)
- Contenu vivant qui évolue avec la compréhension
- Audience co-apprenante engagée vs consommateurs passifs
- Permission de ne pas tout savoir, tout en documentant la progression

**3. Format "Pattern Onion" Incopiable**

Architecture de contenu à entrées multiples servant 3 personas simultanément sans dilution :
- Junior lit Layer 2+3 (Quick Win + Pattern)
- Mid-level lit Layer 3+4 (Pattern + Deep Dive)
- Indie lit Layer 1+2 (Hook + Quick Win actionnable)

Compétiteurs devront choisir une niche ou diluer leur message. Nous capturons la largeur via l'architecture, pas le compromis.

**4. Authenticité Bilingue FR/EN**

Voix unique documentant la pensée technique en deux langues, capturant nuances culturelles d'apprentissage et élargissant portée géographique sans traduction mécanique.

**5. Anti-Stack Philosophy**

Positionnement contre-intuitif dans un marché saturé d'outils : "Votre problème n'est pas le manque d'outils, mais l'excès de complexité non comprise". Réduit friction par soustraction et compréhension, pas addition et automatisation.

**6. Timing Parfait — Fenêtre de 6-18 Mois**

Le marché est **maintenant** prêt : crise visible, douleur universelle, recherche active de solutions, mais avant que les grands acteurs (Vercel, Supabase, GitHub) ne saturent l'espace documentaire. Documenter en temps réel = avantage temporel non rattrapable.

---

## Target Users

**sebc.dev** sert trois segments d'utilisateurs primaires, chacun confronté à des frictions spécifiques dans l'écosystème de développement IA de fin 2025. Ces personas ne sont pas des cas d'usage théoriques, mais des profils réels représentant la majorité des développeurs modernes naviguant le paradoxe de l'efficacité.

### Primary Users

#### Persona 1: Lucas, "Le Bâtisseur Pragmatique" (Mid-Level Developer)

**Identité et Contexte**

- **Nom**: Lucas D.
- **Âge**: 27 ans
- **Poste**: Développeur Fullstack (Vue.js/Nuxt & Node.js)
- **Environnement**: EcoLogistics, startup SaaS B2B (Série A) optimisant les tournées de livraison, équipe tech de 8 personnes
- **Expérience**: 4 ans (bootcamp + alternance → Mid-Level depuis 1 an), autonome sur 80% des tâches mais manque de vision architecturale macro

**Profil Détaillé**

- **Quotidien**: Code depuis la fin de ses études, aime quand "ça marche" et que le code est propre, mais constamment pressé par deadlines produit
- **Motivation**: Être reconnu comme celui qui "ship" rapidement sans casser la prod. Découvrir des astuces qui le font passer pour un magicien auprès de ses collègues
- **Aspirations**:
  - Court terme: Ne plus perdre de temps sur configuration et boilerplate
  - Moyen terme: Devenir Senior Developer ou Tech Lead, être la référence pour les juniors
  - Non-objectifs: Pas manager ou founder pour l'instant, veut coder mieux et plus vite
- **Devise**: "Je veux bien intégrer de l'IA partout, mais je ne veux pas passer ma vie à déboguer des JSON mal formatés"

**Sa Journée Type (Fin 2025)**

- **09h30 - Stand-up**: Annonce blocage sur intégration chatbot assistant avec vieille base SQL legacy
- **10h00 - Deep Work (La lutte)**: Tente de connecter API LLM avec backend Node.js 2022. LLM renvoie données que front Nuxt n'arrive pas à typer correctement. Chaos des types TypeScript
- **11h30 - La recherche (Point de contact)**: Frustré, recherche "Nuxt AI stream integration with legacy API best practices". Tombe sur article sebc.dev
- **14h00 - Implémentation**: Copie-colle pattern (composable) trouvé sur blog, l'adapte en 15 minutes. Ça marche
- **16h00 - Code Review**: Soumet PR fièrement, code plus propre que celui du Senior voisin

**Expérience du Problème (Friction "Glue Code")**

- **Problème concret**: Rajouter intelligence (RAG, Agents) sur architecture non prévue pour ça
- **La friction**: 70% du temps à écrire code "glu" : nettoyer réponses LLM, gérer timeouts, forcer typage, gérer erreurs d'hallucination crashant UI Nuxt
- **Workarounds actuels**: Console.log partout, `any` dans TypeScript (avec culpabilité), regex fragiles pour parser réponses IA
- **Impact émotionnel**: Se sent incompétent alors que problème est systémique. Impression de bricoler au lieu d'ingénier. Fatigué de la hype IA qui rajoute du travail sale

**Vision de Succès avec sebc.dev**

- **Le déclic**: "C'est exactement ce dont j'avais besoin: pas de théorie fumeuse sur le futur de l'IA, mais un bout de code Nuxt utilisable maintenant pour gérer ce stream de données"
- **Mesure du succès**:
  - Temps gagné: Tâche de 4h pliée en 1h
  - Stabilité: Application ne crashe plus quand IA répond bizarrement
  - Apprentissage: Comprend pourquoi ça marche et peut l'expliquer en Code Review (aide à viser poste Senior)

---

#### Persona 2: Chloé, "L'Apprentie Copilote" (Junior Developer)

**Identité et Contexte**

- **Nom**: Chloé M.
- **Âge**: 24 ans
- **Formation**: Reconversion pro, Bootcamp intensif (type Le Wagon/Ironhack) 2024 + beaucoup d'auto-formation
- **Expérience**: 10 mois de code effectif
- **Poste**: Développeuse Junior (Alternance ou premier CDD) en Agence Web Digitale
- **Contexte**: Rythme soutenu, doit livrer fonctionnalités pour clients variés. Souvent seule face à son écran, tuteur très occupé

**Rapport à l'IA (Le Piège du "Hollow Senior")**

- **Usage actuel**: Code avec IA ouverte en permanence (Cursor ou Copilot v5). Coder = prompter: "Génère-moi un composant Nuxt qui fait X"
- **Piège du "Vibe Coding"**: Subit inconsciemment. Juge qualité du code au "feeling": si ça s'affiche sans erreur rouge, c'est bon. Navigue à vue
- **Frustration**: Impression d'être une "imposture". Capable de produire code semblant Senior (complexe, structuré), mais panique si on demande d'expliquer ligne 42. Sent qu'elle construit châteaux de cartes

**Expérience du Problème**

- **Incapacité à découvrir patterns**: IA donne solution finale immédiatement. Saute étape cruciale de "recherche et échec" permettant apprentissage. Ne voit pas le pattern, seulement le résultat
- **Le mur du débogage**:
  - **Scénario typique**: IA a généré composable Nuxt complexe pour gérer état. Bug silencieux apparaît (boucle infinie useEffect, problème hydratation)
  - **Réaction**: Repaste erreur dans IA. IA hallucine correction créant autre bug. Chloé bloquée. Ne comprend pas mécanique interne pour réparer elle-même
- **Impact émotionnel**:
  - Anxiété élevée avant Code Reviews
  - Syndrome de l'imposteur violent: "Je ne suis pas une dév, je suis juste opératrice de prompt"
  - Peur d'être remplacée par IA plus performante puisqu'elle n'apporte pas de valeur ajoutée structurelle

**Vision de Succès avec sebc.dev**

- **Son rôle**: Mentor virtuel prenant le temps de décortiquer la magie
- **"Aha Moment" (Déclic Pattern Onion)**:
  - Lit article, voit d'abord code qu'elle a l'habitude de générer (couche externe oignon)
  - Puis vous montrez ce qu'il y a dedans
  - Réaction: "Ah! Donc l'IA utilisait shallowRef ici pour optimiser perf, pas juste par hasard! Je comprends enfin la différence"
- **Mesure de progression**:
  - Réussit à refactoriser bloc de code généré par IA pour l'adapter sans redemander à l'IA
  - Capable d'expliquer le "pourquoi" d'une architecture lors stand-up
- **Pourquoi elle revient**: Vous ne la jugez pas d'utiliser l'IA, mais l'aidez à la dompter. Donnez clés pour passer de "Passagère" à "Pilote". Contenu la rassure: valide qu'elle peut devenir vraie ingénieure

---

#### Persona 3: Maxime, "L'Architecte Frankenstein" (Indie Hacker/Freelance)

**Identité et Contexte**

- **Nom**: Maxime R.
- **Âge**: 32 ans
- **Background**: Ancien Senior Dev agence, a tout quitté il y a 2 ans pour liberté Indie Hacking. Techniquement très solide mais manque rigueur opérationnelle
- **Statut**: Solopreneur / Freelance à mi-temps
- **Projets en cours**:
  - 3 micro-SaaS: 1 générant 2k€ MRR, 1 "zombie" qu'il n'ose pas tuer, 1 nouveau en lancement
  - 2 jours freelance/semaine pour sécurité financière
- **Modèle économique**: Vit dans terreur que SaaS principal casse pendant mission freelance. MRR couvre frais mais charge mentale d'équipe de 5 personnes

**Stack Actuelle (Le Chaos "Frankenstein")**

- **Réalité**: Pour aller vite, à chaque projet a pris outil "tendance" du moment
- **Inventaire (15+ services)**:
  - Backend: Supabase (projet A), Firebase (projet B), API Node custom sur Railway (projet C)
  - IA: OpenAI API + Pinecone (Vector DB) + LangChain (jamais mis à jour)
  - Ops: Vercel (Hosting), Resend (Emails), Stripe (Paiement), Sentry (Logs), Axiom, Crisp (Chat support)
- **Santé système**: Aucune vue d'ensemble. 15 onglets ouverts en permanence. S'appuie sur emails d'alerte de chaque service. Si Stripe n'alerte pas, suppose que tout va bien (alors qu'API peut être down)

**Expérience du Problème ("Integration Hell")**

- **Friction quotidienne**: Passe plus de temps à faire communiquer outils entre eux (API keys, webhooks échouant, formats JSON différents) qu'à coder valeur pour utilisateurs
- **Scénario typique (Cauchemar)**:
  - Client se plaint: "L'IA ne répond pas"
  - Doit vérifier: Vercel timeout? OpenAI down? Supabase lent? Clé API expirée?
  - Pas de Dashboard Central. Doit se logger sur 4 services différents pour tracer une seule requête
- **Impact Financier & Mental**:
  - Paie ~400€/mois abonnements SaaS, souvent fonctionnalités redondantes qu'il pourrait coder lui-même en Nuxt avec bon template
  - Souffre "Notification Fatigue". Ignore parfois alertes critiques à cause spam de ses propres outils

**Vision de Succès avec sebc.dev**

- **Son rôle**: Partenaire aidant à rationaliser et simplifier
- **"Aha Moment"**: Découvre article "Arrêtez de payer pour ça: Construisez votre Dashboard Monitoring IA dans Nuxt en 30 min" ou "Pattern Monolith Modulaire: Gérer 3 projets Nuxt avec un seul backend"
- **Mesure du ROI**:
  - Économies directes: Résilie 3 abonnements SaaS inutiles car a implémenté solution légère
  - Sérénité: A enfin panneau admin "Maison" montrant état santé écosystème complet en un coup d'œil (Vert/Rouge)
- **Pourquoi il revient**: Vous prônez sobriété technique. Contrairement aux influenceurs poussant nouveaux outils, vous montrez comment faire plus avec moins. Vous êtes antidote à la complexité

---

### User Journey

#### Discovery (Découverte) - La Porte d'Entrée

En 2025, la recherche n'est plus seulement "Mots-clés Google" mais conversationnelle et communautaire.

**Lucas (Mid-Level): L'approche "Search-First"**
- **Contexte**: En plein débogage, le feu au lac
- **Canal**: Interroge IA de recherche (Perplexity, SearchGPT, Gemini Integrated) ou requête Google technique précise: "Nuxt server route streaming response type error"
- **Le lien**: IA cite votre article comme source principale ("Based on sebc.dev..."), ou clique lien position 2 Google car titre contient exactement son message d'erreur

**Chloé (Junior): L'approche "Social & Mentor"**
- **Contexte**: Cherche à comprendre concept flou après journée travail
- **Canal**: Traîne sur Discords communautaires (Vue.js France, Nuxt Nation) ou voit thread BlueSky/LinkedIn où senior partage "Meilleures ressources pour comprendre l'IA en JS"
- **Le lien**: Recommandation humaine: "Regarde ce blog, il explique super bien le cycle de vie, pas comme doc officielle trop sèche"

**Maxime (Indie Hacker): L'approche "Veille Stratégique"**
- **Contexte**: Scrolle X (Twitter) ou lit Newsletter Tech/Business (Indie Hackers, newsletter spécialisée Vue)
- **Canal**: Attiré par titre orienté "Gain": "Comment j'ai économisé 200$/mois de SaaS avec ce pattern Nuxt"

#### Onboarding (Première Expérience) - Les 30 Premières Secondes

Le taux de rebond se joue ici.

**Lucas: Le Scan Rapide**
- **Premier Article**: Tutoriel technique ("How to fix X")
- **Comportement**: Ne lit pas intro. Scrolle directement vers blocs de code
- **Le crochet**: Présence bloc de code copiable, propre et typé. Si code sale ou trop de blabla, il part. Reste car solution immédiatement applicable

**Chloé: La Lecture Profonde**
- **Premier Article**: Article de fond (type "Pattern Onion")
- **Comportement**: Lit intro. Regarde schémas. Apprécie mise en forme aérée
- **Le crochet**: Ton bienveillant. "Enfin quelqu'un qui ne me parle pas comme à un ordinateur". Reste car comprend enfin ce qui lui échappait

**Maxime: L'Audit de Valeur**
- **Premier Article**: Étude de cas architecturale ou comparatif
- **Comportement**: Cherche conclusion ou section "Résultats"
- **Le crochet**: Promesse de simplification. "Pas besoin de Redis pour ça? Intéressant." Reste car vous validez son envie de minimalisme

#### Core Usage (Routine) - L'Intégration dans le Workflow

Comment sebc.dev devient un outil pour eux.

**Lucas: L'Antisèche (Cheat Sheet)**
- **Fréquence**: Ad-hoc, quand démarre nouvelle feature IA
- **Workflow**: Site en favori dossier "Dev Resources". Vient chercher snippet, copie, adapte en 2 min, ferme onglet
- **Device**: Desktop (double écran bureau/télétravail)

**Chloé: Le Manuel Scolaire**
- **Fréquence**: Hebdomadaire (souvent week-end ou soir pour se former)
- **Workflow**: Ouvre projet "bac à sable" à côté. Suit article pas à pas pour reproduire mécanisme. Prend notes
- **Device**: Laptop personnel (café ou canapé)

**Maxime: Le Blueprint (Plan d'architecte)**
- **Fréquence**: Par cycle de projet (tous 1-2 mois) ou lors refonte
- **Workflow**: Lit article pour valider choix techniques avant coder. Utilise vos arguments pour se convaincre (ou convaincre client freelance)
- **Device**: Mobile (veille transports) puis Desktop pour implémenter

#### Success Moment ("Aha!") - La Validation Ultime

**Lucas**: Copie-colle votre composable `useLLMStream` et pour première fois, stream s'affiche lettre par lettre dans UI sans lag ni erreur TypeScript
- **Pensée**: "Putain, c'est propre. Je garde ce site."

**Chloé**: Corrige bug sur projet pro avant que senior ne le voie, en utilisant explication lue chez vous la veille
- **Pensée**: "Je savais ce qu'il fallait faire! Je ne suis pas nulle!"

**Maxime**: Supprime abonnement SaaS à 29$/mois car a implémenté votre solution "DIY" en une après-midi
- **Pensée**: "Ce gars me fait gagner de l'argent."

#### Long-term (Fidélisation) - Devenir Ambassadeur

**Lucas: L'Abonné Passif**
- S'abonne à newsletter pour recevoir "snippets du mois"
- Partage rarement, sauf si collègue galère sur même problème ("Tiens, regarde là-dessus")

**Chloé: La Fan Active**
- Commente pour remercier
- Partage articles sur LinkedIn avec texte sur ce qu'elle a appris
- Devient ambassadrice de votre marque auprès autres juniors

**Maxime: Le Partenaire Potentiel**
- Respecte votre expertise technique
- Pourrait contacter pour consulting pointu ou acheter template "Starter Kit" complet si vous en sortez, car sait que qualité sera au rendez-vous

**Adaptation du Pattern Onion par Persona:**

**Pour Maxime (Indie Hacker):**
- **Peau (Quick Copy)**: Code autonome, 1 fichier max, zéro dépendance custom
- **Chair (Simple Implementation)**: Architecture la plus simple qui fonctionne
- **Noyau (Why Simple Works)**: Pourquoi cette simplicité est suffisante pour un solo project
- **Exemple**: "Auth Pattern: 1 middleware + 1 composable = done"

**Pour Lucas (Mid-level en équipe):**
- **Peau (Quick Copy)**: Module complet copy-paste
- **Chair (Team-Ready Implementation)**: Separation of Concerns pour collaboration
- **Noyau (Architecture Rationale)**: Pourquoi DDD/Hexagonal ici (maintainabilité équipe)
- **Exemple**: "Auth Module: Layers Repository/Service/Controller pour tests isolés"

**Pour Chloé (Junior):**
- **Peau (Quick Copy)**: Code fonctionnel basique
- **Chair (Learning Implementation)**: Code commenté, étapes explicites
- **Noyau (Deep Concepts)**: Principes fondamentaux (closures, reactivity, lifecycle)
- **Exemple**: "Auth Basics: Comment fonctionne un JWT (expliqué à fond)"

**Le même sujet, trois profondeurs différentes.**

---

## Success Metrics

Le succès de **sebc.dev** se mesure à deux niveaux interconnectés: la valeur créée pour chaque persona utilisateur, et l'impact business stratégique sur le personal branding et les opportunités générées.

### User Success Metrics

**Lucas (Mid-Level): "Time-to-Commit"**

Le blog est un outil de productivité, pas de loisir. Lucas cherche à débloquer une tâche Jira et passer du statut "Bloqué" à "En Review" rapidement.

- **Test du succès**: Le "Copier-Coller-Marcher" — Il copie le snippet, le colle dans VS Code, et TypeScript ne génère aucune erreur. Ça compile du premier coup.
- **Moment "Aha!"**: La seconde précise où il voit les logs verts dans sa console après intégration du code. Pensée: "Ok, ce mec gère les edge-cases que j'avais oubliés. C'est du code de prod."
- **Indicateurs comportementaux**:
  - High Click-through rate sur boutons "Copy" des snippets
  - Low Time-on-Page (2-3 min) mais avec copie de code = succès (pas de temps perdu)
  - Bookmark/Star de l'article pour référence future
- **Métrique clé**: Le "Copy Rate" — Pourcentage de visiteurs qui copient effectivement le code

**Chloé (Junior): "Confidence Gain"**

Le blog est un tuteur. Elle consomme les couches internes de l'oignon Pattern pour comprendre le "pourquoi".

- **Test du succès**: Le test de la "Prédiction" — En lisant l'explication, elle anticipe: "Ah, donc à l'étape d'après, il va sûrement utiliser onUnmounted pour nettoyer la mémoire". Elle commence à prédire les patterns.
- **Indicateur de progression**: Elle arrête de "subir" le code. Capable de modifier le code fourni, pas juste l'utiliser tel quel.
- **Moment "Aha!"**: Quand elle réalise qu'elle comprend la mécanique sous-jacente et peut l'adapter à son contexte spécifique.
- **Indicateurs comportementaux**:
  - Scroll Depth 100% (va jusqu'en bas, lit les "Pourquoi" pas juste le code)
  - Temps passé élevé (8-12 min) = bon signe, elle étudie
  - Interaction sociale validante: Commente ou partage "J'ai enfin compris X grâce à cet article"
- **Métrique clé**: "Complétion de Lecture" — Elle consomme tout l'Oignon, pas juste la peau

**Maxime (Indie Hacker): "Simplified Stack"**

Le blog est un consultant en architecture gratuit. La valeur se mesure par soustraction.

- **Test du succès ROI**: Le contenu a de la valeur s'il lui permet d'enlever quelque chose de sa stack (outil payant, librairie lourde, étape de déploiement).
- **Signes de rationalisation**:
  - Remplace un SaaS externe (ex: service de logs payant) par pattern "Nuxt Server Logs"
  - Refactorise deux applis pour utiliser "Shared Layer" proposé
- **Moment "Aha!"**: Quand il calcule les économies réalisées grâce à un pattern et se dit "Ce gars me fait gagner de l'argent".
- **Indicateurs comportementaux**:
  - Clics sortants "négatifs": Ne clique pas sur liens d'affiliation outils tiers (fait sans)
  - Téléchargement de Templates/Boilerplates proposés
  - Retour récurrent cyclique (tous les 3 mois, cycle de projet)
- **Métrique clé**: "Adoption d'Architecture" — Il restructure son projet selon vos plans

### Business Objectives

**Objectifs à 3 mois: "La Validation et l'Amorçage"**

Phase de sortie du bois et preuve d'utilité. Vision de succès: Dépasser le stade du "site fantôme" — des inconnus (pas juste amis) atterrissent, trouvent solution, copient code.

- **Contenu**: 6-8 articles "Piliers" publiés (2 pour Lucas, 2 pour Chloé, 2 pour Maxime). Effet "bibliothèque" immédiat.
- **Trafic**: 500-1000 Visiteurs Uniques/mois. Modeste mais suffisant pour valider la niche.
- **SEO & Tech**:
  - Indexation sur requêtes "Long tail" spécifiques (ex: "Nuxt 4 AI stream hydration mismatch")
  - Score Lighthouse: 100/100 (indispensable pour crédibilité technique auprès de Lucas)
- **Communauté**: 50 premiers abonnés newsletter (hors cercle proche) = "True Fans"

**Objectifs à 12 mois: "L'Autorité de Niche"**

Phase de référence incontournable sur intersection Nuxt/IA/Legacy. Vision de succès: On ne cherche plus "comment faire du stream en Nuxt", on cherche "si sebcdev a écrit un truc dessus".

- **Audience**:
  - 5000-10000 Visiteurs Uniques/mois (audience qualifiée développeurs)
  - Newsletter: ~500 abonnés avec taux d'ouverture > 40%
- **Reconnaissance**:
  - Cité dans newsletter officielle Nuxt Weekly ou Vue.js News
  - Au moins 1 invitation podcast tech (Double Slash, WeLoveDevs, ou équivalent)
- **Opportunités**: 2-3 demandes entrantes (inbound)/mois pour consulting ou freelancing senior, sans prospection

### Key Performance Indicators

**A. Métriques de Croissance (Acquisition)**

Comment les gens trouvent sebc.dev.

- **Référencement IA (AI Search Referrals)** — Nouveau KPI 2025: Trafic venant de Perplexity, ChatGPT, Gemini. Indique que contenu considéré comme "source de vérité" par LLMs.
- **Trafic Organique (SEO classique)**: Croissance mensuelle (MoM) de 10%
- **Abonnés Newsletter (Owned Audience)**: Métrique la plus stable. Objectif: Conversion de 2% des visiteurs uniques en abonnés

**B. Métriques d'Engagement (Rétention & Qualité)**

Preuve que format "Pattern Onion" fonctionne.

- **Le "Copy Rate" (Pour Lucas)**: Tracker sur boutons "Copier le code". Si 15% des visiteurs copient snippet = code utile
- **Le Temps de Lecture (Pour Chloé)**: Viser > 3 minutes sur articles de fond. Prouve qu'elle lit explications, pas juste titre
- **Le Taux de Retour (Pour Maxime)**: Pourcentage visiteurs avec > 2 sessions. Si > 20% = devenu ressource de référence (Cheat Sheet)

**C. Métriques Financières (Modèle Hybride)**

Construire valeur sans vendre son âme.

**Consulting / Freelance (Principal)**: Augmentation TJM (Taux Journalier Moyen) de 20% grâce à crédibilité technique du blog ("Je vous veux vous, parce que vous avez déjà résolu ces problèmes en production et documenté les solutions")
- **Produits Digitaux (Secondaire - Pour Maxime)**:
  - Lancement "Nuxt AI Starter Kit" payant (49€) au mois 9
  - Objectif: Vendre 10 licences/mois (couvre frais hébergement et outils)
- **Sponsoring (Non intrusif)**:
  - Refuser pubs AdSense
  - Accepter uniquement sponsors techniques alignés (hébergeur spécialisé Node, outil monitoring) après 1 an

**D. Métriques Stratégiques (Personal Branding)**

Positionnement "Context Engineering".

- **Backlinks de Qualité**: Nombre domaines tech (GitHub, StackOverflow, blogs dev) pointant vers vous
- **Mentions Sociales**: Être tagué sur LinkedIn/X/BlueSky par développeurs disant "Lisez ça pour comprendre le problème"
- **Influence sur "Context Engineering"**: Si d'autres développeurs utilisent votre vocabulaire ("Legacy Glue", "Pattern Onion") = victoire ultime

---

## MVP Scope

### Core Features

**Plateforme Technique de Contenu Multi-Piliers**

sebc.dev est un blog technique Nuxt avec architecture optimisée pour le format "Pattern Onion" servant trois domaines d'expertise interconnectés : IA, Ingénierie Logicielle, et UX.

**Fonctionnalités Essentielles MVP:**

**1. Architecture de Contenu Multi-Piliers**
- Blog Nuxt avec affichage articles, recherche/filtres basiques, pagination, SSR
- Contenu bilingue FR/EN avec @nuxtjs/i18n (core value proposition)
- **Organisation par 3 piliers égaux : IA (40%), Ingénierie Logicielle (30%), UX (30%)**
- Articles cross-piliers mettant en valeur l'intersection des trois expertises
- Système de tags permettant navigation par pilier ET par thème transversal

**2. Format "Pattern Onion" Différenciant**
- **Table des matières interactive avancée** (composant central du MVP):
  - Sidebar sticky avec highlighting section active
  - Barre de progression visuelle (0-100%)
  - Temps de lecture estimé par section
  - Smooth scroll vers anchors
  - Génération automatique depuis h2/h3
  - Navigation rapide vers sections clés ("Live Demo", "Comparatif coûts")
- Sections `<details>`/`<summary>` pour révélation progressive du contenu approfondi
- Hiérarchie visuelle h2/h3 montrant la profondeur du Pattern Onion

**3. Expérience Développeur (Pour Lucas - Mid-Level)**
- Blocs de code avec bouton "Copy" + syntax highlighting (Shiki/Prism)
- Badge langue sur chaque snippet
- Embed CodeSandbox/StackBlitz pour démonstrations live côte à côte
- Navigation ToC rapide vers sections "Live Demo"

**4. Expérience Apprentissage (Pour Chloé - Junior)**
- ToC montrant structure pédagogique complète
- Indicateur de progression pour visualiser l'avancement
- Révélation progressive via `<details>` pour éviter surcharge cognitive
- Highlight de la section active pour contexte constant

**5. Expérience Architecture (Pour Maxime - Indie Hacker)**
- Téléchargement direct de templates (liens GitHub gists/repos)
- Tableaux comparatifs intégrés (solutions/coûts/performances)
- Ancres directes ToC vers sections "Résultats" et "Comparatifs"
- Temps de lecture par section pour audit rapide

**6. Standards Techniques Non-Négociables**
- **Mode sombre** (@nuxtjs/color-mode) - standard attendu par développeurs
- **SEO complet**: useSeoMeta, sitemap, structured data, hreflang bilingue
- **Performance**: Lighthouse > 90 tous scores (SSR Nuxt + @nuxt/image)
- **Responsive**: Expérience parfaite mobile/desktop
- **Admin fonctionnel**: Nuxt Content pour publication facile

**7. Analytics Essentielles**
- Plausible Analytics (GDPR-friendly) avec tracking:
  - Pageviews + temps lecture moyen
  - Clics ToC (valide utilité Pattern Onion)
  - Scroll depth par article
  - Taux rebond par pilier (IA, Ingénierie, UX)
  - Copy Rate (clics boutons "Copy code")
- Segmentation par pilier pour valider équilibre trafic

**8. Contenu de Lancement**
- **Minimum 3 articles piliers excellents** (qualité > quantité):
  - 1 article IA (ex: "RAG avec Nuxt + Cloudflare")
  - 1 article Ingénierie Logicielle (ex: "Architecture Hexagonale en TypeScript")
  - 1 article UX (ex: "Design System accessible avec Tailwind")
- Testés par 2-3 early readers avant publication

**9. Newsletter (Basique)**
- Formulaire embed simple (Buttondown/Mailchimp iframe)
- Objectif conversion: 2% des visiteurs uniques
- Automation avancée reportée à v2.0

**Composants Techniques Clés:**
- `TableOfContents.vue` (sidebar sticky interactive)
- `useScrollSpy` composable (tracking position scroll)
- `ProgressBar.vue` (barre progression lecture)
- Génération automatique anchors depuis headings
- `CodeBlock.vue` avec copy button et syntax highlighting

---

### Out of Scope for MVP

**Fonctionnalités Reportées v2.0+:**

**Engagement Communautaire:**
- ❌ Système de commentaires intégré (Giscus peut être ajouté en 30min post-MVP si besoin)
- ❌ Forum ou plateforme de discussions
- ❌ Communauté Discord (prévu M24+ selon roadmap)

**Contenu Avancé:**
- ❌ Séries d'articles liés automatiques (simulation manuelle avec tags suffit)
- ❌ "Related articles" intelligent avec ML (logique basique même catégorie/tags suffit)
- ❌ Système de bookmarks/reading list (nécessite authentification)
- ❌ Contenu vidéo/podcast intégré (focus 100% écrit, embeds YouTube OK si nécessaire)

**Distribution & Monétisation:**
- ❌ Newsletter automation avec segmentation/personnalisation
- ❌ Cross-posting automatique Dev.to/Medium (distribution manuelle suffit)
- ❌ Templates/Starter Kits payants (prévu mois 9+ selon roadmap)
- ❌ Cours vidéo premium
- ❌ Membership/communauté payante

**Technique Avancé:**
- ❌ Recherche full-text avec Algolia (recherche côté client suffit)
- ❌ API publique (aucun use case identifié)
- ❌ Authentification visiteurs/dashboard utilisateur (seulement admin auth pour auteur)
- ❌ Analytics avancées (heatmaps, session recordings, A/B testing)

**Rationale:**
Focus MVP sur l'expérience de lecture Pattern Onion et validation du positionnement multi-piliers (IA × Ingénierie × UX). Les fonctionnalités sociales/communautaires viennent après preuve de l'audience et de l'engagement.

---

### MVP Success Criteria

**Horizon de Validation: 6 mois d'observation**

**Seuils de Validation Globaux:**

- ✅ **500+ Visiteurs Uniques/mois** au mois 6
- ✅ **15-18 articles publiés** répartis équitablement :
  - ~6-7 articles IA (RAG, agents, LLM tooling)
  - ~5 articles Ingénierie Logicielle (architecture, clean code, patterns)
  - ~4-5 articles UX (design systems, accessibilité, workflows)
- ✅ **>3min temps lecture moyen** (preuve d'engagement profond)
- ✅ **>60% scroll depth moyen** (Pattern Onion fonctionne sur les 3 piliers)

**Métriques par Pilier (Validation Équilibre Multi-Expertise):**

| Métrique | Seuil Validation |
|----------|------------------|
| **Trafic global** | 500+ UV/mois M6 |
| **Équilibre piliers** | Chaque pilier attire 25-45% du trafic (pas de monopole) |
| **Copy Rate** | >10% visiteurs (code IA + Ingénierie Logicielle) |
| **Temps lecture UX** | >3min (audience moins technique mais engagée) |
| **Temps lecture Ingénierie** | >5min (contenu dense, lecture approfondie) |
| **Taux retour** | >15% visiteurs récurrents (deviennent co-apprenants) |
| **Mentions sociales** | 5+ partages/mois avec retours de "j'apprends avec toi" |

**Signaux Qualitatifs d'Apprentissage Partagé :**
- Commentaires du type "Merci d'être transparent sur ton processus"
- "Je me retrouve dans tes galères, ça me rassure"
- "J'ai appris autant de tes erreurs que de tes solutions"
- Backlinks de blogs similaires "Learning in Public"

**Métriques d'Évolution Personnelle (Transparence):**
- Articles "Before/After" montrant progression sur un sujet
- Séries "Fondamentaux" documentant mon apprentissage (ex: "Je (ré)apprends SOLID")
- Mises à jour d'anciens articles avec nouvelles compréhensions

**Points de Décision:**

**SCALE UP si:**
- ✅ Équilibre 3 piliers maintenu (aucun <20% ou >50% du trafic)
- ✅ Audience engage avec le format "Learning in Public" (commentaires positifs)
- ✅ Plaisir d'écriture sur les 3 sujets reste intact
- ✅ Consolidation fondamentaux visible (je maîtrise mieux qu'au début)

**PIVOT/STOP si:**
- ❌ 1 pilier monopolise >70% trafic (perte identité multi-expertise)
- ❌ Burnout sur l'un des 3 piliers (devient corvée)
- ❌ Syndrome de l'imposteur paralysant (posture "apprentissage" devient fardeau)
- ❌ Pression vers posture "expert" dilue authenticité

**Alerte Équilibre Piliers:**
- 1 pilier <15% trafic M3 → Revoir contenu/distribution ce pilier
- 1 pilier >70% trafic M6 → Risque mono-thématique, dilution positionnement unique

---

### Future Vision

**Vision 2027 (2-3 ans): L'Intersection IA × Ingénierie × UX**

**Vision 2027 (2-3 ans): L'Intersection IA × Ingénierie × UX**

**Positionnement Unique:**
- **Taille audience**: 2000+ Visiteurs Uniques/mois (audience qualifiée, engagée)
- **Identité**: "Développeur full-stack holistique en apprentissage continu"
- **Reconnaissance multi-disciplinaire**: Référence francophone pour développeurs naviguant l'intersection des 3 piliers

**Évolution du Positionnement "R&D Engineering in Public":**
- M0-M12: "Explorateur systématique" - Validation terrain des fondamentaux à l'intersection des 3 piliers
- M12-M24: "Ingénieur R&D confirmé" - Patterns production-ready documentés et battle-tested
- M24+: "Architecte holistique" - Capable de guider les équipes sur les 3 piliers avec expertise éprouvée
- Maintien permanent de la transparence sur explorations en cours (frontières technologiques)

**Contenu par Pilier (Vision Complète):**

**IA (40%):**
- RAG production-ready, agents autonomes
- Nuxt + LangChain, Vercel AI SDK
- Context engineering, observabilité
- Edge AI (Cloudflare Workers AI)

**Ingénierie Logicielle (30%):**
- Architecture hexagonale, DDD, CQRS
- Clean Code, SOLID, Design Patterns
- Testing strategies, CI/CD best practices
- Java/Spring en complément Nuxt (backend robuste)

**UX (30%):**
- Design Systems (tokens, composants réutilisables)
- Accessibilité (WCAG, ARIA, tests utilisateurs)
- UX Writing, Micro-interactions
- IA dans workflows UX (Figma plugins, prototyping)

**Articles Cross-Piliers (Valeur Unique):**
- "Architecture RAG avec UX First" (IA + UX + Ingénierie)
- "Design System pour Apps IA" (UX + IA)
- "Clean Code pour Components Accessibles" (Ingénierie + UX)
- "De Junior à Tech Lead: Maîtriser les 3 Piliers" (Meta apprentissage)

**Vision Ultime:**
sebc.dev devient la référence francophone pour développeurs cherchant à évoluer au-delà du code pur, en maîtrisant l'intersection IA × Ingénierie × UX, à travers un format authentique "Learning in Public" qui normalise l'apprentissage continu et la transparence.

**Reconnaissance par Pilier:**

**IA:**
- Contributeur référencé dans écosystème Nuxt/AI
- GitHub repos avec étoiles communauté
- Membre actif discussions RAG/agents production

**Ingénierie Logicielle (avec tags de complexité):**

**[INDIE] Patterns Légers:**
- Composables Nuxt patterns (simple, 1-file solutions)
- Server Routes architecture (direct, no layers)
- Minimal testing (Vitest essentials)
- Simple deployment (Cloudflare Pages, Railway)

**[TEAM] Patterns Avancés:**
- Architecture hexagonale/DDD (when complexity justifies)
- CQRS for complex domains (event sourcing)
- Testing strategies (comprehensive coverage)
- CI/CD best practices (multi-env pipelines)

**[SCALING] Migration Paths:**
- "From Indie to Team": Refactoring progressif
- "When to Add Complexity": Decision trees
- "Architecture Patterns Comparison": Simple vs DDD vs Hexagonal

**UX:**
- Articles republiés par UX Collective, Smashing Magazine
- Référence pour "UX pour développeurs"
- Design systems accessibles reconnus

**Monétisation (500-1000€/mois):**
- **60% Sponsoring**: GitHub Sponsors, articles sponsorisés multi-piliers alignés
- **30% Produits digitaux segmentés**:
  
  **Pour Indie Hackers (Maxime) - Quick Ship:**
  - "Nuxt Indie Starter" (1 fichier auth + logs + payments, 29€)
  - "RAG 1-File Pattern Collection" (copy-paste ready, 19€)
  - "Deployment Scripts Bundle" (Cloudflare/Railway/Fly, 15€)
  
  **Pour Teams (Lucas) - Production-Ready:**
  - "Nuxt Enterprise Architecture Template" (DDD/Hexagonal, 79€)
  - "Testing Strategy Boilerplate" (Unit/Integration/E2E, 49€)
  - "CI/CD Pipeline Templates" (GitHub Actions complet, 39€)
  
  **Cross-Segment:**
  - "Scaling Guide: Indie → Enterprise" (migration path, 59€)

**Évolution MVP → Vision Complète:**

**M6** → Validation métriques (équilibre 3 piliers confirmé)

**M9** → Newsletter gratuite (deep-dive hebdo alternant IA/Ingénierie/UX)

**M12** → Premier produit cross-pilier (Template Nuxt avec IA + Architecture + Design System, 40€)

**M18** → SaaS MVP (outil IA avec best practices Ingénierie + UX intégrées)

**M24** → Communauté Discord payante (3 expertises = valeur unique, 5€/mois)

**Capacités Ajoutées avec Ressources (Priorité 1-5):**

1. **SaaS outil dev IA**: Agent builder + RAG playground + UX analytics intégré
2. **Communauté Discord**: 3 channels (#ia, #architecture, #ux-design)
3. **Cours vidéo premium**: "IA + Clean Code + UX" bundle holistique (150€)
4. **Templates multi-piliers**: Starter Nuxt avec IA + Design System + Architecture modulaire (40€)
5. **Newsletter premium**: Deep-dives hebdo alternant 3 piliers (5€/mois)

**Nouveaux Segments Utilisateurs (dès MVP):**

- **Développeurs IA** cherchant bonnes pratiques architecture
- **Architectes logiciels** voulant intégrer IA intelligemment
- **UX Designers** comprenant contraintes techniques IA
- **Tech Leads** orchestrant équipes IA+Dev+Design

**Expansion Post-MVP:**

**M12+ Audience Élargie:**
- **PMs**: "Comprendre intersection IA/Architecture/UX pour mieux spec"
- **Designers UX**: "Coder ses propres prototypes IA"
- **CTOs**: "Décisions architecture pour produits IA-first"

**M18+ Expansion Technologique:**
- React/Next en gardant approche 3 piliers (IA + Architecture + UX)
- Maintien focus multi-expertise comme différenciateur

**Sujets par Pilier (Vision Complète):**

**IA:**
- RAG, agents autonomes, LLM tooling, prompting avancé
- Nuxt + LangChain, Vercel AI SDK
- Edge AI (Cloudflare Workers AI)

**Ingénierie Logicielle:**
- Architecture hexagonale, DDD, CQRS
- Clean Code, SOLID, Design Patterns
- Testing strategies, CI/CD
- Java/Spring en complément Nuxt (backend robuste)

**UX:**
- Design Systems (tokens, composants)
- Accessibilité (WCAG, ARIA)
- UX Writing, Micro-interactions
- IA dans workflows UX (Figma plugins, prototyping IA)

**Articles Cross-Piliers (Valeur Unique):**
- "Architecture RAG Production-Ready" (IA + Ingénierie)
- "Design System pour Apps IA" (UX + IA)
- "Clean Code pour Composants UX" (Ingénierie + UX)
- "Full Stack Holistique: IA + Architecture + UX" (3 piliers intégrés)

**Vision Ultime:**
sebc.dev devient la référence francophone pour développeurs cherchant l'excellence à l'intersection de l'IA, de l'ingénierie logicielle et de l'UX - un positionnement unique dans un marché saturé de spécialistes mono-domaine.
