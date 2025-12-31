# Technique Double-Ring Focus

## Problème

Un seul outline coloré échoue sur certains fonds : le bleu disparaît sur fond sombre, le blanc sur fond clair.

## Solution mathématique

En superposant deux anneaux avec un contraste **9:1 entre eux**, au moins l'un des deux est **garanti** d'avoir un contraste 3:1 contre n'importe quel fond solide.

## Implémentation CSS

```css
*:focus-visible {
  outline: 2px solid #FFFFFF;  /* Anneau intérieur */
  outline-offset: 0;
  box-shadow: 0 0 0 4px #193146;  /* Anneau extérieur */
}

/* Le box-shadow de 4px : l'outline de 2px le chevauche,
   seuls 2px du shadow seront visibles */
```

## Implémentation TailwindCSS 4

```html
<button class="
  focus-visible:outline-2
  focus-visible:outline-white
  focus-visible:outline-offset-0
  focus-visible:ring-4
  focus-visible:ring-slate-900
  dark:focus-visible:outline-black
  dark:focus-visible:ring-white
">
  Bouton accessible
</button>
```

## Inversion dark mode

| Mode | Outline (intérieur) | Ring/Shadow (extérieur) |
|------|---------------------|-------------------------|
| Light | Blanc `#FFFFFF` | Sombre `#193146` |
| Dark | Noir `#0A0A0A` | Clair `#FAFAFA` |

## ⚠️ Règle critique : ne jamais supprimer outline

```css
/* ❌ INTERDIT - invisible en Windows High Contrast Mode */
*:focus-visible {
  outline: none;
  box-shadow: 0 0 0 4px blue;
}

/* ✅ CORRECT - maintient l'accessibilité WHCM */
*:focus-visible {
  outline: 2px solid transparent;  /* Transparent mais présent */
  box-shadow: 0 0 0 4px blue;
}
```

Les user agents suppriment souvent les `box-shadow` en mode couleurs forcées (High Contrast). L'`outline` est préservé.

---
