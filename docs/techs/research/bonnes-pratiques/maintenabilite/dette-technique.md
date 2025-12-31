# Gestion de la dette technique pour Nuxt 4 : guide complet 2025

La maintenabilité d'un projet Nuxt 4/Vue 3 repose sur trois piliers fondamentaux : une documentation structurée de la dette technique via un registre TECH_DEBT.md, des Architecture Decision Records (ADR) au format MADR, et un workflow d'intégration continue des revues de dette dans les sprints. L'allocation de **15-20% de chaque sprint** à la réduction de dette, combinée à des quality gates automatisés, représente le standard industriel actuel recommandé par Scrum.org et confirmé par le rapport Oliver Wyman 2024.

Ce guide synthétise les meilleures pratiques pour votre stack technique spécifique : Nuxt 4.2.x avec structure `app/`, SSG sur Cloudflare Pages, TailwindCSS 4.1.x, shadcn-vue et Nuxt Content 3.10.0+.

---

## Le registre TECH_DEBT.md structure votre dette efficacement

Le fichier TECH_DEBT.md constitue la source canonique des améliorations techniques nécessaires, distinct du backlog de fonctionnalités. Placé à la racine du projet et référencé depuis le README, il documente chaque élément de dette selon un format Technical Debt Record (TDR), inspiré des ADR.

### Structure recommandée pour chaque entrée

Chaque item de dette suit ce template minimal :

```markdown
## TD-001: Mise en cache API non implémenté

| Attribut | Valeur |
|----------|--------|
| **Status** | 🟠 Open |
| **Priorité** | High |
| **Type** | Performance |
| **Effort** | Medium (2-3 jours) |
| **Owner** | @backend-team |
| **Issue** | [#234](lien) |

**Description:**  
Les appels API vers Nuxt Content ne sont pas mis en cache côté serveur,
provoquant des rebuilds inutiles en SSG.

**Impact:**  
- Build time multiplié par 3 sur Cloudflare Pages
- Quota de build mensuel consommé rapidement

**Solution proposée:**  
Implémenter le caching avec `cachedEventHandler` de Nitro.
```

Le tableau récapitulatif en en-tête permet un suivi rapide :

```markdown
| Priorité | Nombre | Effort total estimé |
|----------|--------|---------------------|
| 🔴 Critical | 1 | 3 jours |
| 🟠 High | 3 | 8 jours |
| 🟡 Medium | 5 | 10 jours |
```

### Systèmes de priorisation éprouvés

La **matrice Impact/Effort** divise les actions en quatre quadrants : les Quick Wins (impact élevé, effort faible) sont traités immédiatement, les Big Bets (impact élevé, effort élevé) sont planifiés dans les sprints, les Fill-ins peuvent attendre, et les Money Pits (faible impact, effort élevé) sont différés ou abandonnés.

Pour un scoring plus fin, la formule `Priority Score = (Impact × Severity) / Effort` permet de classer objectivement les items. Le Technical Debt Ratio cible est **inférieur à 5%** pour un projet sain selon les benchmarks OpsLevel et SonarQube.

### Intégration GitHub native

Créez un template d'issue `.github/ISSUE_TEMPLATE/tech-debt.yml` pour standardiser les remontées de dette. Les labels recommandés incluent `tech-debt`, `tech-debt/code`, `tech-debt/performance` avec un code couleur cohérent. Chaque entrée TECH_DEBT.md référence son issue GitHub correspondante, créant un lien bidirectionnel qui facilite le tracking.

---

## Les ADR au format MADR 4.0 documentent vos décisions architecturales

Les Architecture Decision Records capturent le contexte, les alternatives considérées et les raisons de chaque décision technique significative. Le format **MADR 4.0.0** (Markdown Architectural Decision Records), publié en septembre 2024, est devenu le standard de facto.

### Template MADR adapté aux projets Vue/Nuxt

```markdown
---
status: accepted
date: 2025-01-15
decision-makers: [Lead Dev, Tech Lead]
---

# Utilisation de Pinia pour la gestion d'état

## Context and Problem Statement
L'application nécessite une gestion d'état globale pour les données utilisateur
et les préférences. Quelle solution adopter dans l'écosystème Nuxt 4 ?

## Decision Drivers
* Support TypeScript natif requis
* Compatibilité SSG Cloudflare Pages
* DevTools intégrés pour debugging

## Considered Options
* Vuex 4
* Pinia
* Composables avec provide/inject

## Decision Outcome
Chosen option: "Pinia", car recommandé officiellement pour Vue 3/Nuxt 4,
avec une API plus simple et un support SSR/SSG natif.

### Consequences
* Good: Intégration seamless avec Nuxt DevTools
* Good: Définition des stores TypeScript-first
* Bad: Migration nécessaire des projets existants Vuex
```

### Organisation des ADR dans le projet

La structure recommandée place les ADR dans `docs/decisions/` :

```
my-nuxt-app/
├── docs/
│   └── decisions/
│       ├── 0000-use-madr-format.md
│       ├── 0001-choose-nuxt-4-ssg.md
│       ├── 0002-use-pinia-state.md
│       ├── 0003-cloudflare-pages-hosting.md
│       └── template.md
├── app/
├── nuxt.config.ts
└── TECH_DEBT.md
```

L'outil **Log4brains** génère automatiquement un site de documentation navigable depuis vos ADR, déployable sur Cloudflare Pages. L'initialisation se fait via `npx log4brains init`, puis `log4brains build` pour la génération statique compatible avec votre hébergement gratuit.

### Connexion ADR et dette technique

Les ADR peuvent explicitement documenter la dette introduite par une décision. Une section "Technical Debt Note" ajoutée au template capture les compromis acceptés avec leur plan de résolution. Le statut "Superseded by ADR-XXX" indique qu'une nouvelle décision a remplacé (et potentiellement résolu) une dette précédente.

---

## Le processus de révision combine fréquences multiples

L'approche hybride combinant détection continue, allocation par sprint et revues périodiques s'impose comme la meilleure pratique. Selon une étude Stripe, les développeurs consacrent en moyenne **13,5 heures hebdomadaires** à gérer la dette technique — un chiffre que ce processus vise à réduire significativement.

### Fréquences recommandées par activité

La **détection continue** via quality gates ESLint et tests automatisés dans la CI prévient l'accumulation de nouvelle dette. L'**allocation sprint** de 15-20% de capacité assure un remboursement régulier. Les **revues trimestrielles** avec stakeholders permettent la priorisation stratégique et l'identification de dette architecturale cachée.

Pour un projet Nuxt déployé sur Cloudflare Pages, intégrez ces vérifications dans votre workflow GitHub Actions :

```yaml
name: Quality Gate
on: [push, pull_request]
jobs:
  lint-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm install
      - run: pnpm lint
      - run: pnpm test --coverage
      - name: Check coverage threshold
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage below 80%"
            exit 1
          fi
```

### Métriques clés à suivre

Le **Technical Debt Ratio** (coût de remédiation / coût de développement × 100) reste la métrique principale, avec un objectif sous 5%. La **complexité cyclomatique** par fonction doit rester inférieure à 10. Le **code coverage** cible 80% minimum, particulièrement critique pour les composables et utils Nuxt. Le **cycle time** (premier commit au déploiement) révèle l'impact de la dette sur la vélocité.

### Intégration Scrum recommandée

Durant le **Sprint Planning**, présentez les items de dette prioritaires comme des PBIs avec story points. Les **Daily Scrums** reportent les impediments liés à la dette et coordonnent le pair programming sur items complexes. La **Sprint Review** démontre les améliorations techniques (gains de performance, réduction du build time). La **Retrospective** analyse la dette créée pendant le sprint et identifie les patterns causant l'accumulation.

La **Definition of Done** renforcée inclut : complexité cyclomatique sous seuil, coverage ≥ 80%, aucune violation ESLint critique, revue de code effectuée, documentation mise à jour.

---

## La documentation maintenabilité suit des conventions établies

### Structure docs/ optimale pour Nuxt 4

```
project-root/
├── docs/
│   ├── decisions/              # ADRs
│   ├── content/                # Si utilisant Nuxt Content pour la doc
│   │   ├── 1.getting-started/
│   │   ├── 2.guide/
│   │   └── 3.api/
│   └── architecture/           # Diagrammes C4, schemas
├── README.md
├── CONTRIBUTING.md
├── CHANGELOG.md
├── TECH_DEBT.md
└── LICENSE
```

### Le CHANGELOG automatisé avec Release Please

Le format **Keep-a-Changelog** structure les modifications par type (Added, Changed, Fixed, Security) avec dates ISO 8601. L'automatisation via **Release Please** de Google parse vos commits Conventional Commits et génère le CHANGELOG automatiquement :

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        with:
          release-type: node
```

La configuration `release-please-config.json` personnalise les sections :

```json
{
  "packages": {
    ".": {
      "changelog-sections": [
        { "type": "feat", "section": "✨ Features" },
        { "type": "fix", "section": "🐛 Bug Fixes" },
        { "type": "perf", "section": "⚡ Performance" }
      ]
    }
  }
}
```

### CONTRIBUTING.md actionnable

Le guide de contribution inclut : configuration de l'environnement, standards de code ESLint/Prettier, convention Conventional Commits avec types autorisés (`feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`), et process de Pull Request avec checklist de documentation.

Commitlint + Husky enforrent les conventions :

```javascript
// commitlint.config.js
export default {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', 
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'chore']
    ]
  }
}
```

---

## La configuration tooling Nuxt 4 moderne utilise ESLint flat config

### ESLint avec @nuxt/eslint

Le module officiel génère une configuration ESLint 9 flat config optimisée :

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/eslint'],
  eslint: {
    config: {
      stylistic: {
        indent: 2,
        quotes: 'single',
        semi: false
      }
    }
  }
})
```

Le fichier `eslint.config.mjs` généré s'étend facilement :

```javascript
import withNuxt from './.nuxt/eslint.config.mjs'

export default withNuxt({
  rules: {
    'vue/multi-word-component-names': 'off', // Pour shadcn-vue
    'no-console': ['warn', { allow: ['warn', 'error'] }]
  }
})
```

### Prettier avec TailwindCSS 4

La configuration Prettier pour TailwindCSS v4 utilise `tailwindStylesheet` au lieu de `tailwindConfig` :

```json
{
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindStylesheet": "./app/assets/css/tailwind.css",
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2
}
```

### Structure app/ et conventions Vue 3

Nuxt 4 adopte la structure `app/` directory :

```
app/
├── components/
│   ├── ui/                 # shadcn-vue
│   │   └── button/
│   └── features/           # Par domaine métier
├── composables/            # useAuth.ts, useFetch.ts
├── layouts/
├── pages/
└── assets/css/tailwind.css
```

Les composants Vue 3 utilisent `<script setup>` avec les macros modernes :

```vue
<script setup lang="ts">
// Vue 3.5+ : Reactive Props Destructure
const { title, count = 0 } = defineProps<{
  title: string
  count?: number
}>()

// defineModel pour v-model bidirectionnel
const modelValue = defineModel<string>()

// Emits typés
const emit = defineEmits<{
  submit: [data: FormData]
}>()
</script>
```

### Vitest pour les tests Nuxt 4

```typescript
// vitest.config.ts
import { defineVitestConfig } from '@nuxt/test-utils/config'

export default defineVitestConfig({
  test: {
    environment: 'nuxt',
    coverage: {
      provider: 'v8',
      include: ['app/**/*.{ts,vue}'],
      exclude: ['app/components/ui/**'], // Exclure shadcn-vue
      thresholds: { lines: 80, branches: 80, functions: 80 }
    }
  }
})
```

Les tests utilisent `mountSuspended` pour les composants Nuxt :

```typescript
import { mountSuspended } from '@nuxt/test-utils/runtime'

it('renders component', async () => {
  const wrapper = await mountSuspended(MyComponent)
  expect(wrapper.text()).toContain('Expected text')
})
```

---

## Conclusion : un système intégré pour la maintenabilité long-terme

La gestion efficace de la dette technique dans un projet Nuxt 4/Vue 3 ne repose pas sur un outil unique mais sur l'intégration cohérente de plusieurs pratiques. Le **TECH_DEBT.md** centralise la visibilité, les **ADR MADR** documentent le contexte décisionnel, et le **workflow de révision continue** (15-20% par sprint + quality gates automatisés) assure le remboursement régulier.

Pour votre contexte SSG Cloudflare Pages avec budget zéro, privilégiez : ESLint via `@nuxt/eslint` (gratuit, intégré), Release Please pour l'automatisation CHANGELOG (GitHub Actions gratuit), et Log4brains déployé sur votre instance Cloudflare Pages pour la documentation ADR. Le coverage Vitest avec seuils à 80% constitue votre principal indicateur de maintenabilité mesurable automatiquement.

L'investissement initial dans cette infrastructure documentaire se rentabilise rapidement : réduction du temps d'onboarding des nouveaux contributeurs, décisions architecturales traçables, et dette technique visible plutôt que cachée dans le code.