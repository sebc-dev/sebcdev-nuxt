# Naming Patterns

**File Naming Conventions:**

| Type | Convention | Exemple |
|------|------------|---------|
| Composants Vue | PascalCase | `ArticleCard.vue` |
| Pages Nuxt | kebab-case | `[slug].vue` |
| Composables | camelCase + `use` | `useReadingTime.ts` |
| Utilitaires | camelCase | `formatDate.ts` |
| Types | PascalCase | `Article.ts` |
| Contenu MDC | kebab-case | `mon-premier-article.md` |

**Component Naming Prefixes (Vue 3 Style Guide):**

| Préfixe | Usage | Exemple |
|---------|-------|---------|
| `Base` | Composants présentationnels réutilisables | `BaseButton.vue`, `BaseInput.vue` |
| `The` | Single-instance components (une seule instance par app) | `TheHeader.vue`, `TheSidebar.vue` |
| Parent+Child | Composants fortement couplés | `ArticleList.vue`, `ArticleListItem.vue` |

**Multi-word rule:** Tous les composants doivent avoir un nom multi-mots pour éviter les collisions avec les éléments HTML actuels et futurs. Exception: le composant root `App.vue`.

```
app/components/
├── BaseButton.vue       # ✅ Présentationnel réutilisable
├── TheHeader.vue        # ✅ Single-instance
├── ArticleCard.vue      # ✅ Multi-word
├── ArticleCardMeta.vue  # ✅ Fortement couplé à ArticleCard
└── Button.vue           # ❌ Single-word (collision potentielle)
```

**Composable TypeScript Conventions (VueUse pattern):**

| Type | Convention | Exemple |
|------|------------|---------|
| Options d'entrée | `Use{Name}Options` | `UseReadingTimeOptions` |
| Type de retour | `Use{Name}Return` | `UseReadingTimeReturn` |

```typescript
// app/composables/useReadingTime.ts
interface UseReadingTimeOptions {
  wordsPerMinute?: number
  locale?: string
}

interface UseReadingTimeReturn {
  minutes: Readonly<Ref<number>>
  formatted: Readonly<Ref<string>>
}

export function useReadingTime(
  content: MaybeRefOrGetter<string>,
  options: UseReadingTimeOptions = {}
): UseReadingTimeReturn {
  // ...
}
```

**i18n Key Naming:**
- Nested objects structure
- Groupé par feature/section

```typescript
// ✅ Correct
{
  header: {
    nav: { home: "Accueil", articles: "Articles" }
  },
  article: {
    readingTime: "{min} min de lecture"
  }
}
```

**CSS/Tailwind Naming:**
- Classes custom: kebab-case (`article-card`)
- CSS variables: `--color-primary`, `--spacing-lg`
- Animations: kebab-case (`fade-in`, `slide-up`)
