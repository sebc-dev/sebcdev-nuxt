# Frontend Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Component Structure** | `ui/`, `content/`, `layout/`, `search/` | Clear separation of concerns |
| **State Management** | Vue composables only | SSG blog doesn't need global store |
| **Prose Components** | Custom (ProseCode, ProseH2, ProseDetails) | Required for FR11-13 (code blocks), FR8 (ToC) |

**Component Organization:**

```
app/components/
├── ui/                    # shadcn-vue
├── content/               # ArticleCard, TableOfContents, ReadingProgress
├── layout/                # TheHeader, TheFooter, LanguageSwitcher
└── search/                # SearchCommand, SearchFilters
```

**Page & Layout Transitions:**

Configuration des transitions de pages et layouts dans `nuxt.config.ts` :

```typescript
export default defineNuxtConfig({
  app: {
    pageTransition: { name: 'page', mode: 'out-in' },
    layoutTransition: { name: 'layout', mode: 'out-in' }
  }
})
```

Classes CSS correspondantes dans `main.css` :

```css
/* Transition de pages */
.page-enter-active,
.page-leave-active {
  transition: opacity 200ms, transform 200ms;
}

.page-enter-from {
  opacity: 0;
  transform: translateY(10px);
}

.page-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

/* Transition de layouts */
.layout-enter-active,
.layout-leave-active {
  transition: opacity 300ms;
}

.layout-enter-from,
.layout-leave-to {
  opacity: 0;
}
```

| Option | Valeur | Description |
|--------|--------|-------------|
| `mode: 'out-in'` | Défaut recommandé | L'ancienne page sort avant que la nouvelle entre (évite chevauchements) |
| `mode: 'in-out'` | Rare | La nouvelle page entre avant que l'ancienne sorte |
| `mode: 'default'` | Simultané | Les deux transitions en parallèle |

**Note accessibilité** : Combiner avec `motion-safe:` pour respecter `prefers-reduced-motion` :

```css
.page-enter-active,
.page-leave-active {
  @apply motion-safe:transition-all motion-safe:duration-200;
}
```
