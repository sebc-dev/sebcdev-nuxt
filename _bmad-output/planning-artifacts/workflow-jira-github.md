# Workflow Solo Developer avec Jira, Claude Code et CodeRabbit

## Introduction

Ce document décrit un workflow pragmatique pour développeur solo qui souhaite :

- Garder une trace structurée de son travail
- Faciliter les reviews de code généré par IA
- Maximiser l'efficacité de Claude Code et CodeRabbit
- Éviter l'over-engineering et la bureaucratie inutile

### Principes fondamentaux

1. **Juste assez de structure** : chaque élément du workflow doit apporter une valeur concrète
2. **PRs reviewables** : le code doit pouvoir être relu efficacement par un humain
3. **Contexte partagé** : les outils (Claude Code, CodeRabbit) doivent comprendre ce qu'on fait
4. **Flux continu** : pas de sprints rigides, un Kanban simple suffit

---

## Structure Jira

### Hiérarchie recommandée

| Niveau | Description | Exemple | Durée typique |
|--------|-------------|---------|---------------|
| **Epic** | Objectif business ou fonctionnalité majeure | "Blog bilingue", "Système d'authentification" | Plusieurs semaines |
| **Story** | Unité de valeur livrable | "Recherche full-text", "Page de profil utilisateur" | 1-5 jours |
| **Task** | ⚠️ Optionnel - sous-étape technique | À éviter pour commencer | - |

### Ce qu'on n'utilise PAS

- **Sub-tasks** : trop granulaire pour un solo
- **Story Points** : sans équipe, la vélocité n'a pas de sens
- **Sprints** : préférer un flux Kanban continu
- **Workflows complexes** avec approbations multiples

### Configuration du board Kanban

```
┌─────────────┬─────────────┬─────────────┬─────────────┐
│   À faire   │  En cours   │  En review  │    Done     │
├─────────────┼─────────────┼─────────────┼─────────────┤
│  Stories    │  1 story    │  PRs en     │  Stories    │
│  priorisées │  max        │  attente    │  terminées  │
│             │             │  de merge   │             │
└─────────────┴─────────────┴─────────────┴─────────────┘
```

#### Règles du board

- **WIP Limit** : 1 seule Story "En cours" à la fois
- **En review** : contient les PRs ouvertes, pas les Stories
- Une Story passe en "Done" quand toutes ses PRs sont mergées

---

## Rédaction des Stories

### Template de Story

```markdown
## [PROJET-XX] Titre descriptif de la fonctionnalité

### Description
[1-2 phrases expliquant la valeur apportée à l'utilisateur ou au système]

### Critères d'acceptance
- [ ] Critère 1 : comportement attendu mesurable
- [ ] Critère 2 : comportement attendu mesurable
- [ ] Critère 3 : comportement attendu mesurable

### Contexte technique
- **Stack** : [technologies concernées]
- **Dépendances** : [autres stories, APIs externes, etc.]
- **Documentation** : [liens vers docs pertinentes]

### Notes de découpage
[À remplir avant de commencer - voir section "Découpage en PRs"]

### Suivi des PRs
- [ ] PR #XX - Description (statut)
- [ ] PR #XX - Description (statut)
```

### Bonnes pratiques

1. **Critères d'acceptance testables** : éviter "l'UI doit être belle", préférer "le formulaire affiche un message d'erreur si l'email est invalide"

2. **Contexte suffisant** : quelqu'un qui lit la Story dans 6 mois (toi inclus) doit comprendre le pourquoi

3. **Pas de solution technique dans le titre** : "Recherche full-text" plutôt que "Implémenter MiniSearch"

---

## Découpage en Pull Requests

### Pourquoi découper ?

Une Story peut représenter plusieurs centaines de lignes de code. Au-delà de ~400 lignes modifiées, la qualité de review chute drastiquement :

- Les erreurs passent inaperçues
- La fatigue cognitive augmente
- Les commentaires deviennent superficiels

### Métriques cibles par PR

| Métrique | Cible | Maximum |
|----------|-------|---------|
| Lignes modifiées | 200-300 | 400 |
| Fichiers touchés | 5-8 | 15 |
| Temps de review | 15-20 min | 30 min |
| Objectifs | 1 seul | 1 seul |

### Stratégie de découpage

#### Approche recommandée : PRs directes vers main

```
main
  ├── PR #1: [SEBC-42] feat(search): infrastructure (1/4)
  ├── PR #2: [SEBC-42] feat(search): indexation logic (2/4)
  ├── PR #3: [SEBC-42] feat(search): UI component (3/4)
  └── PR #4: [SEBC-42] feat(search): integration (4/4)
```

**Avantages** :
- Chaque PR est autonome et reviewable
- Code "propre" mergé régulièrement
- Moins de conflits de merge
- CodeRabbit review chaque morceau avec contexte

**Règle d'or** : chaque PR doit être mergeable sans casser le build, même si la fonctionnalité n'est pas encore visible/activée.

#### Techniques pour garder les PRs indépendantes

1. **Feature flags** : la fonctionnalité existe mais est désactivée
2. **Routes non linkées** : la page existe mais n'est pas dans la navigation
3. **Code mort temporaire** : fonction exportée mais non appelée (à activer dans la PR finale)

### Plan de découpage type

Pour une fonctionnalité standard, le découpage suit souvent ce pattern :

```markdown
## PR 1: Infrastructure (~100-150 lignes)
- Dépendances (package.json)
- Types TypeScript
- Configuration de base
- Pas de logique métier

## PR 2: Core Logic (~200-250 lignes)
- Logique métier principale
- Services / composables
- Tests unitaires associés

## PR 3: UI / Présentation (~200-300 lignes)
- Composants d'interface
- Styles
- Accessibilité
- Tests de composants

## PR 4: Intégration (~100-150 lignes)
- Connexion des éléments
- Activation de la fonctionnalité
- Tests E2E
- Documentation
```

---

## Workflow avec Claude Code

### Démarrage de session

Au début de chaque session de travail, fournir le contexte complet :

```markdown
## Contexte de travail

**Story** : [PROJET-XX] Titre de la story
**Objectif** : [description courte]

**Critères d'acceptance** :
- [ ] Critère 1
- [ ] Critère 2

**Stack** : [technologies]

**Approche** : Je veux découper en PRs de ~200-300 lignes max.
Propose-moi un plan de découpage en 3-5 PRs avant de coder.

Chaque PR doit :
- Être mergeable indépendamment
- Ne pas casser le build
- Avoir un périmètre clair
- Pouvoir être reviewée en 15-20 min
```

### Pendant le développement

#### Points de synchronisation

Après chaque bloc logique de travail, demander :

```
Stop. Récapitule ce qu'on a fait.
- Quels fichiers ont été modifiés/créés ?
- Combien de lignes approximativement ?
- Est-ce un bon point pour faire une PR ?
- Que reste-t-il pour la Story complète ?
```

#### Avant de commit

```
Prépare-moi un résumé pour le message de commit.
Format : type(scope): description

Rappel : cette PR est liée à [PROJET-XX]
```

### Bonnes pratiques Claude Code

1. **Un objectif par session** : ne pas mélanger plusieurs PRs dans une session
2. **Valider le plan avant de coder** : demander le découpage AVANT d'écrire du code
3. **Reviews intermédiaires** : demander à Claude de relire son propre code avant de finaliser
4. **Tests au fil de l'eau** : demander les tests en même temps que le code, pas après

---

## Intégration CodeRabbit

### Configuration `.coderabbit.yaml`

```yaml
language: fr  # Reviews en français

reviews:
  request_changes_workflow: true
  high_level_summary: true
  poem: false  # Désactiver les poèmes (gadget)
  review_status: true
  
  path_instructions:
    - path: "**/*.vue"
      instructions: "Vérifier l'accessibilité (aria-labels, rôles)"
    - path: "**/*.ts"
      instructions: "Vérifier le typage strict, éviter les 'any'"

chat:
  auto_reply: true

integrations:
  jira:
    enabled: true
    # CodeRabbit va automatiquement:
    # - Lire la description de la Story liée
    # - Vérifier la cohérence avec les critères d'acceptance
    # - Contextualiser ses reviews
```

### Ce que CodeRabbit apporte

1. **Lien automatique** avec la Story Jira via le numéro de ticket
2. **Vérification des critères** : compare la PR aux acceptance criteria
3. **Reviews contextualisées** : comprend l'objectif global de la Story
4. **Historique** : sait quelles autres PRs sont liées à la même Story

### Optimiser les reviews CodeRabbit

Pour que CodeRabbit soit efficace :

- **Référencer le ticket** dans le titre ou la description de PR
- **PRs focalisées** : un objectif = des commentaires pertinents
- **Description claire** : expliquer ce que fait la PR ET ce qu'elle ne fait pas

---

## Conventions

### Branches

```
feature/PROJET-XX-description-courte
fix/PROJET-XX-description-courte
refactor/PROJET-XX-description-courte
docs/PROJET-XX-description-courte
```

Exemples :
```
feature/SEBC-42-search-infrastructure
feature/SEBC-42-search-ui
fix/SEBC-51-mobile-menu-overflow
```

### Commits (Conventional Commits)

```
type(scope): description courte

[corps optionnel]

Refs: PROJET-XX
```

#### Types autorisés

| Type | Usage |
|------|-------|
| `feat` | Nouvelle fonctionnalité |
| `fix` | Correction de bug |
| `refactor` | Refactoring sans changement fonctionnel |
| `style` | Formatage, espaces, etc. |
| `docs` | Documentation uniquement |
| `test` | Ajout/modification de tests |
| `chore` | Maintenance, dépendances |
| `perf` | Amélioration de performance |

#### Exemples

```bash
feat(search): add MiniSearch dependency and types

Refs: SEBC-42
```

```bash
fix(auth): prevent session timeout on mobile

Le token était rafraîchi uniquement sur les événements mouse,
pas sur les événements touch.

Refs: SEBC-51
```

### Pull Requests

#### Nommage

```
[PROJET-XX] type(scope): description (N/M)
```

- `N/M` indique la progression (PR 2 sur 4)
- Exemple : `[SEBC-42] feat(search): implement UI component (3/4)`

#### Template de description

```markdown
## Contexte

**Story** : [PROJET-XX] - Titre de la story
**PR** : X/Y - Description de cette PR

## Changements

[Liste concise des modifications principales]

- Ajout de X
- Modification de Y
- Configuration de Z

## Ce que cette PR ne fait PAS

[Important pour clarifier le scope]

- Pas d'intégration dans le layout (PR suivante)
- Pas de tests E2E (PR finale)

## Comment tester

1. Étape 1
2. Étape 2
3. Résultat attendu

## Checklist

- [ ] Le build passe
- [ ] Les tests passent
- [ ] < 400 lignes modifiées
- [ ] Un seul objectif
- [ ] Documentation mise à jour si nécessaire

## Screenshots

[Si changements UI, sinon supprimer cette section]
```

---

## Checklists

### Avant de commencer une Story

- [ ] Story rédigée avec critères d'acceptance clairs
- [ ] Contexte technique documenté
- [ ] Plan de découpage en PRs établi
- [ ] Estimation du nombre de PRs (3-5 typiquement)

### Avant chaque PR

- [ ] Branche créée avec bonne convention de nommage
- [ ] Objectif unique et clair
- [ ] Code fonctionnel (build passe)
- [ ] Tests ajoutés pour le nouveau code
- [ ] < 400 lignes modifiées

### Avant de merger

- [ ] CodeRabbit a reviewé
- [ ] Commentaires CodeRabbit traités
- [ ] Self-review effectuée
- [ ] Description de PR complète
- [ ] Référence au ticket Jira présente

### Avant de clôturer une Story

- [ ] Toutes les PRs mergées
- [ ] Tous les critères d'acceptance validés
- [ ] Documentation mise à jour si nécessaire
- [ ] Story déplacée en "Done"

---

## Exemples concrets

### Exemple : Story "Recherche full-text"

#### Story Jira

```markdown
## [SEBC-42] Recherche full-text dans le blog

### Description
Permettre aux visiteurs de rechercher des articles par mots-clés
dans le titre et le contenu.

### Critères d'acceptance
- [ ] Barre de recherche accessible depuis toutes les pages
- [ ] Recherche dans le titre ET le contenu des articles
- [ ] Résultats affichés avec highlight des termes recherchés
- [ ] Fonctionne en mode statique (SSG, pas de serveur)
- [ ] Accessible au clavier (navigation, fermeture avec Escape)

### Contexte technique
- **Stack** : Nuxt 4.2.x, Vue 3, TypeScript
- **Solution retenue** : MiniSearch (léger, client-side)
- **Contrainte** : Index généré au build, pas de serveur

### Suivi des PRs
- [x] PR #12 - Infrastructure (merged)
- [x] PR #13 - Indexation build-time (merged)
- [ ] PR #14 - Composant UI (in review)
- [ ] PR #15 - Intégration finale (à faire)
```

#### Découpage des PRs

**PR #12 - Infrastructure (127 lignes)**
```
feat(search): add MiniSearch infrastructure

- Ajout de minisearch comme dépendance
- Types TypeScript pour l'index de recherche
- Configuration de base dans nuxt.config.ts

Refs: SEBC-42
```

Fichiers modifiés :
- `package.json` (+2 lignes)
- `pnpm-lock.yaml` (auto)
- `types/search.ts` (+45 lignes)
- `nuxt.config.ts` (+12 lignes)

**PR #13 - Indexation (203 lignes)**
```
feat(search): implement build-time indexing

- Script de génération de l'index MiniSearch
- Hook Nuxt nitro:build:public-assets
- Tests unitaires de l'indexation

Refs: SEBC-42
```

**PR #14 - UI (276 lignes)**
```
feat(search): add SearchBar component

- Composant SearchBar.vue avec Combobox Radix
- Logique de recherche avec debounce
- Highlight des termes dans les résultats
- Support clavier complet

Refs: SEBC-42
```

**PR #15 - Intégration (89 lignes)**
```
feat(search): integrate search in layout

- Ajout de SearchBar dans le header
- Test E2E de la recherche
- Mise à jour de la documentation

Refs: SEBC-42
```

---

## Anti-patterns à éviter

### ❌ Story trop vague

```markdown
## Améliorer le blog
Rendre le blog mieux.
```

### ✅ Story actionnable

```markdown
## [SEBC-42] Recherche full-text dans le blog
Permettre aux visiteurs de rechercher des articles...
[critères mesurables]
```

### ❌ PR mammouth

```
feat: implement entire search feature

- Added MiniSearch
- Created indexing
- Built UI
- Integrated everything
- Added tests

+847 lines
```

### ✅ PRs atomiques

```
PR 1: feat(search): infrastructure (+127 lines)
PR 2: feat(search): indexing (+203 lines)
PR 3: feat(search): UI component (+276 lines)
PR 4: feat(search): integration (+89 lines)
```

### ❌ Commit message inutile

```
fix stuff
wip
updates
```

### ✅ Commit message informatif

```
fix(search): prevent empty query submission

L'utilisateur pouvait soumettre une recherche vide en appuyant
sur Entrée, ce qui affichait tous les résultats.

Refs: SEBC-42
```

---

## Ressources

### Outils

- [Conventional Commits](https://www.conventionalcommits.org/)
- [CodeRabbit Documentation](https://docs.coderabbit.ai/)
- [Jira Automation](https://www.atlassian.com/software/jira/guides/automation)

### Templates à créer

1. `.gitlab/merge_request_templates/default.md` ou `.github/PULL_REQUEST_TEMPLATE.md`
2. `.coderabbit.yaml`
3. `CLAUDE.md` avec section workflow

---

## Changelog du workflow

| Date | Modification |
|------|--------------|
| [Date de création] | Version initiale du workflow |