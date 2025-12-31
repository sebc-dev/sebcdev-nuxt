# Server Components Patterns

**Fichiers `.server.vue` - zéro JavaScript client:**

```vue
<!-- components/BlogContent.server.vue -->
<script setup>
import MarkdownIt from 'markdown-it'
import Prism from 'prismjs'

const props = defineProps<{
  content: string
}>()

const md = new MarkdownIt({
  highlight: (str, lang) => {
    if (lang && Prism.languages[lang]) {
      return Prism.highlight(str, Prism.languages[lang], lang)
    }
    return ''
  }
})

const renderedContent = md.render(props.content)
</script>

<template>
  <article class="prose" v-html="renderedContent" />
</template>
```

**Cas d'usage idéaux pour Server Components:**
- Rendu Markdown avec syntax highlighting
- Contenu statique (footer, header, navigation)
- Calculs lourds côté serveur
- Contenu SEO-critique sans interactivité
