# Story 2.1: Article Metadata Badges

Status: ready-for-dev

## Story

As a visiteur,
I want to see the article's theme, category, tags, and level as visual badges,
So that I can quickly understand the article's context and relevance.

## Acceptance Criteria

1. **AC1 - Badge Thème/Pilier** : Je vois un badge thème avec la couleur du pilier (IA: violet `#8B5CF6`, Ingénierie: bleu `#3B82F6`, UX: rose `#EC4899`)
2. **AC2 - Badge Catégorie** : Je vois un badge catégorie (Actualité/Tutoriel/Décryptage/Étude de cas/Retour d'expérience)
3. **AC3 - Tags** : Je vois les tags associés sous forme de badges (style secondaire/outline)
4. **AC4 - Badge Niveau** : Je vois un badge niveau (Tous niveaux/Débutant/Intermédiaire/Avancé)
5. **AC5 - Positionnement** : Les badges sont positionnés dans le header de l'article, dans un ordre logique (Pilier → Catégorie → Niveau → Tags)
6. **AC6 - Accessibilité** : Les badges ont un contraste suffisant (WCAG AA 4.5:1) et des labels accessibles

## Tasks / Subtasks

- [ ] **Task 1: Installer et configurer shadcn-vue Badge** (AC: #1-5)
  - [ ] 1.1 Initialiser shadcn-vue avec `pnpm dlx shadcn-vue@latest init`
  - [ ] 1.2 Installer le composant Badge : `pnpm dlx shadcn-vue@latest add badge`
  - [ ] 1.3 Vérifier la génération dans `app/components/ui/badge/`
  - [ ] 1.4 Configurer les variants dans `components.json` si nécessaire

- [ ] **Task 2: Définir les couleurs piliers dans le design system** (AC: #1)
  - [ ] 2.1 Ajouter les CSS variables des piliers dans `app/assets/css/main.css`:
    ```css
    :root {
      --pillar-ai: #8B5CF6;
      --pillar-engineering: #3B82F6;
      --pillar-ux: #EC4899;
    }
    ```
  - [ ] 2.2 Créer les classes Tailwind personnalisées ou utiliser `style` inline pour les couleurs dynamiques
  - [ ] 2.3 Tester le contraste des couleurs sur fond `--background` (#0A0A0B)

- [ ] **Task 3: Créer le composant ArticleMeta.vue** (AC: #1-6)
  - [ ] 3.1 Créer `app/components/content/ArticleMeta.vue`
  - [ ] 3.2 Définir les props TypeScript:
    ```typescript
    interface Props {
      pillar: 'ai' | 'engineering' | 'ux'
      category: 'news' | 'tutorial' | 'deep-dive' | 'case-study' | 'retrospective'
      level: 'all' | 'beginner' | 'intermediate' | 'advanced'
      tags: string[]
    }
    ```
  - [ ] 3.3 Implémenter le mapping pillar → couleur et label traduit
  - [ ] 3.4 Implémenter le mapping category → label traduit (FR/EN)
  - [ ] 3.5 Implémenter le mapping level → label traduit
  - [ ] 3.6 Ajouter les attributs ARIA si nécessaire pour l'accessibilité

- [ ] **Task 4: Intégrer dans ArticleHeader.vue** (AC: #5)
  - [ ] 4.1 Créer ou mettre à jour `app/components/content/ArticleHeader.vue`
  - [ ] 4.2 Importer et utiliser `ArticleMeta` avec les données du frontmatter
  - [ ] 4.3 Positionner les badges au-dessus du titre H1
  - [ ] 4.4 Utiliser gap/flex pour l'espacement cohérent

- [ ] **Task 5: Utiliser dans la page [slug].vue** (AC: #1-6)
  - [ ] 5.1 Importer ArticleHeader dans `app/pages/articles/[slug].vue`
  - [ ] 5.2 Passer les métadonnées extraites du frontmatter
  - [ ] 5.3 Tester avec l'article de test créé en story 1-1

- [ ] **Task 6: Créer les traductions** (AC: #2, #4)
  - [ ] 6.1 Ajouter dans `i18n/locales/fr.json`:
    ```json
    {
      "pillars": { "ai": "IA", "engineering": "Ingénierie", "ux": "UX" },
      "categories": {
        "news": "Actualité",
        "tutorial": "Tutoriel",
        "deep-dive": "Décryptage",
        "case-study": "Étude de cas",
        "retrospective": "Retour d'expérience"
      },
      "levels": {
        "all": "Tous niveaux",
        "beginner": "Débutant",
        "intermediate": "Intermédiaire",
        "advanced": "Avancé"
      }
    }
    ```
  - [ ] 6.2 Ajouter les équivalents dans `i18n/locales/en.json`
  - [ ] 6.3 Utiliser `$t()` ou `useI18n()` dans les composants

- [ ] **Task 7: Tests et validation** (AC: #1-6)
  - [ ] 7.1 Vérifier visuellement les badges sur la page article
  - [ ] 7.2 Tester les 3 piliers avec couleurs différentes
  - [ ] 7.3 Tester tous les types de catégories
  - [ ] 7.4 Tester tous les niveaux
  - [ ] 7.5 Vérifier l'accessibilité avec axe DevTools

## Dev Notes

### Critical Architecture Requirements

**Composant Badge shadcn-vue:**
- Utiliser le variant `default` pour les piliers (couleur custom)
- Utiliser le variant `secondary` ou `outline` pour les tags
- Les badges doivent être non-interactifs (pas de onClick dans cette story - navigation ajoutée en Epic 3)

**Mapping des couleurs piliers:**
```typescript
const pillarColors: Record<string, string> = {
  ai: 'bg-[#8B5CF6] text-white',
  engineering: 'bg-[#3B82F6] text-white',
  ux: 'bg-[#EC4899] text-white',
}
```

**Structure des métadonnées (depuis frontmatter):**
```yaml
---
title: "Mon article"
pillar: ai           # ai | engineering | ux
category: tutorial   # news | tutorial | deep-dive | case-study | retrospective
level: intermediate  # all | beginner | intermediate | advanced
tags:
  - vue
  - typescript
  - composables
---
```

### Project Structure Notes

Fichiers à créer/modifier:

```
app/
├── components/
│   ├── content/
│   │   ├── ArticleMeta.vue       # NOUVEAU
│   │   └── ArticleHeader.vue     # NOUVEAU ou MODIFIER
│   └── ui/
│       └── badge/                # shadcn-vue (auto-généré)
├── assets/
│   └── css/
│       └── main.css              # MODIFIER (variables piliers)
├── pages/
│   └── articles/
│       └── [slug].vue            # MODIFIER (intégrer header)
i18n/
└── locales/
    ├── fr.json                   # CRÉER ou MODIFIER
    └── en.json                   # CRÉER ou MODIFIER
```

### Technical Implementation Details

**ArticleMeta.vue - Structure recommandée:**
```vue
<script setup lang="ts">
import { Badge } from '@/components/ui/badge'

interface Props {
  pillar: 'ai' | 'engineering' | 'ux'
  category: 'news' | 'tutorial' | 'deep-dive' | 'case-study' | 'retrospective'
  level: 'all' | 'beginner' | 'intermediate' | 'advanced'
  tags: string[]
}

const props = defineProps<Props>()
const { t } = useI18n()

const pillarColors: Record<string, string> = {
  ai: 'bg-[#8B5CF6] hover:bg-[#7C3AED]',
  engineering: 'bg-[#3B82F6] hover:bg-[#2563EB]',
  ux: 'bg-[#EC4899] hover:bg-[#DB2777]',
}
</script>

<template>
  <div class="flex flex-wrap items-center gap-2">
    <!-- Pilier -->
    <Badge :class="pillarColors[pillar]" class="text-white">
      {{ t(`pillars.${pillar}`) }}
    </Badge>

    <!-- Catégorie -->
    <Badge variant="secondary">
      {{ t(`categories.${category}`) }}
    </Badge>

    <!-- Niveau -->
    <Badge variant="outline">
      {{ t(`levels.${level}`) }}
    </Badge>

    <!-- Tags -->
    <Badge
      v-for="tag in tags"
      :key="tag"
      variant="outline"
      class="text-foreground-muted"
    >
      {{ tag }}
    </Badge>
  </div>
</template>
```

**Configuration shadcn-vue (components.json):**
```json
{
  "$schema": "https://shadcn-vue.com/schema.json",
  "style": "default",
  "typescript": true,
  "tailwind": {
    "config": "",
    "css": "app/assets/css/main.css",
    "baseColor": "zinc"
  },
  "framework": "nuxt",
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils"
  }
}
```

### Package Versions (Décembre 2025)

| Package | Version |
|---------|---------|
| shadcn-vue | latest (basé sur reka-ui) |
| reka-ui | 2.x (ex radix-vue) |
| @nuxtjs/i18n | 9.x ou 10.x |
| nuxt | 4.2.2 |

### Dépendances de cette story

**Requiert (stories précédentes):**
- Story 1-1 : Projet initialisé avec Nuxt Content, structure app/, article de test

**Bloque (stories futures):**
- Story 3-4 : Clickable badge navigation (ajoutera les liens sur les badges)

### Anti-patterns à éviter

- ❌ Ne PAS utiliser `radix-vue` directement - utiliser `reka-ui` (rebranding février 2025)
- ❌ Ne PAS hardcoder les labels en français - utiliser i18n
- ❌ Ne PAS oublier le contraste WCAG AA sur fond sombre
- ❌ Ne PAS rendre les badges cliquables dans cette story (Epic 3)
- ❌ Ne PAS créer de variants shadcn-vue personnalisés complexes - utiliser les classes Tailwind

### Considérations accessibilité

- Les badges avec couleurs piliers doivent avoir `text-white` pour contraste suffisant
- Envisager `aria-label` si le texte n'est pas suffisamment descriptif
- Les badges sont informatifs, pas interactifs (pas de `role="button"`)

### References

- [Source: architecture/project-structure-boundaries.md#Component Boundaries] - Structure composants
- [Source: ux-design-specification/visual-design-foundation.md#Pillar Colors] - Couleurs piliers
- [Source: ux-design-specification/design-system-foundation.md] - Choix shadcn-vue
- [Source: ux-design-specification/component-strategy.md#ArticleHeader] - Spécification composant
- [Source: architecture/core-architectural-decisions/frontend-architecture.md] - Organisation composants
- [Source: epics/epic-2-rich-article-experience.md#Story 2.1] - Requirements originaux
- [Source: architecture/implementation-patterns-consistency-rules/naming-patterns.md] - Conventions nommage

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

### File List
