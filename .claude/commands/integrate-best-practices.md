---
description: 'Analyse un fichier de bonnes pratiques et intègre les éléments pertinents dans l''architecture du projet'
---

# Integrate Best Practices

Analyse le fichier de bonnes pratiques fourni par l'utilisateur et intègre les éléments pertinents dans la documentation d'architecture du projet.

## Fichier à analyser

$ARGUMENTS

## Instructions

### Étape 1 : Lecture et contexte

1. Lire le fichier de bonnes pratiques fourni
2. Lire les fichiers d'architecture existants :
   - `_bmad-output/planning-artifacts/architecture/core-architectural-decisions.md`
   - `_bmad-output/planning-artifacts/architecture/implementation-patterns-consistency-rules.md`
   - `_bmad-output/planning-artifacts/architecture/starter-template-evaluation.md`

### Étape 2 : Analyse comparative

Présenter à l'utilisateur une analyse structurée en 3 catégories :

#### ✅ Éléments déjà couverts
Lister les éléments du fichier de bonnes pratiques qui sont déjà documentés dans l'architecture existante (pas besoin d'ajouter).

#### ✅ Éléments pertinents à intégrer
Lister les éléments nouveaux et pertinents pour le projet, avec une brève explication de leur valeur ajoutée.

#### ⚠️ Éléments NON pertinents ou incorrects
Identifier les éléments qui :
- Ne correspondent pas au contexte du projet (ex: SSR vs SSG)
- Contiennent des informations erronées ou obsolètes
- Sont du over-engineering pour le scope du projet

Expliquer pourquoi chaque élément n'est pas pertinent.

### Étape 3 : Confirmation utilisateur

Demander à l'utilisateur quels éléments il souhaite intégrer :
- Tous les éléments pertinents identifiés
- Une sélection spécifique
- Aucun

### Étape 4 : Intégration

Pour chaque élément à intégrer :
1. Identifier le fichier d'architecture approprié
2. Trouver la section la plus pertinente
3. Ajouter le contenu de manière cohérente avec le style existant
4. Utiliser des exemples de code TypeScript/Vue quand applicable

### Règles importantes

- **Contexte projet** : Blog SSG Nuxt 4 + Content 3 + Cloudflare Pages
- **Ne pas ajouter** : Configurations SSR/edge-only, D1 si non nécessaire pour SSG, ISR/SWR non supportés par CF Pages
- **Toujours vérifier** : La documentation officielle en cas de doute sur une fonctionnalité
- **Format** : Maintenir la cohérence avec le formatage existant (tableaux, code blocks, headers)
