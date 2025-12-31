# Story 2.2: Reading Time & Publication Date

Status: ready-for-dev

## Story

As a visiteur,
I want to see the estimated reading time and publication date,
So that I can decide if I have time to read and know how recent the content is.

## Acceptance Criteria

1. **AC1 - Temps de lecture affiché** : Je vois le temps de lecture estimé au format "X min de lecture" (FR) / "X min read" (EN)
2. **AC2 - Calcul temps de lecture** : Le calcul est basé sur 200 mots/minute, arrondi à la minute supérieure (minimum 1 min)
3. **AC3 - Date de publication** : Je vois la date de publication au format "15 janvier 2025" (FR) / "January 15, 2025" (EN)
4. **AC4 - Localisation date** : La date est localisée selon la langue active via `Intl.DateTimeFormat`
5. **AC5 - Positionnement** : Le temps de lecture et la date sont affichés dans le header de l'article, sous les badges (story 2-1)
6. **AC6 - Accessibilité** : Les éléments utilisent des balises sémantiques (`<time>` avec `datetime`)

## Tasks / Subtasks

- [ ] **Task 1: Créer le composable useReadingTime** (AC: #1, #2)
  - [ ] 1.1 Créer `app/composables/useReadingTime.ts`
  - [ ] 1.2 Implémenter le calcul: `Math.ceil(wordCount / 200)`
  - [ ] 1.3 Gérer le cas minimum (1 min pour articles très courts)
  - [ ] 1.4 Exporter une fonction qui accepte le contenu texte ou le wordCount

- [ ] **Task 2: Créer l'utilitaire formatDate** (AC: #3, #4)
  - [ ] 2.1 Créer `app/utils/formatDate.ts`
  - [ ] 2.2 Utiliser `Intl.DateTimeFormat` avec options: `{ day: 'numeric', month: 'long', year: 'numeric' }`
  - [ ] 2.3 Supporter les locales 'fr-FR' et 'en-US'
  - [ ] 2.4 Parser la date ISO depuis le frontmatter (`publishedAt`)

- [ ] **Task 3: Créer le composant ArticleTimeMeta.vue** (AC: #1-6)
  - [ ] 3.1 Créer `app/components/content/ArticleTimeMeta.vue`
  - [ ] 3.2 Définir les props:
    ```typescript
    interface Props {
      publishedAt: string  // ISO date from frontmatter
      readingTime: number  // minutes
      updatedAt?: string   // optional ISO date
    }
    ```
  - [ ] 3.3 Utiliser `<time datetime="...">` pour la sémantique
  - [ ] 3.4 Afficher l'icône horloge (optionnel) et l'icône calendrier
  - [ ] 3.5 Ajouter les traductions pour le format temps de lecture

- [ ] **Task 4: Intégrer dans ArticleHeader.vue** (AC: #5)
  - [ ] 4.1 Modifier `app/components/content/ArticleHeader.vue` (créé en 2-1)
  - [ ] 4.2 Ajouter `ArticleTimeMeta` sous les badges `ArticleMeta`
  - [ ] 4.3 Calculer le readingTime depuis le body de l'article
  - [ ] 4.4 Passer publishedAt depuis le frontmatter

- [ ] **Task 5: Utiliser dans la page [slug].vue** (AC: #1-6)
  - [ ] 5.1 Modifier `app/pages/articles/[slug].vue`
  - [ ] 5.2 Extraire `publishedAt` et calculer `readingTime`
  - [ ] 5.3 Passer les props à ArticleHeader

- [ ] **Task 6: Ajouter les traductions** (AC: #1, #3)
  - [ ] 6.1 Ajouter dans `i18n/locales/fr.json`:
    ```json
    {
      "article": {
        "readingTime": "{minutes} min de lecture",
        "publishedOn": "Publié le {date}",
        "updatedOn": "Mis à jour le {date}"
      }
    }
    ```
  - [ ] 6.2 Ajouter dans `i18n/locales/en.json`:
    ```json
    {
      "article": {
        "readingTime": "{minutes} min read",
        "publishedOn": "Published on {date}",
        "updatedOn": "Updated on {date}"
      }
    }
    ```

- [ ] **Task 7: Tests et validation** (AC: #1-6)
  - [ ] 7.1 Tester avec articles de différentes longueurs
  - [ ] 7.2 Vérifier le formatage des dates en FR et EN
  - [ ] 7.3 Vérifier l'attribut `datetime` sur les éléments `<time>`
  - [ ] 7.4 Tester le switch de langue (date se reformate)

## Dev Notes

### Critical Architecture Requirements

**Calcul du temps de lecture:**
- Vitesse standard: 200 mots/minute
- Arrondi supérieur: `Math.ceil()`
- Minimum: 1 minute
- Le wordCount peut être fourni par Nuxt Content ou calculé manuellement

**Formatage de date - Zéro dépendance:**
```typescript
// ✅ Correct - Intl.DateTimeFormat natif
const formatDate = (isoDate: string, locale: string): string => {
  const date = new Date(isoDate)
  return new Intl.DateTimeFormat(locale, {
    day: 'numeric',
    month: 'long',
    year: 'numeric'
  }).format(date)
}

// Résultats:
// formatDate('2025-01-15', 'fr-FR') → "15 janvier 2025"
// formatDate('2025-01-15', 'en-US') → "January 15, 2025"
```

**Structure frontmatter attendue:**
```yaml
---
title: "Mon article"
publishedAt: "2025-01-15"      # ISO 8601 obligatoire
updatedAt: "2025-01-20"        # Optionnel
---
```

### Project Structure Notes

Fichiers à créer/modifier:

```
app/
├── composables/
│   └── useReadingTime.ts       # NOUVEAU
├── utils/
│   └── formatDate.ts           # NOUVEAU
├── components/
│   └── content/
│       ├── ArticleTimeMeta.vue # NOUVEAU
│       └── ArticleHeader.vue   # MODIFIER (ajouter ArticleTimeMeta)
├── pages/
│   └── articles/
│       └── [slug].vue          # MODIFIER (passer readingTime)
i18n/
└── locales/
    ├── fr.json                 # MODIFIER (ajouter article.*)
    └── en.json                 # MODIFIER (ajouter article.*)
```

### Technical Implementation Details

**useReadingTime.ts:**
```typescript
// app/composables/useReadingTime.ts
const WORDS_PER_MINUTE = 200

export function useReadingTime(content: string): number {
  // Compte les mots en supprimant le HTML/markdown
  const text = content
    .replace(/<[^>]*>/g, '')     // Remove HTML tags
    .replace(/```[\s\S]*?```/g, '') // Remove code blocks
    .replace(/`[^`]*`/g, '')     // Remove inline code
    .replace(/\[.*?\]\(.*?\)/g, '') // Remove markdown links
    .trim()

  const wordCount = text.split(/\s+/).filter(Boolean).length
  return Math.max(1, Math.ceil(wordCount / WORDS_PER_MINUTE))
}

// Alternative si Nuxt Content fournit déjà le wordCount
export function calculateReadingTime(wordCount: number): number {
  return Math.max(1, Math.ceil(wordCount / WORDS_PER_MINUTE))
}
```

**formatDate.ts:**
```typescript
// app/utils/formatDate.ts
export function formatDate(isoDate: string, locale: string = 'fr-FR'): string {
  const date = new Date(isoDate)

  // Validation
  if (isNaN(date.getTime())) {
    console.warn(`Invalid date: ${isoDate}`)
    return isoDate
  }

  return new Intl.DateTimeFormat(locale, {
    day: 'numeric',
    month: 'long',
    year: 'numeric'
  }).format(date)
}

// Pour l'attribut datetime (toujours ISO)
export function toISODate(isoDate: string): string {
  return new Date(isoDate).toISOString().split('T')[0]
}
```

**ArticleTimeMeta.vue:**
```vue
<script setup lang="ts">
import { formatDate } from '~/utils/formatDate'

interface Props {
  publishedAt: string
  readingTime: number
  updatedAt?: string
}

const props = defineProps<Props>()
const { t, locale } = useI18n()

const formattedPublishDate = computed(() =>
  formatDate(props.publishedAt, locale.value)
)

const formattedUpdateDate = computed(() =>
  props.updatedAt ? formatDate(props.updatedAt, locale.value) : null
)
</script>

<template>
  <div class="flex items-center gap-4 text-sm text-foreground-muted">
    <!-- Temps de lecture -->
    <span class="flex items-center gap-1">
      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <circle cx="12" cy="12" r="10" stroke-width="2"/>
        <path d="M12 6v6l4 2" stroke-width="2" stroke-linecap="round"/>
      </svg>
      {{ t('article.readingTime', { minutes: readingTime }) }}
    </span>

    <!-- Séparateur -->
    <span class="text-border">•</span>

    <!-- Date de publication -->
    <time :datetime="publishedAt" class="flex items-center gap-1">
      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <rect x="3" y="4" width="18" height="18" rx="2" stroke-width="2"/>
        <path d="M16 2v4M8 2v4M3 10h18" stroke-width="2" stroke-linecap="round"/>
      </svg>
      {{ formattedPublishDate }}
    </time>

    <!-- Date de mise à jour (optionnel) -->
    <template v-if="formattedUpdateDate">
      <span class="text-border">•</span>
      <time :datetime="updatedAt" class="text-foreground-muted/70">
        {{ t('article.updatedOn', { date: formattedUpdateDate }) }}
      </time>
    </template>
  </div>
</template>
```

**Intégration dans ArticleHeader.vue:**
```vue
<script setup lang="ts">
import ArticleMeta from './ArticleMeta.vue'
import ArticleTimeMeta from './ArticleTimeMeta.vue'

interface Props {
  title: string
  pillar: 'ai' | 'engineering' | 'ux'
  category: string
  level: string
  tags: string[]
  publishedAt: string
  readingTime: number
  updatedAt?: string
}

defineProps<Props>()
</script>

<template>
  <header class="mb-8">
    <!-- Badges (Story 2-1) -->
    <ArticleMeta
      :pillar="pillar"
      :category="category"
      :level="level"
      :tags="tags"
      class="mb-4"
    />

    <!-- Titre -->
    <h1 class="text-4xl font-bold text-foreground mb-4">
      {{ title }}
    </h1>

    <!-- Temps de lecture + Date (Story 2-2) -->
    <ArticleTimeMeta
      :published-at="publishedAt"
      :reading-time="readingTime"
      :updated-at="updatedAt"
    />
  </header>
</template>
```

### Package Versions (Décembre 2025)

| Package | Version |
|---------|---------|
| nuxt | 4.2.2 |
| @nuxtjs/i18n | 9.x ou 10.x |

### Dépendances de cette story

**Requiert (stories précédentes):**
- Story 1-1 : Structure projet, article de test avec frontmatter
- Story 2-1 : ArticleHeader.vue créé avec ArticleMeta

**Bloque (stories futures):**
- Aucune dépendance directe

### Anti-patterns à éviter

- ❌ Ne PAS utiliser `dayjs`, `date-fns` ou `moment` - `Intl.DateTimeFormat` suffit
- ❌ Ne PAS hardcoder les formats de date - utiliser les locales
- ❌ Ne PAS oublier l'attribut `datetime` sur les éléments `<time>`
- ❌ Ne PAS calculer le temps de lecture côté client à chaque render - le calculer une fois
- ❌ Ne PAS utiliser des icônes lourdes - SVG inline simples ou Lucide icons

### Considérations accessibilité

- Utiliser `<time datetime="2025-01-15">` pour la sémantique
- Le format ISO dans `datetime` permet aux lecteurs d'écran de comprendre la date
- Texte alternatif clair pour les icônes si non décoratives

### Nuxt Content - Extraction wordCount

Nuxt Content 3 peut fournir automatiquement le wordCount via les meta du document :

```typescript
// Dans [slug].vue
const { data: article } = await useAsyncData(
  `article-${slug}`,
  () => queryCollection('articles_fr')
    .where('slug', '=', slug)
    .first()
)

// article.meta.readingTime ou article.meta.wordCount selon config
// Sinon, calculer manuellement depuis article.body
```

### References

- [Source: architecture/implementation-patterns-consistency-rules/format-patterns.md] - Formatage dates Intl
- [Source: architecture/project-structure-boundaries.md#composables] - Structure composables
- [Source: epics/epic-2-rich-article-experience.md#Story 2.2] - Requirements originaux
- [Source: ux-design-specification/visual-design-foundation.md#Typography] - Styles texte
- [Source: architecture/core-architectural-decisions/frontend-architecture.md] - Organisation composants

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

### File List
