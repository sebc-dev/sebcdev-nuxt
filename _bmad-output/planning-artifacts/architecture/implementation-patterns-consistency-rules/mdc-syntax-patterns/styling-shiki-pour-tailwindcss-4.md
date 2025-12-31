# Styling Shiki pour TailwindCSS 4

## CSS Variables Dark Mode

Shiki génère automatiquement des CSS variables pour chaque token lors du dual-theme. L'intégration avec TailwindCSS 4 et la classe `html.dark` nécessite ce CSS :

```css
/* app/assets/css/shiki.css (à importer dans main.css) */

/* Switch automatique dark mode via CSS variables Shiki */
html.dark .shiki,
html.dark .shiki span {
  color: var(--shiki-dark) !important;
  background-color: var(--shiki-dark-bg) !important;
  font-style: var(--shiki-dark-font-style) !important;
}
```

## Styling lignes highlight/diff (oklch)

Les transformers Shiki ajoutent des classes CSS mais ne fournissent pas le styling. Ajouter ces styles pour les lignes surlignées et les diffs :

```css
/* Lignes surlignées */
.shiki .line.highlighted {
  background: oklch(0.95 0.01 250);
}
html.dark .shiki .line.highlighted {
  background: oklch(0.25 0.02 250);
}

/* Diffs add/remove */
.shiki .line.diff.add {
  background: oklch(0.95 0.05 145);
}
html.dark .shiki .line.diff.add {
  background: oklch(0.30 0.08 145);
}

.shiki .line.diff.remove {
  background: oklch(0.95 0.05 25);
}
html.dark .shiki .line.diff.remove {
  background: oklch(0.30 0.08 25);
}

/* Focus avec blur des autres lignes */
.shiki.has-focused .line:not(.focused) {
  opacity: 0.5;
  filter: blur(1px);
  transition: opacity 0.3s, filter 0.3s;
}

/* Annotations erreur */
.shiki .line.highlighted.error {
  background: oklch(0.95 0.08 25);
}
html.dark .shiki .line.highlighted.error {
  background: oklch(0.30 0.10 25);
}
```

## Accessibilité : Focus Visible

Shiki ajoute automatiquement `tabindex="0"` aux éléments `<pre>` pour la navigation clavier. Ajouter un outline visible pour le focus :

```css
/* Focus visible pour navigation clavier */
pre.shiki:focus-visible {
  outline: 2px solid oklch(0.6 0.15 250);
  outline-offset: 2px;
}

html.dark pre.shiki:focus-visible {
  outline-color: oklch(0.7 0.15 250);
}
```

## Icônes de langage (Nuxt UI)

Pour afficher des icônes de langage dans les headers de code blocks avec Nuxt UI, configurer `app.config.ts` :

```typescript
// app.config.ts
export default defineAppConfig({
  ui: {
    prose: {
      codeIcon: {
        // Fichiers spécifiques
        'nuxt.config.ts': 'i-vscode-icons-file-type-nuxt',
        'package.json': 'i-vscode-icons-file-type-npm',
        'tsconfig.json': 'i-vscode-icons-file-type-tsconfig',
        'tailwind.config.ts': 'i-vscode-icons-file-type-tailwind',
        '.env': 'i-vscode-icons-file-type-dotenv',

        // Langages
        ts: 'i-vscode-icons-file-type-typescript',
        typescript: 'i-vscode-icons-file-type-typescript',
        vue: 'i-vscode-icons-file-type-vue',
        js: 'i-vscode-icons-file-type-js',
        json: 'i-vscode-icons-file-type-json',
        css: 'i-vscode-icons-file-type-css',
        html: 'i-vscode-icons-file-type-html',
        md: 'i-vscode-icons-file-type-markdown',
        yaml: 'i-vscode-icons-file-type-yaml',
        shell: 'i-vscode-icons-file-type-shell',
        bash: 'i-vscode-icons-file-type-shell',
      }
    }
  }
})
```

**Prérequis :** Installer `@iconify-json/vscode-icons` pour les icônes VS Code.

---
