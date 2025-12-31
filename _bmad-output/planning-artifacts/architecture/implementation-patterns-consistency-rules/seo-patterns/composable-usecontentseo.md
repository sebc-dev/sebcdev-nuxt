# Composable useContentSeo()

Composable réutilisable pour les pages de contenu avec fallbacks automatiques.

```typescript
// app/composables/useContentSeo.ts
interface ContentSeoOptions {
  defaultDescription?: string
  defaultImage?: string
  truncateAt?: number
}

export function useContentSeo(
  content: Ref<any>,
  options: ContentSeoOptions = {}
) {
  const siteConfig = useSiteConfig()
  const route = useRoute()

  const {
    defaultDescription = 'Blog technique sur le développement web',
    defaultImage = '/og-default.jpg',
    truncateAt = 155
  } = options

  const siteUrl = siteConfig.url

  const description = computed(() => {
    let desc = content.value?.description
      ?? content.value?.excerpt
      ?? defaultDescription

    if (desc.length > truncateAt) {
      desc = `${desc.slice(0, truncateAt - 3)}...`
    }
    return desc
  })

  const image = computed(() => {
    const img = content.value?.image ?? content.value?.ogImage
    if (!img) return `${siteUrl}${defaultImage}`
    return img.startsWith('http') ? img : `${siteUrl}${img}`
  })

  const canonical = computed(() =>
    content.value?.canonical ?? `${siteUrl}${route.path}`
  )

  return { description, image, canonical }
}
```

**Usage :**

```typescript
<script setup lang="ts">
const { data: post } = await useAsyncData(...)
const { description, image, canonical } = useContentSeo(post)

useSeoMeta({
  description,
  ogImage: image,
  ogUrl: canonical
})
</script>
```
