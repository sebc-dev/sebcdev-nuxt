# Structure des Headings (SSG Multi-Pages)

La hiérarchie des titres `h1` à `h6` est le **principal mécanisme de navigation** pour les utilisateurs de lecteurs d'écran (touche H dans NVDA/JAWS).

## Règles fondamentales

| Règle | Explication |
|-------|-------------|
| **Un seul `<h1>` par page** | Décrit le contenu principal de la page |
| **Layout ne définit JAMAIS le `<h1>`** | Le `<h1>` appartient au contenu spécifique |
| **Pas de saut vers le bas** | `h2` suivi de `h4` = interdit |
| **Saut vers le haut autorisé** | `h4` suivi de `h2` = OK (ferme la sous-section) |

## Pattern Layout SSG

```vue
<!-- layouts/default.vue -->
<template>
  <div>
    <header>
      <nav aria-label="Navigation principale">
        <!-- Navigation sans h1 -->
      </nav>
    </header>

    <main id="main-content">
      <!-- Le slot contiendra le h1 de la page -->
      <slot />
    </main>

    <footer>
      <!-- h2 maximum dans le footer -->
    </footer>
  </div>
</template>
```

```vue
<!-- pages/about.vue -->
<script setup lang="ts">
useSeoMeta({ title: 'À propos - Mon Site' })
</script>

<template>
  <article>
    <h1>À propos de notre équipe</h1>

    <section aria-labelledby="history-heading">
      <h2 id="history-heading">Notre histoire</h2>
      <h3>Les débuts en 2015</h3>
      <h3>L'expansion internationale</h3>
    </section>

    <section aria-labelledby="values-heading">
      <h2 id="values-heading">Nos valeurs</h2>
    </section>
  </article>
</template>
```

---
