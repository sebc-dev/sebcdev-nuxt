# Configuration Shiki dans Nuxt Content 3.10+ : guide complet

La syntaxe multi-thèmes `{ default: 'github-light', dark: 'github-dark' }` reste parfaitement valide dans Nuxt Content 3.10+, mais le chemin de configuration a migré vers `content.build.markdown.highlight`. L'intégration avec TailwindCSS 4 se fait via CSS variables générées automatiquement par Shiki, compatibles avec votre mode sombre oklch. Pour un blog SSG déployé sur Cloudflare Pages, le highlighting se fait entièrement au build — **zéro JavaScript côté client**.

---

## Configuration multi-thèmes : la nouvelle syntaxe v3

Le changement majeur entre Content 2.x et 3.x concerne l'emplacement de la configuration. Le chemin `content.highlight` est désormais **déprécié** ; utilisez `content.build.markdown.highlight`.

```typescript
// nuxt.config.ts — Configuration recommandée pour Nuxt Content 3.10+
export default defineNuxtConfig({
  modules: ['@nuxt/content'],
  content: {
    build: {
      markdown: {
        highlight: {
          theme: {
            default: 'github-light',
            dark: 'github-dark'
          },
          langs: ['json', 'js', 'ts', 'html', 'css', 'vue', 'shell', 'yaml', 'md', 'mdc']
        }
      }
    }
  }
})
```

**Attention** : le fichier `content.config.ts` nouvellement introduit sert uniquement à définir les collections. La configuration Shiki reste dans `nuxt.config.ts`. Les langages par défaut chargés sont `['json', 'js', 'ts', 'html', 'css', 'vue', 'shell', 'mdc', 'md', 'yaml']` — ajoutez seulement ceux dont vous avez besoin pour minimiser le bundle.

### Thèmes recommandés pour l'accessibilité

Pour un contraste optimal avec TailwindCSS 4 et ses tokens oklch, privilégiez les thèmes haute-contraste :

- **Light** : `github-light-high-contrast`, `min-light`, `vitesse-light`
- **Dark** : `github-dark-high-contrast`, `nord`, `one-dark-pro`
- **Paires équilibrées** : `github-light` / `github-dark` (ratio ≥4.5:1 WCAG AA)

Le thème `css-variables` est **cassé depuis la migration vers Shikiji** (Content 2.8.5+). Utilisez systématiquement les thèmes nommés avec dual-theme.

---

## Intégration mode sombre TailwindCSS 4

Shiki génère automatiquement des CSS variables pour chaque token lors du dual-theme. L'intégration avec TailwindCSS 4 nécessite ce CSS minimal :

```css
/* assets/css/shiki.css */
html.dark .shiki,
html.dark .shiki span {
  color: var(--shiki-dark) !important;
  background-color: var(--shiki-dark-bg) !important;
  font-style: var(--shiki-dark-font-style) !important;
}
```

Pour les navigateurs modernes supportant `light-dark()`, configurez `defaultColor: 'light-dark()'` dans Shiki pour un switch automatique basé sur `prefers-color-scheme`. Avec `@nuxtjs/color-mode`, le toggle classe `html.dark` déclenche le changement via les variables CSS générées.

---

## Line highlighting et syntaxe MDC complète

La mise en surbrillance de lignes utilise la syntaxe `{lignes}` après le langage :

```markdown
```ts {1,3-5}
// Ligne 1 surlignée
const config = {
  // Lignes 3-5 surlignées
  theme: 'github-dark',
  langs: ['ts', 'vue']
}
```
```

### Options de highlighting disponibles

| Syntaxe | Effet | Classes CSS générées |
|---------|-------|---------------------|
| `{2}` | Ligne unique | `.line.highlighted` |
| `{1,3,5}` | Lignes multiples | `.line.highlighted` |
| `{3-7}` | Plage de lignes | `.line.highlighted` |
| `// [!code highlight]` | Via commentaire inline | `.line.highlighted`, `.has-highlighted` sur `<pre>` |
| `// [!code focus]` | Focus avec blur des autres lignes | `.line.focused`, `.has-focused` |
| `// [!code ++]` / `// [!code --]` | Diff add/remove | `.line.diff.add` / `.line.diff.remove` |
| `// [!code error]` | Annotation erreur | `.line.highlighted.error` |

**Important** : ces transformers ajoutent uniquement des classes CSS. Vous devez fournir le styling :

```css
/* Styling des lignes highlight/diff */
.shiki .line.highlighted { background: oklch(0.95 0.01 250); }
html.dark .shiki .line.highlighted { background: oklch(0.25 0.02 250); }
.shiki .line.diff.add { background: oklch(0.95 0.05 145); }
.shiki .line.diff.remove { background: oklch(0.95 0.05 25); }
```

---

## Filename et header dans les code blocks

La syntaxe `[filename]` après le langage affiche le nom de fichier :

```markdown
```typescript [nuxt.config.ts]{4-5}
export default defineNuxtConfig({
  content: {
    build: {
      markdown: {} // Ces lignes seront highlight
    }
  }
})
```
```

Le composant `ProsePre` reçoit ces props parsées :

```typescript
{
  code: "...",
  language: "typescript",
  filename: "nuxt.config.ts",
  highlights: [4, 5],
  meta: "meta-info=val"
}
```

Pour les icônes de langage avec Nuxt UI, configurez `app.config.ts` :

```typescript
// app.config.ts
export default defineAppConfig({
  ui: {
    prose: {
      codeIcon: {
        'nuxt.config.ts': 'i-vscode-icons-file-type-nuxt',
        ts: 'i-vscode-icons-file-type-typescript',
        vue: 'i-vscode-icons-file-type-vue'
      }
    }
  }
})
```

---

## Code groups avec persistance utilisateur

La syntaxe `::code-group` crée des onglets pour alternatives :

```markdown
::code-group

```bash [pnpm]
pnpm add @nuxt/content
```

```bash [npm]
npm install @nuxt/content
```

```bash [yarn]
yarn add @nuxt/content
```

::
```

Pour la **persistance du choix utilisateur**, implémentez un composant custom `ProseCodeGroup.vue` :

```vue
<!-- components/content/ProseCodeGroup.vue -->
<script setup>
const STORAGE_KEY = 'preferred-package-manager'
const activeTab = ref(0)

onMounted(() => {
  const saved = localStorage.getItem(STORAGE_KEY)
  if (saved) activeTab.value = parseInt(saved)
})

watch(activeTab, (val) => {
  localStorage.setItem(STORAGE_KEY, val.toString())
})
</script>
```

Avec shadcn-vue/Reka UI, utilisez les composants `Tabs`, `TabsList`, `TabsTrigger`, `TabsContent` pour le styling des onglets.

---

## Optimisation SSG et Cloudflare Pages

Shiki génère du **HTML pur avec styles inline au build** — aucun JS/CSS supplémentaire envoyé au client. C'est optimal pour SSG.

### Configuration production optimisée

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/content'],
  
  content: {
    build: {
      markdown: {
        highlight: {
          theme: { default: 'github-light', dark: 'github-dark' },
          // Minimal set — évitez le bundle full
          langs: ['json', 'js', 'ts', 'html', 'css', 'vue', 'shell', 'yaml']
        }
      }
    }
  },

  nitro: {
    preset: 'cloudflare-pages',
    prerender: { routes: ['/'], crawlLinks: true }
  },

  routeRules: {
    '/blog/**': { 
      prerender: true,
      headers: { 'cache-control': 'public, max-age=31536000, immutable' }
    }
  }
})
```

### Impact bundle size

| Bundle Shiki | Minifié | Gzippé |
|--------------|---------|--------|
| `shiki/bundle/full` | **6.4 MB** | 1.2 MB |
| `shiki/bundle/web` | 3.8 MB | 695 KB |
| Fine-grained (core) | ~12 KB + thèmes/langages | Variable |

**Recommandation** : Nuxt Content utilise déjà une approche fine-grained via `@nuxtjs/mdc`. Limitez les langages dans `langs` pour réduire le payload. Le **JavaScript RegExp engine** est plus léger que Oniguruma WASM.

---

## Accessibilité des code blocks

Shiki ajoute automatiquement `tabindex="0"` aux éléments `<pre>` pour la navigation clavier des blocs scrollables. Points critiques à vérifier :

- **Contraste** : Les thèmes `*-high-contrast` garantissent un ratio ≥5.5:1
- **Screen readers** : Le HTML sémantique généré (`<pre><code>`) est correctement interprété
- **Focus visible** : Ajoutez un outline sur `:focus-visible` pour `<pre>`

```css
pre.shiki:focus-visible {
  outline: 2px solid oklch(0.6 0.15 250);
  outline-offset: 2px;
}
```

Pour corriger des couleurs problématiques dans un thème, utilisez `colorReplacements` :

```typescript
highlight: {
  theme: 'min-dark',
  colorReplacements: {
    '#ff79c6': '#189eff' // Remplace une couleur low-contrast
  }
}
```

---

## Breaking changes : Content 2.x vers 3.x

### Changements de configuration

| Aspect | Content 2.x | Content 3.x |
|--------|-------------|-------------|
| Chemin config | `content.highlight` | `content.build.markdown.highlight` |
| Langages | `preload: ['ts']` | `langs: ['ts']` |
| Tags mapping | `content.markdown.tags` | `content.renderer.alias` |

### Renommage des composants Prose (critique)

| Composant v2 | Composant v3 | Usage |
|--------------|--------------|-------|
| `ProseCodeInline` | `ProseCode` | `` `inline` `` |
| `ProseCode` | `ProsePre` | ` ``` blocks ``` ` |

Si vous avez un composant custom `ProseCode.vue` pour les blocs, **renommez-le en `ProsePre.vue`**.

### Nouveau fichier content.config.ts

```typescript
// content.config.ts — pour les collections uniquement
import { defineContentConfig, defineCollection } from '@nuxt/content'

export default defineContentConfig({
  collections: {
    blog: defineCollection({
      type: 'page',
      source: 'blog/**/*.md'
    })
  }
})
```

---

## Conclusion et checklist production

Pour un blog technique sur Nuxt 4.2.x SSG avec Cloudflare Pages, votre configuration est validée. Les points essentiels :

1. **Config dans `content.build.markdown.highlight`** — le chemin v2 est déprécié
2. **Dual-theme avec CSS variables** — compatible TailwindCSS 4 via classe `html.dark`
3. **Limitez `langs` au strict nécessaire** — impact direct sur le bundle
4. **`prerender: true` sur `/blog/**`** — highlighting au build, zéro JS client
5. **Renommez `ProseCode` → `ProsePre`** — breaking change critique si vous avez des composants custom
6. **Thèmes high-contrast** — `github-*-high-contrast` pour l'accessibilité

La syntaxe MDC (`{1,3-5}`, `[filename]`, `::code-group`) reste stable entre les versions. Les transformers `@shikijs/transformers` s'exécutent au build sans overhead client.