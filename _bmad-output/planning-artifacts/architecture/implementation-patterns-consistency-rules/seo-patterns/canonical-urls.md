# Canonical URLs

## Configuration SSG avec Cloudflare

Chaque page doit avoir une **URL canonique absolue unique**. Cloudflare redirige par défaut sans trailing slash.

```typescript
<script setup lang="ts">
const siteConfig = useSiteConfig()
const route = useRoute()

// URL canonique absolue (sans trailing slash pour cohérence Cloudflare)
const canonicalUrl = computed(() => {
  const path = route.path.toLowerCase()
  // Normaliser sans trailing slash (comportement par défaut Cloudflare)
  const normalizedPath = path.endsWith('/') && path !== '/'
    ? path.slice(0, -1)
    : path
  return `${siteConfig.url}${normalizedPath}`
})

useHead({
  link: [{ rel: 'canonical', href: canonicalUrl }]
})

useSeoMeta({
  ogUrl: canonicalUrl
})
</script>
```

## Erreurs canoniques courantes

```typescript
// ❌ MAUVAIS : URL relative
useHead({
  link: [{ rel: 'canonical', href: '/ma-page' }]
})

// ❌ MAUVAIS : Canonical vers une page noindex
useSeoMeta({ robots: 'noindex' })
useHead({
  link: [{ rel: 'canonical', href: 'https://sebc.dev/page/' }]
})
// Signaux contradictoires !

// ✅ BON : URL absolue
useHead({
  link: [{ rel: 'canonical', href: 'https://sebc.dev/page' }]
})
```

## Canonical cross-language (CRITIQUE)

**Règle absolue** : Chaque version linguistique DOIT avoir un canonical **auto-référencé**. Ne JAMAIS pointer le canonical vers une autre langue — cela **invalide tous les signaux hreflang**.

```html
<!-- Sur /fr/page - CORRECT -->
<link rel="canonical" href="https://example.com/fr/page" />
<link rel="alternate" hreflang="fr-FR" href="https://example.com/fr/page" />
<link rel="alternate" hreflang="en-US" href="https://example.com/page" />

<!-- Sur /fr/page - INCORRECT (invalide tout le hreflang) -->
<link rel="canonical" href="https://example.com/page" />  <!-- ❌ Pointe vers EN -->
```

| Erreur | Conséquence |
|--------|-------------|
| Canonical FR → EN | Google ignore tous les hreflang |
| Canonical EN → FR | Google ignore tous les hreflang |
| Canonical auto-référencé | ✅ hreflang fonctionne correctement |

**Note importante** : Le contenu traduit n'est **PAS** considéré comme dupliqué par Google. Seul le contenu identique non traduit pose problème.
