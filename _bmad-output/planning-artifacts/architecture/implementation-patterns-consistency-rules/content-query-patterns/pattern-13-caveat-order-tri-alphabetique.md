# Pattern 13 : Caveat `.order()` - Tri alphabétique

⚠️ **Attention** : `.order()` effectue un **tri alphabétique**, pas numérique.

## Le problème

```
# Fichiers avec préfixes numériques
content/
├── 1.intro.md
├── 2.basics.md
├── 10.advanced.md
├── 11.expert.md

# Résultat du tri alphabétique : 1, 10, 11, 2 ❌
```

## La solution : zero-padding

```
# Fichiers avec zero-padding
content/
├── 01.intro.md
├── 02.basics.md
├── 10.advanced.md
├── 11.expert.md

# Résultat du tri alphabétique : 01, 02, 10, 11 ✅
```

## Recommandation

Toujours utiliser le **zero-padding** pour les préfixes numériques dans les noms de fichiers :

| ❌ Mauvais | ✅ Correct |
|-----------|-----------|
| `1.intro.md` | `01.intro.md` |
| `9.chapter.md` | `09.chapter.md` |
| `10.finale.md` | `10.finale.md` |
