# User Journey Flows

## Journey 1: Quick Win Reading (Lucas)

**Goal:** Find working code and copy it in under 60 seconds.

**Flow:**
1. Arrive from search → Page loads < 2.5s
2. See title + TL;DR → Scan for relevance (5-10s)
3. Scroll to code block → Identify solution (10-20s)
4. Click Copy → Feedback "Copié!" (1s)
5. Paste in IDE → Code compiles
6. Success → Relief + possible bookmark

**Critical UX Elements:**
- LCP < 2.5s
- TL;DR above fold
- Copy button always visible
- Code with inline comments

## Journey 2: Deep Dive Learning (Chloé)

**Goal:** Understand the "why" behind a pattern.

**Flow:**
1. Read TL;DR → Get context
2. Encounter code → Check understanding
3. If confused → Read inline comments
4. Need more depth → Open `<details>` sections
5. "Eureka" moment → Pattern understood
6. Continue to "Approfondir" → Build complete understanding
7. Success → Can explain to colleague

**Critical UX Elements:**
- Progressive disclosure via `<details>`
- "Why this choice?" sections
- Non-intimidating structure
- Supportive tone

## Journey 3: Discovery & Filtering

**Goal:** Find relevant article in catalog.

**Flow:**
1. Enter site (homepage, /articles, or badge click)
2. See sidebar filters
3. Select filters (theme, level, category)
4. Results update instantly (no reload)
5. URL reflects filters (shareable)
6. Find relevant article → Click card
7. Navigate to article

**Critical UX Elements:**
- Instant filter updates
- URL synchronization
- Filter counts displayed
- Reset button when active

## Journey 4: Bilingual Navigation

**Goal:** Switch language without friction.

**Flow:**
1. Click language switch (FR/EN)
2. System checks article availability
3. If translated → Load translated version
4. If not → Show message + option to stay
5. Preserve filters and scroll position
6. Continue reading in new language

**Critical UX Elements:**
- Graceful handling of untranslated content
- Filter preservation
- Position preservation when possible

## Journey Patterns

**Immediate Feedback Pattern**
| Action | Feedback | Duration |
|--------|----------|----------|
| Copy | "Copié!" | 2s |
| Filter | Count update | Instant |
| Details open | Chevron rotate | 200ms |

**Progressive Disclosure Pattern**
| Level | Content | Trigger |
|-------|---------|---------|
| 1 | TL;DR | Default visible |
| 2 | Code + explanation | Scroll |
| 3 | "Why?" details | Explicit click |
| 4 | FAQ, resources | End of article |

**Continuous Orientation Pattern**
- Progress bar (article position)
- ToC highlight (active section)
- Breadcrumbs (navigation path)
- URL sync (app state)

## Flow Optimization Principles

1. **Minimize Steps to Value** — Lucas gets code in < 60s
2. **Reduce Cognitive Load** — One decision at a time
3. **Provide Clear Progress** — Always know where you are
4. **Handle Errors Gracefully** — No dead ends
5. **Create Eureka Moments** — Strategic "Ah!" opportunities
