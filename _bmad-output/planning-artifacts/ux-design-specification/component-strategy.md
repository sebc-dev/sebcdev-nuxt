# Component Strategy

## Design System Components

**shadcn-vue Components (Direct Use)**
- Button, Badge, Input, Select, Checkbox
- Sheet, Dialog, Tooltip, Separator
- Skeleton, Collapsible, Progress (base)

**Components to Extend**
- Card → ArticleCard, FeaturedArticleCard
- Progress → ReadingProgress

## Custom Components

### CodeBlock
- **Purpose:** Syntax-highlighted code with copy button
- **Props:** `code`, `language`, `filename?`, `highlightLines?`
- **States:** Default, Hover, Copied (2s feedback)
- **Tech:** Shiki for syntax highlighting

### TableOfContents
- **Purpose:** Sticky article navigation with active highlight
- **Props:** `headings[]`, `activeId`
- **Behavior:** IntersectionObserver for active detection
- **Position:** Sticky sidebar (desktop), modal (mobile)

### ReadingProgress
- **Purpose:** Visual reading progress indicator
- **Position:** Fixed top, 3px height, accent color
- **Behavior:** Scroll-based calculation

### ArticleCard
- **Purpose:** Article preview in listings
- **Content:** Badges, title, description excerpt, meta
- **States:** Default, Hover (border accent)

### ArticleHeader
- **Purpose:** Article page header with metadata
- **Content:** Badges row, H1, reading time, date

### SearchFilters
- **Purpose:** Combinable filter sidebar
- **Behavior:** Instant update, URL sync
- **Mobile:** Sheet drawer

### ProseDetails
- **Purpose:** Styled `<details>` for "Approfondir" sections
- **Animation:** CSS transition on open/close

## Component Implementation Strategy

**Principles:**
1. Use shadcn-vue tokens for all custom components
2. Ensure WCAG AA accessibility on all components
3. Support keyboard navigation throughout
4. Optimize for Lighthouse 100 (minimal JS)

**File Structure:**
```
components/
├── ui/              # shadcn-vue (installed)
├── layout/          # Header, Footer, Container
├── article/         # CodeBlock, ToC, Progress, Header
├── cards/           # ArticleCard, FeaturedCard
└── search/          # Filters, Pagination
```

## Implementation Roadmap

**Phase 1 — Core (MVP Blocking)**
- CodeBlock, TableOfContents, ReadingProgress
- ArticleCard, ArticleHeader

**Phase 2 — MVP Complete**
- FeaturedArticleCard, SearchFilters
- LanguageSwitch, Pagination, ProseDetails

**Phase 3 — Polish**
- Skeleton loaders, Toast, Breadcrumbs
