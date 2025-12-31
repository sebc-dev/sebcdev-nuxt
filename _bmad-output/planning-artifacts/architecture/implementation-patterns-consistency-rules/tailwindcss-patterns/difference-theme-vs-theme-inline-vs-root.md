# Différence @theme vs @theme inline vs :root

## Tableau comparatif

| Déclaration | Génère classes Tailwind | Utilisable via `var()` | Modifiable par JS |
|-------------|-------------------------|------------------------|-------------------|
| `@theme {}` | ✅ Oui | ❌ Non | ❌ Non |
| `@theme inline {}` | ✅ Oui | ✅ Oui | ✅ Oui |
| `:root {}` | ❌ Non | ✅ Oui | ✅ Oui |

## Quand utiliser quoi

| Cas d'usage | Déclaration recommandée |
|-------------|------------------------|
| Valeurs statiques (fonts, breakpoints) | `@theme {}` |
| Theming dynamique (dark mode, marques) | `:root` + `@theme inline` |
| Variables CSS pures (non-Tailwind) | `:root {}` |

## Pattern theming dynamique

```css
/* 1. Définir les variables dans :root et .dark */
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
}

/* 2. Connecter à Tailwind via @theme inline */
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
}
```

**Résultat** : Les classes `bg-background`, `text-foreground` changent automatiquement quand la classe `.dark` est ajoutée au `<html>`.
