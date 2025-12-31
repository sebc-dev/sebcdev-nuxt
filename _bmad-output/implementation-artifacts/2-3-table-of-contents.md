# Story 2.3: Table of Contents

Status: ready-for-dev

## Story

As a visiteur,
I want to see a table of contents generated from article headings,
So that I can navigate quickly to specific sections.

## Acceptance Criteria

1. **AC1 - Génération automatique** : La ToC est générée automatiquement à partir des titres H2 et H3 de l'article
2. **AC2 - Desktop sidebar sticky** : Sur desktop (>=1024px), la ToC est affichée en sidebar sticky (240px largeur) visible pendant le scroll
3. **AC3 - Mobile accessible** : Sur mobile (<1024px), la ToC est accessible via un bouton flottant qui ouvre un Sheet/Modal
4. **AC4 - Navigation smooth** : Au clic sur un item, la page scroll smooth vers la section correspondante
5. **AC5 - Section active** : La section actuellement visible est mise en évidence dans la ToC pendant la lecture
6. **AC6 - Accessibilité** : Navigation clavier possible, `aria-current` sur l'item actif, landmark `<nav>`

## Tasks / Subtasks

- [ ] **Task 1: Créer le composable useTableOfContents** (AC: #5)
  - [ ] 1.1 Créer `app/composables/useTableOfContents.ts`
  - [ ] 1.2 Implémenter IntersectionObserver pour détecter la section visible
  - [ ] 1.3 Gérer le threshold (ex: 0.2 ou rootMargin pour offset header)
  - [ ] 1.4 Retourner `activeId` réactif

- [ ] **Task 2: Créer les composants Prose pour ancres** (AC: #1, #4)
  - [ ] 2.1 Créer `app/components/prose/ProseH2.vue` avec id et ancre
  - [ ] 2.2 Créer `app/components/prose/ProseH3.vue` avec id et ancre
  - [ ] 2.3 Générer les IDs slugifiés à partir du texte du heading
  - [ ] 2.4 Ajouter un lien d'ancre au hover (optionnel)

- [ ] **Task 3: Créer le composant TableOfContents.vue** (AC: #1-6)
  - [ ] 3.1 Créer `app/components/content/TableOfContents.vue`
  - [ ] 3.2 Définir les props:
    ```typescript
    interface TocItem {
      id: string
      text: string
      level: 2 | 3
    }
    interface Props {
      items: TocItem[]
    }
    ```
  - [ ] 3.3 Afficher la liste avec indentation pour H3
  - [ ] 3.4 Implémenter le highlight de l'item actif
  - [ ] 3.5 Ajouter `scroll-behavior: smooth` au clic
  - [ ] 3.6 Wrapper dans `<nav aria-label="Table des matières">`

- [ ] **Task 4: Créer le composant TocMobile.vue** (AC: #3)
  - [ ] 4.1 Créer `app/components/content/TocMobile.vue`
  - [ ] 4.2 Utiliser shadcn-vue Sheet pour le drawer
  - [ ] 4.3 Ajouter un bouton flottant (FAB) en bas à droite
  - [ ] 4.4 Fermer le sheet après navigation

- [ ] **Task 5: Créer le layout article.vue** (AC: #2, #3)
  - [ ] 5.1 Créer `app/layouts/article.vue`
  - [ ] 5.2 Implémenter la grille: contenu (720px max) + sidebar ToC (240px)
  - [ ] 5.3 Sidebar sticky avec `position: sticky; top: 80px`
  - [ ] 5.4 Responsive: cacher sidebar desktop sur mobile, afficher TocMobile

- [ ] **Task 6: Intégrer dans la page [slug].vue** (AC: #1-6)
  - [ ] 6.1 Modifier `app/pages/articles/[slug].vue`
  - [ ] 6.2 Extraire les headings depuis l'article (Nuxt Content toc)
  - [ ] 6.3 Passer les items au composant ToC
  - [ ] 6.4 Utiliser le layout `article`

- [ ] **Task 7: Ajouter les traductions** (AC: #3, #6)
  - [ ] 7.1 Ajouter dans `i18n/locales/fr.json`:
    ```json
    {
      "toc": {
        "title": "Sommaire",
        "openButton": "Voir le sommaire"
      }
    }
    ```
  - [ ] 7.2 Ajouter les équivalents en anglais

- [ ] **Task 8: Tests et validation** (AC: #1-6)
  - [ ] 8.1 Tester avec un article ayant plusieurs H2 et H3
  - [ ] 8.2 Vérifier le scroll smooth et le highlight actif
  - [ ] 8.3 Tester le responsive (desktop sidebar vs mobile FAB)
  - [ ] 8.4 Tester la navigation clavier

## Dev Notes

### Critical Architecture Requirements

**Layout Article avec Sidebar ToC:**
```
┌─────────────────────────────────────────────────────────────┐
│  Header (fixed)                                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────────────────────┐  ┌─────────────────────────┐ │
│   │                          │  │  Table of Contents      │ │
│   │    Article Content       │  │  (sticky sidebar)       │ │
│   │    max-width: 720px      │  │  width: 240px           │ │
│   │                          │  │                         │ │
│   │                          │  │  • Section 1 (active)   │ │
│   │                          │  │    ◦ Subsection 1.1     │ │
│   │                          │  │    ◦ Subsection 1.2     │ │
│   │                          │  │  • Section 2            │ │
│   │                          │  │  • Section 3            │ │
│   │                          │  │                         │ │
│   └──────────────────────────┘  └─────────────────────────┘ │
│         gap: 48px                                            │
├─────────────────────────────────────────────────────────────┤
│  Footer                                                      │
└─────────────────────────────────────────────────────────────┘
```

**Extraction des headings - Nuxt Content:**
```typescript
// Nuxt Content 3 fournit article.body.toc automatiquement
const { data: article } = await useAsyncData(...)

// Structure toc:
interface TocLink {
  id: string
  text: string
  depth: number  // 2 pour H2, 3 pour H3
  children?: TocLink[]
}

// article.body.toc.links contient les headings
```

### Project Structure Notes

Fichiers à créer/modifier:

```
app/
├── composables/
│   └── useTableOfContents.ts     # NOUVEAU
├── components/
│   ├── content/
│   │   ├── TableOfContents.vue   # NOUVEAU
│   │   └── TocMobile.vue         # NOUVEAU
│   └── prose/
│       ├── ProseH2.vue           # NOUVEAU
│       └── ProseH3.vue           # NOUVEAU
├── layouts/
│   └── article.vue               # NOUVEAU
├── pages/
│   └── articles/
│       └── [slug].vue            # MODIFIER (layout + ToC)
i18n/
└── locales/
    ├── fr.json                   # MODIFIER
    └── en.json                   # MODIFIER
```

### Technical Implementation Details

**useTableOfContents.ts:**
```typescript
// app/composables/useTableOfContents.ts
export function useTableOfContents(headingIds: Ref<string[]>) {
  const activeId = ref<string | null>(null)

  onMounted(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            activeId.value = entry.target.id
          }
        })
      },
      {
        rootMargin: '-80px 0px -70% 0px', // Offset pour header fixe
        threshold: 0
      }
    )

    headingIds.value.forEach((id) => {
      const el = document.getElementById(id)
      if (el) observer.observe(el)
    })

    onUnmounted(() => observer.disconnect())
  })

  return { activeId }
}
```

**ProseH2.vue:**
```vue
<script setup lang="ts">
const props = defineProps<{ id?: string }>()
const slots = useSlots()

// Générer l'ID depuis le contenu si non fourni
const headingId = computed(() => {
  if (props.id) return props.id
  // Nuxt Content génère déjà les IDs
  return undefined
})
</script>

<template>
  <h2 :id="headingId" class="group relative scroll-mt-20">
    <slot />
    <a
      v-if="headingId"
      :href="`#${headingId}`"
      class="absolute -left-6 opacity-0 group-hover:opacity-100 text-foreground-muted"
      aria-hidden="true"
    >
      #
    </a>
  </h2>
</template>
```

**TableOfContents.vue:**
```vue
<script setup lang="ts">
interface TocItem {
  id: string
  text: string
  depth: number
  children?: TocItem[]
}

interface Props {
  items: TocItem[]
  activeId?: string | null
}

const props = defineProps<Props>()
const { t } = useI18n()

function scrollToSection(id: string) {
  const el = document.getElementById(id)
  if (el) {
    el.scrollIntoView({ behavior: 'smooth', block: 'start' })
  }
}
</script>

<template>
  <nav aria-label="Table des matières" class="text-sm">
    <h2 class="font-semibold text-foreground mb-4">
      {{ t('toc.title') }}
    </h2>

    <ul class="space-y-2">
      <li v-for="item in items" :key="item.id">
        <a
          :href="`#${item.id}`"
          :aria-current="activeId === item.id ? 'true' : undefined"
          class="block py-1 transition-colors"
          :class="[
            activeId === item.id
              ? 'text-accent font-medium'
              : 'text-foreground-muted hover:text-foreground'
          ]"
          @click.prevent="scrollToSection(item.id)"
        >
          {{ item.text }}
        </a>

        <!-- Nested H3 items -->
        <ul v-if="item.children?.length" class="ml-4 mt-1 space-y-1">
          <li v-for="child in item.children" :key="child.id">
            <a
              :href="`#${child.id}`"
              :aria-current="activeId === child.id ? 'true' : undefined"
              class="block py-1 text-xs transition-colors"
              :class="[
                activeId === child.id
                  ? 'text-accent font-medium'
                  : 'text-foreground-muted hover:text-foreground'
              ]"
              @click.prevent="scrollToSection(child.id)"
            >
              {{ child.text }}
            </a>
          </li>
        </ul>
      </li>
    </ul>
  </nav>
</template>
```

**TocMobile.vue:**
```vue
<script setup lang="ts">
import { Sheet, SheetContent, SheetTrigger, SheetHeader, SheetTitle } from '@/components/ui/sheet'
import { Button } from '@/components/ui/button'
import TableOfContents from './TableOfContents.vue'

interface TocItem {
  id: string
  text: string
  depth: number
  children?: TocItem[]
}

interface Props {
  items: TocItem[]
  activeId?: string | null
}

defineProps<Props>()
const { t } = useI18n()

const isOpen = ref(false)

function handleNavigation() {
  // Fermer le sheet après navigation
  setTimeout(() => {
    isOpen.value = false
  }, 100)
}
</script>

<template>
  <Sheet v-model:open="isOpen">
    <SheetTrigger as-child>
      <Button
        variant="secondary"
        size="icon"
        class="fixed bottom-6 right-6 z-40 rounded-full shadow-lg lg:hidden"
        :aria-label="t('toc.openButton')"
      >
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M4 6h16M4 12h16M4 18h7" />
        </svg>
      </Button>
    </SheetTrigger>

    <SheetContent side="right" class="w-80">
      <SheetHeader>
        <SheetTitle>{{ t('toc.title') }}</SheetTitle>
      </SheetHeader>

      <div class="mt-6" @click="handleNavigation">
        <TableOfContents :items="items" :active-id="activeId" />
      </div>
    </SheetContent>
  </Sheet>
</template>
```

**Layout article.vue:**
```vue
<script setup lang="ts">
// Layout pour les pages article avec sidebar ToC
</script>

<template>
  <div class="min-h-screen flex flex-col">
    <!-- Header sera dans default.vue ou app.vue -->

    <main class="flex-1 container mx-auto px-4 py-8">
      <div class="lg:grid lg:grid-cols-[1fr_240px] lg:gap-12">
        <!-- Contenu article -->
        <article class="max-w-[720px]">
          <slot />
        </article>

        <!-- Sidebar ToC (desktop only) -->
        <aside class="hidden lg:block">
          <div class="sticky top-24">
            <slot name="toc" />
          </div>
        </aside>
      </div>
    </main>

    <!-- Footer sera dans default.vue ou app.vue -->
  </div>
</template>
```

**Intégration dans [slug].vue:**
```vue
<script setup lang="ts">
definePageMeta({
  layout: 'article'
})

const route = useRoute()
const slug = route.params.slug as string

const { data: article } = await useAsyncData(...)

// Extraire les headings pour la ToC
const tocItems = computed(() => {
  return article.value?.body?.toc?.links ?? []
})

// IDs pour l'IntersectionObserver
const headingIds = computed(() => {
  const ids: string[] = []
  function extractIds(items: any[]) {
    items.forEach(item => {
      ids.push(item.id)
      if (item.children) extractIds(item.children)
    })
  }
  extractIds(tocItems.value)
  return ids
})

const { activeId } = useTableOfContents(headingIds)
</script>

<template>
  <div>
    <!-- Contenu article via slot par défaut -->
    <ArticleHeader ... />
    <ContentRenderer :value="article" />

    <!-- ToC via slot nommé -->
    <template #toc>
      <TableOfContents :items="tocItems" :active-id="activeId" />
    </template>
  </div>

  <!-- Mobile ToC (toujours visible) -->
  <TocMobile :items="tocItems" :active-id="activeId" />
</template>
```

### Package Versions (Décembre 2025)

| Package | Version |
|---------|---------|
| nuxt | 4.2.2 |
| shadcn-vue | latest (pour Sheet) |

### Dépendances de cette story

**Requiert (stories précédentes):**
- Story 1-1 : Structure projet, article de test avec headings H2/H3
- Story 2-1 : ArticleHeader.vue
- Story 2-2 : Intégration dans header article

**Bloque (stories futures):**
- Story 2-4 : ReadingProgress peut utiliser le même layout

### Anti-patterns à éviter

- ❌ Ne PAS utiliser une lib externe pour l'IntersectionObserver - API native suffisante
- ❌ Ne PAS oublier `scroll-mt-20` sur les headings pour compenser le header fixe
- ❌ Ne PAS rendre la ToC trop large - 240px max pour laisser place au contenu
- ❌ Ne PAS utiliser `position: fixed` pour la sidebar - `sticky` suffit et reste dans le flow
- ❌ Ne PAS oublier de fermer le Sheet mobile après navigation

### Considérations accessibilité

- Wrapper dans `<nav aria-label="Table des matières">`
- `aria-current="true"` sur l'item actif
- Navigation clavier: Tab entre les liens
- Focus visible sur les items
- `scroll-behavior: smooth` en CSS global (ou via JS pour plus de contrôle)

### Considérations performance

- IntersectionObserver est plus performant que scroll listeners
- Throttle/debounce si nécessaire pour le calcul de section active
- Ne pas re-créer l'observer à chaque render

### CSS Global requis

```css
/* app/assets/css/main.css */
html {
  scroll-behavior: smooth;
}

/* Offset pour header fixe */
[id] {
  scroll-margin-top: 5rem; /* 80px */
}
```

### References

- [Source: architecture/project-structure-boundaries.md#Architectural Boundaries] - Layout avec sidebar
- [Source: ux-design-specification/component-strategy.md#TableOfContents] - Spécification composant
- [Source: epics/epic-2-rich-article-experience.md#Story 2.3] - Requirements originaux
- [Source: architecture/project-structure-boundaries.md#composables] - useTableOfContents.ts

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

### File List
