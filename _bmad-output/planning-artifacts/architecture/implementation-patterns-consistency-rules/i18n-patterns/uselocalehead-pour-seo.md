# useLocaleHead() pour SEO

Injection automatique des meta tags hreflang :

```vue
<!-- app/layouts/default.vue -->
<script setup lang="ts">
const head = useLocaleHead({
  addSeoAttributes: true,
  addDirAttribute: true,
  seo: {
    // Préserve certains query params dans l'URL canonique (optionnel)
    canonicalQueries: ['ref', 'utm_source']
  }
})
</script>

<template>
  <Html :lang="head.htmlAttrs.lang" :dir="head.htmlAttrs.dir">
    <Head>
      <template v-for="link in head.link" :key="link.hid">
        <Link
          :rel="link.rel"
          :href="link.href"
          :hreflang="link.hreflang"
        />
      </template>
      <template v-for="meta in head.meta" :key="meta.hid">
        <Meta :property="meta.property" :content="meta.content" />
      </template>
    </Head>
    <Body>
      <slot />
    </Body>
  </Html>
</template>
```

**Génère automatiquement :**
- `<link rel="alternate" hreflang="en" href="https://sebc.dev/blog/article" />`
- `<link rel="alternate" hreflang="fr" href="https://sebc.dev/fr/blog/article" />`
- `<link rel="alternate" hreflang="x-default" href="https://sebc.dev/blog/article" />`
- `<meta property="og:locale" content="en_US" />`
- `<meta property="og:locale:alternate" content="fr_FR" />`
