# Syntaxe de base

## Composants inline vs block

| Type | Syntaxe | Usage |
|------|---------|-------|
| **Inline** | `:component[contenu]{props}` | Éléments simples sans slots (badges, icônes) |
| **Block** | `::component{props}` ... `::` | Contenu Markdown complet avec slots |

```markdown
<!-- Inline - badge simple -->
:badge[Nouveau]{type="success"}

<!-- Block - carte avec contenu -->
::card{title="Ma carte"}
Contenu Markdown avec **formatage**.
::
```

## Props et attributs

```markdown
<!-- Props simples inline -->
::alert{type="warning" title="Attention"}

<!-- Props complexes avec YAML -->
::icon-card
---
icon: IconNuxt
title: Architecture Nuxt
description: Exploitez toute la puissance de Nuxt
---
::

<!-- Binding dynamique depuis frontmatter -->
::alert{:type="frontmatterType"}

<!-- Tableaux JSON (guillemets simples externes, doubles internes) -->
:tags{:items='["Nuxt", "Vue", "TypeScript"]'}

<!-- Classes et ID -->
::div{.bg-blue-100 .p-4 .rounded-lg #section-intro}
```

## Nesting (imbrication)

Ajouter des colons pour chaque niveau d'imbrication :

```markdown
::container
:::grid{cols="2"}
::::card{title="Carte 1"}
Contenu carte 1
::::

::::card{title="Carte 2"}
Contenu carte 2
::::
:::
::
```

**Limite recommandée : 3-4 niveaux maximum.** Au-delà, restructurer en sous-composants.

## Piège du spécificateur vide

Quand un composant inline est suivi de `-`, `_` ou `:`, ajouter un spécificateur vide :

```markdown
<!-- Problème : le - est interprété comme partie du composant -->
:icon-world

<!-- Solution : spécificateur vide {} -->
:icon{}-world
```
