# Automatisation des changelogs et versioning Git pour Nuxt 4

**standard-version est officiellement déprécié depuis mai 2022** — pour un projet Nuxt 4/Vue 3 avec pnpm 10.26+ déployé en SSG sur Cloudflare Pages, les alternatives recommandées sont **bumpp** (simplicité maximale) ou **release-it** (fonctionnalités étendues). L'outil **commit-and-tag-version** offre une migration directe depuis standard-version, tandis que **changesets** convient mieux aux monorepos. Cette recherche couvre la configuration complète de l'écosystème : Conventional Commits, validation des commits, hooks Git, génération de CHANGELOG et workflows GitHub Actions — le tout compatible avec un budget de 0€.

---

## Standard-version et ses alternatives en décembre 2025

**standard-version** (v9.5.0, mai 2022) est officiellement déprécié et ne reçoit plus de mises à jour. Le README du dépôt GitHub contient un avertissement de dépréciation et recommande deux alternatives : **release-please** (outil Google pour GitHub Actions) ou **commit-and-tag-version** (fork maintenu activement).

L'analyse comparative des outils révèle des profils distincts adaptés à différents cas d'usage :

| Outil | Statut | Downloads/semaine | Idéal pour |
|-------|--------|-------------------|------------|
| **bumpp** v10.3.2 | ✅ Actif (antfu) | ~80K | Projets frontend simples |
| **release-it** v19.x | ✅ Actif | ~630K | Projets avec besoins de personnalisation |
| **commit-and-tag-version** v12.x | ✅ Actif | ~200K | Migration depuis standard-version |
| **changesets** v2.27.x | ✅ Actif | ~2M | Monorepos et packages npm |
| **semantic-release** v25.0.2 | ✅ Actif | ~2M | CI/CD entièrement automatisé |

Pour un projet SSG Nuxt 4 qui ne publie pas sur npm, **bumpp** est le choix optimal : maintenu par Anthony Fu (figure centrale de l'écosystème Vue), il offre une interface interactive, supporte les Conventional Commits, et fonctionne parfaitement avec pnpm. **release-it** convient si vous avez besoin de créer des GitHub Releases automatiquement ou d'une configuration plus avancée via son architecture de plugins.

**semantic-release** exige Node.js 22.14+ (attention avec Node 22 LTS standard) et impose une discipline stricte des commits conventionnels — son approche "zero human intervention" peut être contraignante pour des projets frontend où le jugement humain sur les versions est parfois préférable.

---

## Spécification Conventional Commits 1.0.0

La spécification Conventional Commits définit une structure normalisée pour les messages de commit qui permet l'automatisation du versioning et de la génération de changelogs.

### Structure d'un commit

```
<type>[scope optionnel]: <description>

[corps optionnel]

[pied de page optionnel]
```

### Types de commits standards

Les types suivants sont définis par la configuration `@commitlint/config-conventional` et correspondent à des sections du CHANGELOG :

| Type | Description | Impact SemVer | Section CHANGELOG |
|------|-------------|---------------|-------------------|
| `feat` | Nouvelle fonctionnalité | **MINOR** | Features |
| `fix` | Correction de bug | **PATCH** | Bug Fixes |
| `docs` | Documentation uniquement | Aucun | Documentation |
| `style` | Formatage, points-virgules manquants | Aucun | Masqué |
| `refactor` | Refactoring sans changement fonctionnel | Aucun | Code Refactoring |
| `perf` | Amélioration des performances | Aucun | Performance |
| `test` | Ajout ou correction de tests | Aucun | Masqué |
| `build` | Système de build, dépendances externes | Aucun | Masqué |
| `ci` | Configuration CI | Aucun | Masqué |
| `chore` | Tâches de maintenance | Aucun | Masqué |

### Breaking changes : deux syntaxes possibles

La notation `!` après le type signale un changement incompatible (déclenche un bump MAJOR) :
```bash
feat!: suppression de l'API legacy
feat(api)!: nouveau format de réponse
```

Le footer `BREAKING CHANGE:` offre une alternative plus descriptive :
```bash
feat: nouveau système de configuration

BREAKING CHANGE: le format du fichier de config passe de JSON à YAML
```

### Scopes recommandés pour Vue/Nuxt

Pour un projet Nuxt 4, définissez des scopes cohérents avec l'architecture du framework : `components`, `composables`, `pages`, `layouts`, `middleware`, `plugins`, `assets`, `utils`, `store`, `api`, `config`, `types`, `deps`.

---

## Configuration de commitlint avec pnpm

L'installation et la configuration de commitlint permettent de valider automatiquement les messages de commit via un hook Git.

### Installation

```bash
pnpm add -D @commitlint/cli @commitlint/config-conventional
```

### Fichier de configuration

Créez `commitlint.config.js` à la racine du projet :

```javascript
export default {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'build', 'ci', 'chore', 'revert']
    ],
    'scope-enum': [
      1, // Warning (non bloquant)
      'always',
      ['components', 'composables', 'pages', 'layouts', 'middleware', 'plugins', 'assets', 'utils', 'store', 'api', 'config', 'types', 'deps']
    ],
    'header-max-length': [2, 'always', 100],
    'subject-case': [0] // Désactivé pour plus de flexibilité
  }
};
```

### Commitizen pour les commits interactifs

Pour guider les développeurs avec des prompts interactifs :

```bash
pnpm add -D commitizen cz-conventional-changelog
```

Ajoutez dans `package.json` :
```json
{
  "scripts": {
    "commit": "cz"
  },
  "config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  }
}
```

Utilisez ensuite `pnpm commit` au lieu de `git commit` pour un assistant interactif.

---

## Husky vs simple-git-hooks avec pnpm 10.26+

**pnpm 10 bloque par défaut les scripts lifecycle des dépendances** (mesure de sécurité), mais le script `prepare` de votre propre projet s'exécute normalement — les deux solutions fonctionnent donc sans configuration supplémentaire.

### Comparaison détaillée

| Critère | Husky v9.1.7 | simple-git-hooks v2.13.1 |
|---------|--------------|--------------------------|
| **Downloads/semaine** | ~16M | ~260K |
| **Dépendances** | 1 | 0 |
| **Configuration** | Fichiers dans `.husky/` | Dans `package.json` |
| **Communauté** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Flexibilité** | Élevée | Moyenne |
| **Complexité setup** | Moyenne | Faible |

### Recommandation : Husky v9

Pour un projet Nuxt 4, **Husky v9** offre une documentation plus extensive, une meilleure flexibilité pour les workflows complexes, et une communauté plus large pour le support. Cependant, **simple-git-hooks** reste un excellent choix si vous préférez zero dépendances et une configuration entièrement dans `package.json`.

### Configuration Husky complète

```bash
# Installation
pnpm add -D husky lint-staged
pnpm exec husky init
```

Créez `.husky/pre-commit` :
```sh
npx lint-staged
```

Créez `.husky/commit-msg` :
```sh
npx --no -- commitlint --edit "$1"
```

Ajoutez dans `package.json` :
```json
{
  "scripts": {
    "prepare": "husky"
  },
  "lint-staged": {
    "*.{js,ts,vue}": ["eslint --fix", "prettier --write"],
    "*.{json,md,css,scss}": ["prettier --write"]
  }
}
```

### Alternative : simple-git-hooks

```bash
pnpm add -D simple-git-hooks lint-staged
npx simple-git-hooks
```

Configuration dans `package.json` :
```json
{
  "scripts": {
    "prepare": "simple-git-hooks"
  },
  "simple-git-hooks": {
    "pre-commit": "npx lint-staged",
    "commit-msg": "npx --no -- commitlint --edit $1"
  }
}
```

**Note importante** : après toute modification de la config simple-git-hooks, exécutez manuellement `npx simple-git-hooks` pour appliquer les changements.

---

## Semantic Versioning et gestion des versions

### Quand utiliser 0.x.x vs 1.x.x

La version **0.x.x** signale explicitement que l'API n'est pas stable — les changements peuvent survenir à tout moment. Pour un projet frontend/SSG, restez en 0.x.x pendant le développement actif et passez à **1.0.0** lorsque l'interface utilisateur et les fonctionnalités principales sont considérées stables pour la production.

**Comportement npm particulier** : pour les dépendances `^0.x.x`, npm n'autorise que les mises à jour patch (pas minor), traitant le minor comme indicateur de breaking change pour les versions 0.x.

### Pre-releases

L'ordre de précédence des pre-releases suit cette logique :
```
1.0.0-alpha.0 < 1.0.0-alpha.1 < 1.0.0-beta.0 < 1.0.0-rc.1 < 1.0.0
```

- **alpha** : Tests internes précoces
- **beta** : Fonctionnalités complètes, tests externes
- **rc** (release candidate) : Candidat à la production, derniers ajustements

---

## Format CHANGELOG et génération automatique

### Format recommandé : Keep a Changelog v1.1.0

Ce format offre la meilleure lisibilité humaine :

```markdown
# Changelog

Tous les changements notables de ce projet sont documentés dans ce fichier.

Le format est basé sur [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/),
et ce projet adhère au [Semantic Versioning](https://semver.org/lang/fr/).

## [Unreleased]

### Added
- Support du mode sombre

## [0.2.0] - 2025-12-30

### Added
- Composant de navigation responsive ([#15](https://github.com/user/repo/issues/15))
- Intégration Pinia pour le state management

### Fixed
- Erreur d'hydratation sur les pages SSG ([#23](https://github.com/user/repo/issues/23))

### Changed
- Migration vers Vue 3.5 pour les performances améliorées

## [0.1.0] - 2025-12-01

### Added
- Configuration initiale Nuxt 4
- Déploiement Cloudflare Pages

[Unreleased]: https://github.com/user/repo/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/user/repo/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/user/repo/releases/tag/v0.1.0
```

### Catégories du changelog

- **Added** : Nouvelles fonctionnalités
- **Changed** : Modifications de fonctionnalités existantes
- **Deprecated** : Fonctionnalités qui seront supprimées
- **Removed** : Fonctionnalités supprimées
- **Fixed** : Corrections de bugs
- **Security** : Corrections de vulnérabilités

---

## Scripts pnpm et workflow de release complet

### Configuration package.json recommandée

```json
{
  "name": "my-nuxt-app",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build",
    "generate": "nuxt generate",
    "preview": "nuxt preview",
    
    "prepare": "husky",
    "commit": "cz",
    
    "release": "commit-and-tag-version",
    "release:patch": "commit-and-tag-version --release-as patch",
    "release:minor": "commit-and-tag-version --release-as minor",
    "release:major": "commit-and-tag-version --release-as major",
    "release:alpha": "commit-and-tag-version --prerelease alpha",
    "release:beta": "commit-and-tag-version --prerelease beta",
    "release:rc": "commit-and-tag-version --prerelease rc",
    "release:dry-run": "commit-and-tag-version --dry-run",
    "release:first": "commit-and-tag-version --first-release",
    
    "postrelease": "git push --follow-tags origin main"
  },
  "commit-and-tag-version": {
    "types": [
      {"type": "feat", "section": "✨ Features"},
      {"type": "fix", "section": "🐛 Bug Fixes"},
      {"type": "perf", "section": "⚡ Performance"},
      {"type": "docs", "section": "📚 Documentation", "hidden": false},
      {"type": "refactor", "section": "♻️ Refactoring", "hidden": false},
      {"type": "style", "hidden": true},
      {"type": "test", "hidden": true},
      {"type": "build", "hidden": true},
      {"type": "ci", "hidden": true},
      {"type": "chore", "hidden": true}
    ],
    "header": "# Changelog\n\nTous les changements notables de ce projet sont documentés ici.\n\n",
    "releaseCommitMessageFormat": "chore(release): {{currentTag}}"
  },
  "lint-staged": {
    "*.{js,ts,vue}": ["eslint --fix", "prettier --write"],
    "*.{json,md,css,scss}": ["prettier --write"]
  },
  "config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  }
}
```

### Workflow de release manuel

```bash
# 1. S'assurer d'être sur main avec les derniers changements
git checkout main && git pull

# 2. Prévisualiser les changements (dry run)
pnpm release:dry-run

# 3. Créer la release (version auto-déterminée depuis les commits)
pnpm release

# 4. Le script postrelease pousse automatiquement
# Sinon : git push --follow-tags origin main
```

---

## GitHub Actions pour l'automatisation

### Tier gratuit GitHub Actions (décembre 2025)

- **Repos publics** : Minutes illimitées sur runners standards
- **Repos privés** : 2 000 minutes/mois (GitHub Free), 3 000 minutes/mois (Pro/Team)

### Workflow de release automatisé sur tag

Créez `.github/workflows/release.yml` :

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: pnpm/action-setup@v4
        with:
          version: 10

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm generate

      - name: Extract changelog for this version
        id: changelog
        run: |
          VERSION=${GITHUB_REF#refs/tags/v}
          CHANGELOG=$(awk "/^## \[${VERSION}\]/{flag=1; next} /^## \[/{flag=0} flag" CHANGELOG.md)
          echo "changelog<<EOF" >> $GITHUB_OUTPUT
          echo "$CHANGELOG" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          body: ${{ steps.changelog.outputs.changelog }}
          prerelease: ${{ contains(github.ref, 'alpha') || contains(github.ref, 'beta') || contains(github.ref, 'rc') }}

      - name: Deploy to Cloudflare Pages
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy .output/public --project-name=${{ vars.CLOUDFLARE_PROJECT_NAME }}
```

### Secrets requis

Configurez dans Settings > Secrets and variables > Actions :
- `CLOUDFLARE_API_TOKEN` : Token avec permission "Cloudflare Pages — Edit"
- `CLOUDFLARE_ACCOUNT_ID` : ID de votre compte Cloudflare

---

## Stack finale recommandée

| Composant | Outil | Justification |
|-----------|-------|---------------|
| **Versioning tool** | commit-and-tag-version | Fork maintenu de standard-version, intégration CHANGELOG native |
| **Commit validation** | commitlint + @commitlint/config-conventional | Standard de l'industrie |
| **Git hooks** | Husky v9 | Communauté large, documentation Vue/Nuxt |
| **Pre-commit linting** | lint-staged | Lint uniquement les fichiers modifiés |
| **Commits interactifs** | commitizen (optionnel) | Aide à l'adoption des conventions |
| **CI/CD** | GitHub Actions | Gratuit pour repos publics |
| **Déploiement** | Cloudflare Pages via wrangler-action | SSG optimisé, CDN global |

### Installation complète en une commande

```bash
pnpm add -D commit-and-tag-version @commitlint/cli @commitlint/config-conventional husky lint-staged commitizen cz-conventional-changelog && pnpm exec husky init && echo 'npx lint-staged' > .husky/pre-commit && echo 'npx --no -- commitlint --edit "$1"' > .husky/commit-msg
```

Cette configuration offre un workflow de release professionnel à coût zéro, avec validation automatique des commits, génération de changelogs lisibles, et déploiement continu sur Cloudflare Pages.