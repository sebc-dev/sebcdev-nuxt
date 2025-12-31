# `focus-visible:` vs `focus:`

## Différence comportementale

| Pseudo-classe | Déclenchement | Usage |
|---------------|---------------|-------|
| `:focus` | Tout focus (clavier, souris, programmatique) | ❌ Éviter |
| `:focus-visible` | Focus clavier uniquement (heuristique navigateur) | ✅ Recommandé |

## Pourquoi utiliser `focus-visible:`

```html
<!-- ❌ Focus ring apparaît au clic souris (mauvaise UX) -->
<button class="focus:ring-2 focus:ring-blue-500">Clic</button>

<!-- ✅ Focus ring uniquement au Tab clavier -->
<button class="focus-visible:ring-2 focus-visible:ring-blue-500">Tab</button>
```

**Règle** : Toujours utiliser `focus-visible:` pour les indicateurs visuels de focus, sauf besoin spécifique d'indiquer le focus au clic (champs de formulaire par exemple).

---
