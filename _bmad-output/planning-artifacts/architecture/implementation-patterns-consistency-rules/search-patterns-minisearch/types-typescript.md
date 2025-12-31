# Types TypeScript

```typescript
// app/types/search.ts
export interface BlogSearchDocument {
  id: string
  title: string
  description: string
  content: string
  slug: string
  locale: 'fr' | 'en'
  pillar: string
  tags: string[]
}

export interface SearchResult {
  id: string
  title: string
  description: string
  slug: string
  locale: string
  pillar: string
  score: number
  match: Record<string, string[]>
}
```
