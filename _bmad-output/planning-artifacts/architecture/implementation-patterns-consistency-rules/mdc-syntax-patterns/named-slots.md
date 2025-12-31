# Named Slots

## Syntaxe slots nommés

```markdown
::hero{image="/images/bg.jpg"}
#title
Bienvenue sur notre site

#description
Une description avec du **Markdown** formaté.

Contenu du slot par défaut ici.
::
```

## Attribut `mdc-unwrap`

Le parser Markdown enveloppe automatiquement le texte dans des balises `<p>`. Pour les slots où `<p>` serait invalide (ex: dans un `<h1>`), utiliser `mdc-unwrap` :

```vue
<!-- app/components/content/Hero.vue -->
<template>
  <section class="hero">
    <!-- Sans mdc-unwrap: <h1><p>Titre</p></h1> = HTML invalide -->
    <!-- Avec mdc-unwrap: <h1>Titre</h1> = HTML valide -->
    <h1 v-if="$slots.title" class="hero__title">
      <slot name="title" mdc-unwrap="p" />
    </h1>

    <p v-if="$slots.description" class="hero__description">
      <slot name="description" mdc-unwrap="p" />
    </p>

    <!-- Slot par défaut - garder les <p> -->
    <div class="hero__content">
      <slot />
    </div>
  </section>
</template>
```

**Valeurs possibles pour `mdc-unwrap` :**
- `"p"` - Supprime les balises `<p>`
- `"li"` - Supprime les balises `<li>` (pour listes)
- `"p li"` - Supprime les deux

## Vérification conditionnelle des slots

```vue
<template>
  <!-- Afficher seulement si le slot est fourni -->
  <footer v-if="$slots.footer" class="card-footer">
    <slot name="footer" mdc-unwrap="p" />
  </footer>
</template>
```

**Conventions de nommage slots :**
- `#title` - Titre principal
- `#description` - Description courte
- `#content` - Contenu principal étendu
- `#footer` - Pied de composant
- `#default` - Slot principal (implicite)
