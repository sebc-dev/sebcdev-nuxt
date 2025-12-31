# BreadcrumbList Schema

## Composable useBreadcrumbSchema()

Génération automatique du breadcrumb basée sur la route actuelle.

```typescript
// app/composables/useBreadcrumbSchema.ts
export function useBreadcrumbSchema() {
  const route = useRoute()
  const { t } = useI18n()

  const segments = route.path.split('/').filter(Boolean)
  const items = [
    { name: t('nav.home'), item: '/' }
  ]

  let currentPath = ''
  segments.forEach((segment, index) => {
    currentPath += `/${segment}`
    const isLast = index === segments.length - 1

    items.push({
      name: formatSegmentName(segment),
      // Pas de 'item' pour le dernier élément (convention Google)
      ...(isLast ? {} : { item: currentPath })
    })
  })

  useSchemaOrg([
    defineBreadcrumb({
      itemListElement: items
    })
  ])
}

function formatSegmentName(segment: string): string {
  // Convertit "mon-article" en "Mon article"
  return segment
    .replace(/-/g, ' ')
    .replace(/\b\w/g, (c) => c.toUpperCase())
}
```

## Usage dans une page article

```typescript
<script setup lang="ts">
// Breadcrumb manuel pour contrôle total
useSchemaOrg([
  defineBreadcrumb({
    itemListElement: [
      { name: 'Accueil', item: '/' },
      { name: 'Blog', item: '/blog' },
      { name: article.value?.title }  // Pas d'item = page courante
    ]
  })
])
</script>
```

**Exigences Google** : Au minimum **deux ListItems** pour l'éligibilité aux rich results. La propriété `position` est auto-calculée par le module.
