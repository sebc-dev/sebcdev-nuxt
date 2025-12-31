# OKLCH : Espace Colorimétrique Moderne

## Syntaxe

```css
oklch(L C H / alpha)
```

| Paramètre | Plage | Description |
|-----------|-------|-------------|
| **L** (Lightness) | 0-1 | Luminosité perceptuelle (0 = noir, 1 = blanc) |
| **C** (Chroma) | 0-0.4 | Intensité chromatique (saturation) |
| **H** (Hue) | 0-360° | Teinte sur le cercle chromatique |
| **alpha** | 0-1 | Opacité (optionnel) |

## Avantages par rapport à HSL/RGB

| Aspect | HSL/RGB | OKLCH |
|--------|---------|-------|
| **Uniformité perceptuelle** | ❌ Jaune et bleu à 50% paraissent très différents | ✅ Changement de L produit résultat visuel cohérent |
| **Gamut** | sRGB uniquement | Display P3, Rec2020 |
| **Gradients** | Zones grises entre couleurs saturées | Transitions fluides |
| **Prédictibilité** | Ajustements imprévisibles | Résultats attendus |

## Exemples de palettes

```css
/* Nuances de bleu cohérentes */
--blue-100: oklch(0.95 0.05 250);
--blue-300: oklch(0.75 0.12 250);
--blue-500: oklch(0.55 0.18 250);
--blue-700: oklch(0.35 0.15 250);
--blue-900: oklch(0.20 0.10 250);

/* Couleurs sémantiques */
--success: oklch(0.65 0.15 145);   /* Vert */
--warning: oklch(0.80 0.15 85);    /* Orange */
--error: oklch(0.55 0.20 25);      /* Rouge */
--info: oklch(0.60 0.15 250);      /* Bleu */
```

## Support navigateur

**92.86%** global (Chrome 111+, Safari 15.4+, Firefox 113+).

Lightning CSS transpile automatiquement vers RGB avec fallbacks si nécessaire.
