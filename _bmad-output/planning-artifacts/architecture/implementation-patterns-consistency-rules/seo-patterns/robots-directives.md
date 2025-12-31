# Robots Directives

## Configuration via routeRules

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    // Pages à ne pas indexer
    '/admin/**': { robots: 'noindex, nofollow' },
    '/preview/**': { robots: 'noindex' },
    '/merci': { robots: 'noindex' },           // Page confirmation
    '/recherche': { robots: 'noindex' },       // Recherche interne

    // Pages paginées
    '/blog/page/**': { robots: 'noindex, follow' },
  }
})
```

## Directive dynamique par page

```typescript
<script setup lang="ts">
const isDraft = computed(() => post.value?.draft === true)

useSeoMeta({
  robots: () => isDraft.value ? 'noindex, nofollow' : 'index, follow'
})
</script>
```

## robots.txt vs meta robots

| Aspect | robots.txt | Meta robots |
|--------|------------|-------------|
| **Fonction** | Bloque le crawl | Bloque l'indexation |
| **Portée** | Patterns URL | Page spécifique |
| **Si bloqué par robots.txt** | Google ne voit PAS le meta robots | — |
| **Usage SEO** | Économiser le crawl budget | Contrôler l'indexation |

**Point critique** : Si vous bloquez une page via `robots.txt`, Google ne peut pas lire la balise `<meta name="robots">`, donc `noindex` sera ignoré.

```
# public/robots.txt - Configuration SSG
User-agent: *
Allow: /

# Ne pas bloquer les pages que vous voulez noindex
# (Google doit pouvoir les crawler pour voir le meta noindex)
Disallow: /api/
Disallow: /_nuxt/

Sitemap: https://sebc.dev/sitemap.xml
```

## Crawlers AI dans robots.txt

Les principaux crawlers AI respectent robots.txt. Configuration pour contrôler l'indexation vs l'entraînement :

```
# public/robots.txt - Ajouts pour crawlers AI

# Autoriser les crawlers de recherche AI (citations dans réponses)
User-agent: OAI-SearchBot
User-agent: ChatGPT-User
User-agent: ClaudeBot
User-agent: PerplexityBot
Allow: /

# Optionnel: bloquer l'entraînement des modèles
User-agent: GPTBot
User-agent: anthropic-ai
User-agent: Google-Extended
Disallow: /
```

**User-agents AI à connaître :**

| User-agent | Propriétaire | Fonction |
|------------|--------------|----------|
| `GPTBot` | OpenAI | Entraînement modèles GPT |
| `OAI-SearchBot` | OpenAI | ChatGPT Search (citations) |
| `ChatGPT-User` | OpenAI | Plugins/browsing ChatGPT |
| `ClaudeBot` | Anthropic | Citations Claude |
| `anthropic-ai` | Anthropic | Entraînement Claude |
| `PerplexityBot` | Perplexity | Index Perplexity AI |
| `Google-Extended` | Google | Contrôle Gemini/Bard |

**Note importante** : Bloquer `GPTBot` empêche l'entraînement mais n'affecte pas les citations dans ChatGPT Search (géré par `OAI-SearchBot`).
