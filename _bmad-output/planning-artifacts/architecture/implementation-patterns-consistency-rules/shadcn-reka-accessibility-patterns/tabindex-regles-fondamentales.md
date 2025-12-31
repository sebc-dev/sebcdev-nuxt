# Tabindex : Règles Fondamentales

## Valeurs et usages

| Valeur | Comportement | Cas d'utilisation |
|--------|--------------|-------------------|
| **Absent** | Ordre naturel du DOM | Éléments HTML interactifs natifs (`<button>`, `<a>`, `<input>`) |
| `tabindex="0"` | Inclus dans l'ordre naturel | Éléments custom rendus interactifs |
| `tabindex="-1"` | Focusable par JS uniquement | Conteneurs de focus, navigation programmatique |
| `tabindex > 0` | **❌ INTERDIT** | Jamais - casse l'ordre prévisible |

## Règle : Utiliser les éléments HTML natifs en priorité

Les éléments `<button>`, `<a>`, `<input>` possèdent un comportement clavier natif (Enter, Space) et sont déjà dans l'ordre de tabulation.

```vue
<template>
  <!-- ✅ Éléments natifs - aucun tabindex nécessaire -->
  <button @click="handleAction">Action</button>
  <a href="/page">Navigation</a>

  <!-- ✅ tabindex="0" pour élément custom interactif -->
  <div
    role="button"
    tabindex="0"
    @click="handleClick"
    @keydown.enter="handleClick"
    @keydown.space.prevent="handleClick"
  >
    Bouton personnalisé
  </div>

  <!-- ✅ tabindex="-1" pour focus programmatique -->
  <main id="main-content" tabindex="-1">
    <!-- Le focus est dirigé ici après navigation -->
  </main>
</template>
```

## Anti-patterns tabindex

```vue
<template>
  <!-- ❌ tabindex positif - détruit l'ordre prévisible -->
  <button tabindex="1">Premier</button>
  <button tabindex="2">Deuxième</button>

  <!-- ❌ Div cliquable sans accessibilité clavier -->
  <div @click="action">Cliquable mais pas accessible</div>

  <!-- ❌ tabindex="0" sur élément non-interactif -->
  <p tabindex="0">Ce paragraphe n'a pas besoin d'être focusable</p>

  <!-- ❌ Retirer un élément interactif de la tabulation -->
  <button tabindex="-1">Bouton inaccessible au clavier</button>
</template>
```

---
