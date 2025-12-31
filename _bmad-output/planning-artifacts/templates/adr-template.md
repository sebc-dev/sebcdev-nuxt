---
status: proposed
date: YYYY-MM-DD
decision-makers: []
consulted: []
informed: []
---

# [Short Title of Decision]

## Context and Problem Statement

[Describe the context and problem statement. What is the issue that motivates this decision or change?]

## Decision Drivers

* [Driver 1: e.g., "TypeScript support required"]
* [Driver 2: e.g., "SSG compatibility with Cloudflare Pages"]
* [Driver 3: e.g., "Developer experience"]

## Considered Options

* [Option 1]
* [Option 2]
* [Option 3]

## Decision Outcome

Chosen option: "[Option X]", because [justification].

### Consequences

* Good, because [positive consequence]
* Good, because [positive consequence]
* Bad, because [negative consequence]
* Neutral, because [neutral consequence]

### Confirmation

[Describe how the implementation of the decision will be confirmed. E.g., "Review of the implementation in PR #XXX"]

## Pros and Cons of the Options

### [Option 1]

[Example: Pinia for state management]

* Good, because [argument]
* Good, because [argument]
* Neutral, because [argument]
* Bad, because [argument]

### [Option 2]

[Example: Vuex 4]

* Good, because [argument]
* Bad, because [argument]

### [Option 3]

[Example: Composables with provide/inject]

* Good, because [argument]
* Bad, because [argument]

## More Information

[Optional: Links to related ADRs, documentation, or discussions]

---

## MADR 4.0.0 Status Values

| Status | Description |
|--------|-------------|
| `proposed` | Decision is being discussed |
| `accepted` | Decision has been made |
| `deprecated` | Decision is no longer relevant |
| `superseded` | Replaced by another ADR (link it) |

## Naming Convention

```
docs/decisions/
├── 0000-use-madr-format.md
├── 0001-choose-nuxt-4-ssg.md
├── 0002-use-pinia-state.md
└── template.md
```

## Technical Debt Note (Optional)

If this decision introduces known technical debt:

| Debt Item | Impact | Remediation Plan |
|-----------|--------|------------------|
| [Description] | [High/Medium/Low] | [When/How to address] |
