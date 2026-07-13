---
name: create-plan
description: "Designs the solution from an approved spec, traces every acceptance criterion to a design element, and routes exploratory work off the line. Use after Gate 1 when a spec section is approved."
---

# Create Plan

Input: an approved spec section (from `docs/specs/<slug>/<section>.md`).
Output: `docs/plans/<slug>.md` (one plan per slug, updated incrementally as sections are designed).

## Before designing — is this a line part or R&D?

- **Line part (well-defined):** proceed below.
- **Exploratory (unknown shape):** do NOT force it through spec→plan→tasks. Work it
  turn-by-turn with the human (conductor mode). Once the shape is known, write it up as a
  spec section and *then* put it on the line.

Record which items go where in the plan under "Line vs R&D."

## Procedure

1. Confirm each target section's `Status: approved`. Do not design against a draft.
2. Read the code the change will touch — reuse existing patterns over new dependencies.
3. Design the approach: components, data flow, key decisions. Use a mermaid diagram if it helps.
4. Build an **acceptance-criteria trace table**: every AC from the spec must map to a design element. A criterion with no home = design is incomplete.
5. Call out **risks and judgment calls** (security, data handling, open choices) — these become Inspection focus.
6. Note which changes are non-testable (config, migration, pure refactor) — these get a verification checklist instead of tests at the task stage.
7. Write to `docs/plans/<slug>.md`. If designing incrementally, update the existing plan and list the section(s) you just covered under **Sections covered**.

## Plan structure

```markdown
# Plan: <title>
- Slug: <slug>
- Spec: docs/specs/<slug>/overview.md
- Sections covered: <section-slugs designed so far>
- Status: draft
- Mode: github | local

## Line vs R&D
## Approach
## Design (mermaid if useful)
## Components touched
## Acceptance-criteria trace
| AC | Delivered by |
|----|--------------|
## Risks & judgment calls
## Non-testable changes (verification checklist needed)
```

## Key constraint

The plan is where *how* lives (signatures, algorithms, components). This detail is
**guidance for the implementer** — it must NOT be copied verbatim into task bodies as
requirements. Tasks state *what* (behavior); the plan stays as reference.

Next: `break-into-tasks`
