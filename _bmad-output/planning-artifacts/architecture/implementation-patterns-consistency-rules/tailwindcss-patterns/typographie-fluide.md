# Typographie Fluide

## Tokens typographiques avec clamp()

L'échelle typographique fluide utilise `clamp()` avec des propriétés CSS associées (line-height, font-weight, letter-spacing) :

```css
@theme {
  /* Corps de texte fluide */
  --text-fluid-body: clamp(1rem, 0.913rem + 0.4348vw, 1.2rem);
  --text-fluid-body--line-height: 1.6;

  /* Titres fluides avec propriétés associées */
  --text-fluid-h1: clamp(2.488rem, 1.943rem + 2.726vw, 3.5rem);
  --text-fluid-h1--line-height: 1.15;
  --text-fluid-h1--font-weight: 700;
  --text-fluid-h1--letter-spacing: -0.02em;

  --text-fluid-h2: clamp(1.953rem, 1.586rem + 1.835vw, 2.625rem);
  --text-fluid-h2--line-height: 1.2;
  --text-fluid-h2--font-weight: 600;
  --text-fluid-h2--letter-spacing: -0.015em;

  --text-fluid-h3: clamp(1.563rem, 1.318rem + 1.222vw, 2rem);
  --text-fluid-h3--line-height: 1.3;
  --text-fluid-h3--font-weight: 600;

  /* Display pour hero sections */
  --text-fluid-display: clamp(3rem, 2rem + 5vw, 6rem);
  --text-fluid-display--line-height: 1.05;
  --text-fluid-display--font-weight: 800;
  --text-fluid-display--letter-spacing: -0.03em;

  /* Small/caption fluide */
  --text-fluid-small: clamp(0.833rem, 0.769rem + 0.321vw, 0.9rem);
  --text-fluid-small--line-height: 1.5;
}
```

## Usage dans les templates

```html
<!-- Utilisation directe du token -->
<h1 class="text-(--text-fluid-h1) leading-(--text-fluid-h1--line-height) font-(--text-fluid-h1--font-weight) tracking-(--text-fluid-h1--letter-spacing)">
  Titre principal
</h1>

<!-- Avec classes custom dans @layer -->
<h1 class="text-fluid-h1">Titre principal</h1>
```

## Classes utilitaires recommandées

Pour simplifier l'usage, définir des classes dans `@layer utilities` :

```css
@layer utilities {
  .text-fluid-body {
    font-size: var(--text-fluid-body);
    line-height: var(--text-fluid-body--line-height);
  }

  .text-fluid-h1 {
    font-size: var(--text-fluid-h1);
    line-height: var(--text-fluid-h1--line-height);
    font-weight: var(--text-fluid-h1--font-weight);
    letter-spacing: var(--text-fluid-h1--letter-spacing);
  }

  .text-fluid-h2 {
    font-size: var(--text-fluid-h2);
    line-height: var(--text-fluid-h2--line-height);
    font-weight: var(--text-fluid-h2--font-weight);
    letter-spacing: var(--text-fluid-h2--letter-spacing);
  }

  .text-fluid-display {
    font-size: var(--text-fluid-display);
    line-height: var(--text-fluid-display--line-height);
    font-weight: var(--text-fluid-display--font-weight);
    letter-spacing: var(--text-fluid-display--letter-spacing);
  }
}
```

## Calcul des valeurs clamp()

Formule pour calculer la valeur préférée `vw` :

```
preferred = (maxSize - minSize) / (maxViewport - minViewport) × 100vw
```

| Paramètre | Valeur typique |
|-----------|----------------|
| `minViewport` | 320px (mobile) |
| `maxViewport` | 1440px (desktop) |
| `minSize` | Taille minimum souhaitée |
| `maxSize` | Taille maximum souhaitée |

**Outil recommandé** : [Utopia Fluid Type Scale](https://utopia.fyi/type/calculator/) pour générer les valeurs automatiquement.

---
