# 2. Personas Détaillés

## 2.1 Lucas — "Le Mid-Level Bloqué" (Primary Persona)

### Fiche d'Identité

| Attribut | Valeur |
|----------|--------|
| **Âge** | 27 ans |
| **Poste** | Développeur Fullstack (Vue.js/Nuxt & Node.js) |
| **Entreprise** | EcoLogistics, startup SaaS B2B (Série A) |
| **Équipe** | 8 personnes tech |
| **Expérience** | 4 ans (bootcamp + alternance → Mid-Level depuis 1 an) |
| **Autonomie** | 80% des tâches, manque vision architecturale macro |

### Journée Type

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

### Frictions Quotidiennes

| Friction | Impact | Workaround Actuel |
|----------|--------|-------------------|
| 70% du temps = "glue code" | Productivité divisée par 3 | Regex fragiles, `any` TypeScript |
| Réponses LLM mal formatées | Crashes UI imprévisibles | Console.log partout |
| Manque vision architecturale | Bloqué sur évolution Senior | Imite sans comprendre |
| Legacy vs Moderne | Tension constante | Évite sujets architecture |

### Vision de Succès

**Le Déclic**: "C'est exactement ce dont j'avais besoin : pas de théorie fumeuse sur le futur de l'IA, mais un bout de code Nuxt utilisable maintenant."

**Mesures Concrètes**:
- Tâche de 4h pliée en 1h
- Application ne crashe plus quand IA répond bizarrement
- Comprend le "pourquoi" et peut l'expliquer en Code Review
- Progression visible vers poste Senior

### Triggers de Recherche

- Erreurs TypeScript avec LLM
- Intégration IA sur legacy
- Patterns streaming Nuxt
- "Best practices" production IA

---

## 2.2 Chloé — "L'Apprentie Copilote" (Junior Developer)

### Fiche d'Identité

| Attribut | Valeur |
|----------|--------|
| **Âge** | 24 ans |
| **Formation** | Reconversion, Bootcamp 2024 + auto-formation |
| **Expérience** | 10 mois de code effectif |
| **Poste** | Développeuse Junior (Alternance/Premier CDD) |
| **Entreprise** | Agence Web Digitale |
| **Contexte** | Rythme soutenu, tuteur très occupé |

### Journée Type

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

### Frictions Quotidiennes

| Friction | Impact | Workaround Actuel |
|----------|--------|-------------------|
| IA donne solution finale | Saute étape d'apprentissage | Aucun (subit) |
| Code semble Senior mais fragile | Châteaux de cartes | Évite modifications |
| Mur du débogage | Blocage total sans tuteur | Re-prompt IA en boucle |
| Syndrome imposteur | Anxiété Code Reviews | Minimise ses contributions |

### Vision de Succès

**Le Déclic**: "Ah! Donc l'IA utilisait shallowRef ici pour optimiser perf, pas juste par hasard! Je comprends enfin la différence."

**Mesures Concrètes**:
- Refactorise code IA sans redemander
- Explique le "pourquoi" lors stand-up
- Corrige bug avant que senior ne le voie
- Passe de "Passagère" à "Pilote"

### Triggers de Recherche

- "Comprendre X en profondeur"
- "Pourquoi ça marche"
- Concepts fondamentaux (closures, reactivity, lifecycle)
- Tutoriels pas-à-pas bienveillants

---

## 2.3 Maxime — "L'Architecte Frankenstein" (Indie Hacker/Freelance)

### Fiche d'Identité

| Attribut | Valeur |
|----------|--------|
| **Âge** | 32 ans |
| **Background** | Ancien Senior Dev agence (tout quitté il y a 2 ans) |
| **Statut** | Solopreneur / Freelance mi-temps |
| **Projets** | 3 micro-SaaS (1 à 2k€ MRR, 1 zombie, 1 en lancement) |
| **Revenus** | 2 jours freelance/semaine pour sécurité financière |
| **Stack** | 15+ services (chaos Frankenstein) |

### Journée Type

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

### Stack Actuelle (Chaos)

| Catégorie | Services | Coût Mensuel |
|-----------|----------|--------------|
| Backend | Supabase, Firebase, Node.js/Railway | ~80€ |
| IA | OpenAI, Pinecone, LangChain | ~100€ |
| Ops | Vercel, Resend, Stripe, Sentry, Axiom, Crisp | ~200€ |
| **Total** | **15+ services** | **~400€** |

### Frictions Quotidiennes

| Friction | Impact | Workaround Actuel |
|----------|--------|-------------------|
| Integration Hell | 70% temps sur "glu" | Webhooks fragiles |
| 15 onglets monitoring | Zero vue d'ensemble | Emails d'alerte (spam) |
| Notification fatigue | Ignore alertes critiques | Espère que ça marche |
| Stack redondante | 400€/mois gaspillés | Accepte le coût |

### Vision de Succès

**Le Déclic**: "Ce gars me fait gagner de l'argent."

**Mesures Concrètes**:
- Résilie 3 abonnements SaaS inutiles
- Dashboard "Maison" montrant état santé global (Vert/Rouge)
- Solution DIY implémentée en une après-midi
- Sérénité retrouvée

### Triggers de Recherche

- "Simplifier stack"
- "DIY alternative to [SaaS]"
- "Pattern monolith modulaire"
- "Économiser" + techno

---

## 2.4 Validation Personas

### Source Actuelle des Personas

| Persona | Source | Niveau de Confiance |
|---------|--------|---------------------|
| Lucas | Auto-biographie (moi il y a 3 ans) + observations collègues | ⭐⭐⭐ Moyen |
| Chloé | Observations juniors en mission + forums/Discord | ⭐⭐ Faible |
| Maxime | Projection future + suivi créateurs indie (Twitter/IH) | ⭐⭐ Faible |

**Verdict** : Personas basés sur intuition + observation indirecte. Pas d'interviews formelles. **Risque de biais de confirmation.**

### Plan de Validation M1-M3

| Action | Timing | Objectif | Critère Succès |
|--------|--------|----------|----------------|
| **5 interviews utilisateurs** | M1-M2 | Valider frictions | 3/5 confirment frictions identifiées |
| **Sondage Discord Vue.js FR** | M1 | Quantifier besoins | 30+ réponses, patterns clairs |
| **Analyse commentaires** | M1-M3 | Feedback organique | Commentaires mentionnent frictions prédites |
| **Heatmaps premiers articles** | M2-M3 | Comportement réel | Scroll/Copy patterns alignés personas |

### Questions Interviews (5 entretiens 30min)

**Cibles** : 2 mid-level, 2 juniors, 1 indie hacker (recrutement via LinkedIn/Discord)

1. "Quand tu bloques sur un problème technique, que fais-tu en premier?"
2. "Décris ta dernière galère avec l'IA/LLM dans ton code."
3. "C'est quoi un article technique que tu as trouvé vraiment utile récemment? Pourquoi?"
4. "Comment tu te formes actuellement? Qu'est-ce qui te frustre?"
5. "Si je te disais 'IA + Architecture + UX', ça t'évoque quoi?"

### Pivot Personas si Validation Échoue

Si les interviews révèlent des frictions totalement différentes :
- M3 : Reformuler personas
- M4 : Ajuster calendrier éditorial
- Documenter publiquement le pivot ("Learning in Public")

---
