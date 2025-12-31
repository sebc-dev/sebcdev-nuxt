---
stepsCompleted: []
inputDocuments: []
---

# {{project_name}} - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for {{project_name}}, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

{{fr_list}}

### NonFunctional Requirements

{{nfr_list}}

### Additional Requirements

{{additional_requirements}}

### FR Coverage Map

{{requirements_coverage_map}}

## Epic List

{{epics_list}}

<!-- Repeat for each epic in epics_list (N = 1, 2, 3...) -->

## Epic {{N}}: {{epic_title_N}}

{{epic_goal_N}}

<!-- Repeat for each story (M = 1, 2, 3...) within epic N -->

### Story {{N}}.{{M}}: {{story_title_N_M}}

As a {{user_type}},
I want {{capability}},
So that {{value_benefit}}.

**Acceptance Criteria:**

<!-- for each AC on this story -->

**Given** {{precondition}}
**When** {{action}}
**Then** {{expected_outcome}}
**And** {{additional_criteria}}

<!-- PR breakdown section - include only if story requires multiple PRs -->

#### PR Breakdown Plan

<!-- Analysis determines if breakdown is needed based on:
     - Multiple technical layers touched (infra, logic, UI, integration)
     - Complex business logic requiring isolated review
     - Multiple domains/modules impacted
     - New patterns or significant architectural changes
-->

**Breakdown Rationale:**
{{breakdown_rationale}}

<!-- Repeat for each PR (P = 1, 2, 3...) -->

**PR {{N}}.{{M}}.{{P}}: {{pr_title}}**
- **Objective:** {{pr_objective}}
- **Scope:** {{pr_scope}}
- **Verifiable by:** {{pr_verification}}

<!-- End PR repeat -->

<!-- If no breakdown needed, replace this section with:
**Single PR:** This story is cohesive enough for a single PR.
-->

<!-- End story repeat -->
