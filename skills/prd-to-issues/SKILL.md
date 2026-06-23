---
name: prd-to-issues
description: Rozlož PRD na nezávislé GitHub Issues pomocí vertical slices (tracer bullets). Použij když chce uživatel rozdělit PRD na úkoly, vytvořit implementační tickety, nebo zmíní "issues" / "vertical slices" / "rozděl PRD".
---

# PRD to Issues

Break a PRD into independently-grabbable GitHub issues using vertical slices (tracer bullets).

## Process

### 1. Locate the PRD

Ask the user for the PRD GitHub issue number (or URL).

If the PRD is not already in your context window, fetch it with `gh issue view <number>` (with comments).

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code.

### 3. Draft vertical slices

Break the PRD into **tracer bullet** issues. Each issue is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Slices may be **HITL** (human-in-the-loop, requires human interaction, e.g. architectural decision or design review) or **AFK** (can be implemented and merged without human interaction). Prefer AFK over HITL where possible.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- Order slices to flush out unknown unknowns early (tracer bullet principle)
</vertical-slice-rules>

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each slice show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices (if any) must complete first
- **User stories covered**: which user stories from the PRD this addresses

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?

Iterate until the user approves the breakdown.

### 5. Create the GitHub issues

For each approved slice, create a GitHub issue using `gh issue create`. Use the issue body template below.

Create issues in dependency order (blockers first) so you can reference real issue numbers in the "Blocked by" field.

<issue-template>
## Parent PRD

#<prd-issue-number>

## What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation. Reference specific sections of the parent PRD rather than duplicating content.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Blocked by

- Blocked by #<issue-number> (if any)

Or "None, can start immediately" if no blockers.

## User stories addressed

Reference by number from the parent PRD:

- User story 3
- User story 7

## Type

AFK / HITL
</issue-template>

Do NOT close or modify the parent PRD issue.

### 6. Update external task tracker (if applicable)

If the project tracks tasks in an external tool, add links to the created GitHub Issues into the corresponding task.

## Rules

- Each issue = thin vertical slice through ALL layers, not a horizontal slice of one layer
- Tracer bullet principle: flush out unknowns early
- PRD describes the destination, issues describe the journey
- After creating, print all issue URLs and a summary table
