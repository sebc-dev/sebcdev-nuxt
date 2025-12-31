# Git et branching pour développeur solo : le guide Nuxt 4 + Cloudflare Pages

**GitHub Flow avec feature branches courtes** est la stratégie optimale pour un développeur solo sur un projet Nuxt 4 SSG déployé sur Cloudflare Pages. Cette approche combine simplicité, PRs pour self-review du code IA, et intégration native avec les preview deployments — tout en permettant de limiter les builds via les branch controls et la directive `[Skip CI]`.

Le consensus 2024-2025 est clair : GitFlow est considéré comme legacy pour les projets web en continu, tandis que GitHub Flow et Trunk-Based Development (avec short-lived branches) dominent l'écosystème Jamstack. Vincent Driessen lui-même, créateur de GitFlow, recommande désormais des workflows plus simples pour les équipes pratiquant la livraison continue.

---

## GitHub Flow s'impose comme standard pour les projets Jamstack solo

Pour un développeur solo utilisant Claude Code et cherchant un équilibre entre détection précoce des problèmes et limitation des déploiements, **GitHub Flow** offre le meilleur compromis. Le workflow se résume à une branche principale `main` toujours déployable, des feature branches courtes (`feat/`, `fix/`), et des Pull Requests systématiques même en solo.

La force de cette approche réside dans son intégration naturelle avec Cloudflare Pages. Chaque feature branch génère automatiquement un preview deployment avec une URL unique du type `feat-nom-feature.projet.pages.dev`. Cela permet de tester sur l'infrastructure réelle avant de merger, sans polluer la production. Les études montrent que **les équipes pratiquant ces workflows simples déploient jusqu'à 200x plus fréquemment** avec moins d'incidents.

Le Trunk-Based Development pur (commits directs sur main) perd l'avantage crucial des PRs pour la self-review — particulièrement important quand on travaille avec du code généré par IA. La variante TBD avec short-lived branches est en pratique **quasi-identique à GitHub Flow** et tout aussi recommandable.

**Configuration recommandée des branches** : production sur `main`, preview branches configurées en custom avec inclusion de `feat/*`, `fix/*`, `refactor/*`, et exclusion de `dependabot/*` et `renovate/*`. Cette granularité évite les builds inutiles tout en conservant les previews sur les branches de travail actives.

---

## Cloudflare Pages offre 500 builds mensuels avec des contrôles granulaires

Le free tier de Cloudflare Pages impose **500 builds par mois par compte** (pas par projet), avec un seul build concurrent. Les preview deployments sont illimités, la bande passante également — seuls les builds comptent. Pour un solo dev, ce quota est généralement suffisant, mais quelques stratégies permettent de l'optimiser significativement.

La directive **`[Skip CI]`** dans le message de commit désactive le build pour ce commit spécifique. Les variantes `[CI Skip]`, `[CI-Skip]`, `[Skip-CI]` et `[CF-Pages-Skip]` fonctionnent également (insensible à la casse). C'est la méthode la plus directe pour éviter un build sur un commit cosmétique ou de documentation.

Les **Build Watch Paths** permettent de déclencher les builds uniquement quand certains fichiers changent. Configuration typique pour un projet Nuxt : inclure `src/*`, `content/*`, `components/*`, `pages/*` et exclure `docs/*`, `README.md`, `.github/*`. Attention : si un push contient plus de 3000 fichiers ou 20+ commits, le path matching est bypassé automatiquement.

Le **Build Caching** (en beta) réduit les temps de build de 50%+ sur les builds répétés. Il supporte pnpm nativement et cache les dépendances pendant 7 jours. Pour Nuxt, le répertoire `.nuxt` n'est pas encore dans la liste officielle des frameworks avec incremental builds, mais le cache des dépendances accélère déjà considérablement le processus.

| Limite Free Tier | Valeur |
|------------------|--------|
| Builds/mois | **500** |
| Concurrent builds | 1 |
| Preview deployments | Illimité |
| Bande passante | Illimitée |
| Fichiers par site | 20 000 |
| Taille fichier max | 25 MiB |

**Astuce avancée** : les déploiements via `wrangler pages deploy` (Direct Upload) ne comptent pas dans le quota de builds. En cas de limite atteinte, utiliser GitHub Actions pour le build puis Direct Upload préserve la capacité de déploiement.

---

## Conventional Commits structure vos messages pour la traçabilité IA

La spécification Conventional Commits 1.0.0 impose un format strict : `<type>(<scope>): <description>`. Cette structure devient particulièrement précieuse quand on travaille avec du code généré par IA, car elle force une réflexion sur la nature de chaque changement et facilite la review différée.

Les **types principaux** pour un projet Nuxt sont `feat` (nouvelles fonctionnalités, bump MINOR), `fix` (corrections, bump PATCH), `refactor` (restructuration sans changement de comportement), `chore` (maintenance), `docs` (documentation), `style` (formatage), `test` (tests), `build` (dépendances/système de build), et `ci` (configuration CI/CD).

Pour les **scopes adaptés à Nuxt**, la convention recommande : `components`, `pages`, `composables`, `content`, `config`, `layouts`, `plugins`, `middleware`, `server`, `utils`, `assets`, `types`, `i18n`, `store`, `api`, `deps`. L'utilisation de scopes multiples est supportée avec des délimiteurs (`feat(pages,layouts): implement dark mode`).

La notation des **breaking changes** utilise soit le `!` après le type (`feat(api)!: change response format`), soit le footer `BREAKING CHANGE:` dans le corps du commit. Ces breaking changes déclenchent un bump MAJOR en versioning sémantique automatisé.

**Installation avec pnpm** : `pnpm add -D @commitlint/cli @commitlint/config-conventional husky`, puis `pnpm exec husky init` pour initialiser les hooks. Le hook `commit-msg` valide chaque commit avec `pnpm exec commitlint --edit $1`. La configuration `commitlint.config.js` étend `@commitlint/config-conventional` et peut définir des scopes spécifiques au projet.

---

## Squash merge et commits atomiques optimisent l'historique

Pour un développeur solo, le **Squash & Merge** est la stratégie de merge recommandée. Elle produit un historique linéaire et propre où chaque commit représente une feature complète, tout en préservant la granularité des commits individuels dans la PR pour la review. L'alternative Rebase & Merge conserve les commits atomiques mais crée des risques de conflits répétés.

Un **commit atomique** est la plus petite unité de changement significatif qui reste complète et fonctionnelle. Le code doit compiler et les tests passer après chaque commit. Atomique ne signifie pas petit — un commit peut toucher plusieurs fichiers s'ils sont liés à la même modification logique. La règle d'or : si vous ne pouvez pas décrire le commit en une phrase claire, il est probablement trop gros.

Pour le **staging partiel**, `git add -p` permet de sélectionner interactivement les "hunks" (blocs de code) à inclure. Les options `y` (stage), `n` (skip), `s` (split en hunks plus petits) et `e` (édition manuelle) offrent un contrôle fin. `git reset --soft HEAD~1` annule le dernier commit en conservant les changements stagés — utile pour réorganiser avant une PR.

L'organisation des commits pour une feature complexe suit une structure logique : d'abord les refactorings préparatoires (`refactor: extract validation utils`), puis l'implémentation par étapes (`feat: implement JWT service`, `feat: add login endpoint`), et enfin les tests et la documentation. Cette séparation facilite considérablement la review du code généré par IA.

---

## La review du code IA exige une vigilance particulière sur les dépendances

Le code généré par Claude Code ou d'autres LLMs présente des risques spécifiques que les commits atomiques et les PRs permettent de mitiger. Les statistiques 2024-2025 sont préoccupantes : **environ 20% des packages recommandés par les LLMs sont des hallucinations** — ils n'existent pas. Ce phénomène, nommé "slopsquatting", peut mener à des attaques supply chain si un attaquant crée un package malveillant avec le nom halluciné.

Les **red flags prioritaires** à surveiller dans le code IA incluent : packages inexistants (toujours vérifier sur npm), API keys hardcodées, fonctions ou méthodes appelées qui n'existent pas dans la librairie, absence de validation des inputs, et gestion d'erreurs manquante. Une étude récente indique que **42% des snippets IA contiennent des erreurs** de nature diverse.

La stratégie recommandée consiste à traiter l'IA comme un développeur junior talentueux mais inexpérimenté. Générer le code par petits incréments plutôt qu'en gros blocs, demander des explications sur les choix d'implémentation, et vérifier systématiquement chaque dépendance ajoutée. L'exécution de `npm audit` ou `pnpm audit` après chaque ajout de dépendance est non-négociable.

Pour la **traçabilité**, certains développeurs indiquent dans le commit message si le code provient de l'IA (`feat(components): add UserCard (AI-generated base)`, puis `refactor(components): optimize UserCard (manual review)`). Cette pratique facilite les audits de sécurité ultérieurs et la compréhension de l'historique.

---

## Le versioning sémantique s'adapte aux projets web avec bumpp

SemVer (MAJOR.MINOR.PATCH) est conçu pour des logiciels avec API publique, mais s'adapte aux sites web avec une réinterprétation : **MAJOR** pour les refontes majeures ou changements de structure d'URLs, **MINOR** pour les nouvelles fonctionnalités ou sections, **PATCH** pour les corrections de bugs et typos. Pour un blog personnel, cette granularité reste optionnelle mais apporte discipline et traçabilité.

L'outil **bumpp** (par Anthony Fu, créateur de Vitest) est le plus adapté pour un solo dev. Sans configuration, `npx bumpp` lance un prompt interactif pour choisir le type de bump, puis commit, tag et push automatiquement. Aucun CHANGELOG n'est généré — uniquement les git tags au format `v1.0.0`.

**release-it** constitue une alternative plus complète si vous souhaitez des GitHub Releases automatiques ou une intégration CI/CD. Sa configuration minimale via `.release-it.json` permet de désactiver la publication npm (`"npmPublish": false`) et de créer des releases GitHub avec le GITHUB_TOKEN. Standard-version, autrefois populaire, est **abandonné depuis 4 ans** et ne doit plus être utilisé.

L'intégration avec Cloudflare Pages est transparente : le workflow devient `git commit` → `npx bumpp minor` → Cloudflare Pages détecte le push et déploie. Les tags sont poussés automatiquement et peuvent déclencher des workflows GitHub Actions spécifiques si nécessaire.

---

## Configuration complète recommandée pour votre projet Nuxt 4

Le setup optimal combine toutes ces pratiques dans une configuration cohérente. Voici les fichiers de configuration essentiels pour démarrer :

**package.json (scripts et dépendances)** :
```json
{
  "scripts": {
    "prepare": "husky",
    "release": "bumpp",
    "release:patch": "bumpp patch",
    "release:minor": "bumpp minor"
  },
  "devDependencies": {
    "@commitlint/cli": "^19.0.0",
    "@commitlint/config-conventional": "^19.0.0",
    "husky": "^9.0.0",
    "bumpp": "^9.0.0"
  }
}
```

**commitlint.config.js** :
```javascript
export default {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'scope-enum': [1, 'always', [
      'components', 'pages', 'composables', 'content', 
      'config', 'layouts', 'plugins', 'middleware',
      'server', 'utils', 'types', 'deps', 'ci'
    ]]
  }
};
```

**Configuration Cloudflare Pages** : production branch `main`, preview branches custom avec `feat/*`, `fix/*`, `refactor/*` inclus et `dependabot/*`, `renovate/*` exclus, build cache activé.

## Conclusion

Pour un développeur solo sur Nuxt 4 avec Cloudflare Pages et assistance IA, le workflow optimal est **GitHub Flow + Conventional Commits + Squash Merge + bumpp**. Cette combinaison offre la simplicité d'une seule branche principale, la traçabilité des messages de commit structurés, la propreté d'un historique linéaire après merge, et l'automatisation minimale du versioning sans overhead.

L'insight clé est que les PRs, même en solo, deviennent essentielles quand on travaille avec du code IA — elles créent un checkpoint de review naturel et s'intègrent parfaitement avec les preview deployments Cloudflare. Le quota de 500 builds mensuels est rarement atteint avec une utilisation judicieuse de `[Skip CI]` et des branch filters, mais le Direct Upload reste disponible en secours.

La validation des dépendances hallucinées par l'IA représente le risque le plus sous-estimé — intégrer `pnpm audit` dans votre routine de review et vérifier l'existence de chaque nouveau package sur npm avant tout commit protège contre les attaques supply chain émergentes dans l'ère du code assisté par IA.