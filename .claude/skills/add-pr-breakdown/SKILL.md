---
name: add-pr-breakdown
description: Add PR breakdown plans to existing stories. Analyzes story complexity and proposes reviewable PR splits. Usage: /add-pr-breakdown [epic-file or story-id]
---

# Add PR Breakdown to Existing Stories

**Goal:** Analyze existing stories and add PR breakdown plans to facilitate code reviews.

## Arguments

- `$ARGUMENTS` can be:
  - An epic file path (e.g., `epic-1-core-reading-experience.md`) → process all stories in that epic
  - A story ID (e.g., `1.2`) → process only that specific story
  - Empty → list available epics and ask user to choose

## Execution

### 1. Locate Stories

If no argument provided:
```
List epic files in: {project-root}/_bmad-output/planning-artifacts/epics/
Display: "Which epic do you want to process? (or specify a story ID like 1.2)"
```

If argument is a file path:
```
Load the specified epic file
Extract all stories from the file
```

If argument is a story ID (format N.M):
```
Find the epic file containing that story
Load only that specific story
```

### 2. PR Breakdown Analysis

For each story, analyze against these complexity factors:

| Factor | Triggers Breakdown | Single PR OK |
|--------|-------------------|--------------|
| **Technical Layers** | Touches 3+ layers (types, services, UI, tests) | 1-2 related layers |
| **Business Logic** | Complex algorithms, state machines, validations | Simple CRUD |
| **Modules/Domains** | Impacts multiple domains | Single domain |
| **New Patterns** | New architectural patterns | Existing patterns |
| **External Systems** | New API integrations | Internal only |

**Analysis Process:**

1. Read the story's Technical Scope section
2. Identify which layers are touched (infra, logic, UI, integration)
3. Check for complexity indicators in acceptance criteria
4. Count domains/modules involved
5. Determine if 2+ factors trigger breakdown

### 3. Propose PR Plan

For each story requiring breakdown, propose:

```markdown
#### PR Breakdown Plan

**Breakdown Rationale:**
[Explain why this story needs multiple PRs based on analysis]

**PR {N}.{M}.1: {title}**
- Objective: [single clear goal]
- Scope: [files/modules]
- Verifiable by: [how to validate]

**PR {N}.{M}.2: {title}**
- Objective: [single clear goal]
- Scope: [files/modules]
- Verifiable by: [how to validate]

[... additional PRs as needed]
```

For stories NOT requiring breakdown:

```markdown
#### PR Breakdown Plan

**Single PR:** This story is cohesive enough for a single PR.
- Scope: [summarize technical scope]
- Verifiable by: [acceptance criteria summary]
```

### 4. PR Pattern Reference

Use this pattern as guide (adapt based on story):

```
PR 1: Infrastructure (~100-150 lines)
  - Dependencies, types, configuration
  - Verifiable by: Types compile, deps install

PR 2: Core Logic (~200-250 lines)
  - Services, composables, business logic
  - Verifiable by: Unit tests pass

PR 3: UI / Presentation (~200-300 lines)
  - Components, styles, accessibility
  - Verifiable by: Component renders correctly

PR 4: Integration (~100-150 lines)
  - Connect pieces, E2E tests, documentation
  - Verifiable by: E2E tests pass
```

### 5. User Validation

After proposing each story's PR breakdown:

```
Display the proposed PR breakdown plan.
Ask: "Does this breakdown make sense? [Y]es / [E]dit / [S]kip"

IF Y: Append the PR breakdown section to the story in the file
IF E: Let user suggest modifications, then re-propose
IF S: Move to next story without adding breakdown
```

### 6. File Update

When user approves, insert the PR Breakdown Plan section after the story's existing content:

- Find the story section in the epic file
- Locate the end of that story (before `---` or next `## Story`)
- Insert the PR Breakdown Plan section
- Preserve all existing content

### 7. Summary

After processing all stories, display:

```
## PR Breakdown Summary

Processed: X stories
- With breakdown: Y stories (Z total PRs)
- Single PR: W stories
- Skipped: V stories

Files modified:
- [list of modified files]
```

## Example Output

For Story 1.2 (Dark Theme & Typography System):

```markdown
#### PR Breakdown Plan

**Breakdown Rationale:**
This story touches infrastructure (TailwindCSS config, fonts),
styling system (tokens, theme), and UI components (shadcn-vue setup).
Breaking into 3 PRs ensures focused reviews.

**PR 1.2.1: TailwindCSS 4 & Font Infrastructure**
- Objective: Set up TailwindCSS 4 with Vite plugin and font loading
- Scope: package.json, vite.config.ts, app/assets/css/main.css (base), font files
- Verifiable by: Build succeeds, fonts load in browser

**PR 1.2.2: Dark Theme Token System**
- Objective: Implement oklch color tokens and dark theme
- Scope: app/assets/css/main.css (@theme tokens), tailwind config
- Verifiable by: Colors render correctly, contrast meets WCAG AA

**PR 1.2.3: shadcn-vue Integration**
- Objective: Initialize shadcn-vue with Reka UI base components
- Scope: components.json, lib/utils.ts, initial shadcn components
- Verifiable by: Sample component renders with correct theme
```