---
name: intake-requirement
description: "Understands what the user wants through one-question-at-a-time dialogue, then captures it as a resumable spec map with per-section Work Orders. Use when a new requirement, idea, or bug needs to become a spec. Stops at Gate 1."
---

# Intake Requirement — Gate 1

Goal: turn a fuzzy request into something a stranger could verify. The deliverable is
**shared understanding, captured** — not a document for its own sake.

## How to elicit

- Look facts up in the codebase; only ask the human about **decisions** (scope, priorities, trade-offs).
- One question at a time. Propose your recommended answer with each.
- Stay on the *what* and *why*. If you're reaching for data structures or function names, stop — that's design (Station 2).
- Every request gets a spec, no matter how simple. Keep it short for simple asks; never skip it.

## Output structure (files are the memory, not the chat)

```
docs/specs/<slug>/
  overview.md        ← the map
  <section>.md       ← one Work Order per section
```

### Procedure

1. Elicit until you can state the user's intent in 1-2 sentences. Reflect back; get confirmation.
2. Draft `overview.md` — intent, goals/non-goals, sections table, cross-cutting constraints. Present the map; confirm before detailing.
3. For each section (priority order):
   - Elicit details (one question at a time, recommend answers).
   - Draft `<section>.md` using the template below.
   - Re-read asking: "what would make this correct-looking but wrong?" Fix it.
   - Present the section spec; **stop for approval (Gate 1)**.
4. Update `overview.md` whenever a section changes the big picture.

### Resuming / pausing

Read `overview.md` → find the first non-`approved` section → continue there.
An approved section can flow to `create-plan` while you continue intake on the next section.

## Templates

### `overview.md`

```markdown
# Spec map: <title>
- Slug: <slug>
- Status: drafting | sections-in-progress | complete
- Date: <yyyy-mm-dd>

## Intent
<1-2 sentences: what and why>

## Goals / Non-goals
## Sections
| # | Section | Purpose | Status |
|---|---------|---------|--------|
| 1 | <slug>  | <...>   | todo   |

## Cross-cutting constraints
## Open questions
```

### `<section>.md`

```markdown
# Spec: <title> — <section>
- Parent: docs/specs/<slug>/overview.md
- Status: drafting | drafted | approved | implemented

## Context
Why this exists, what area it touches, what behavior changes.

## Business rules
1. <testable rule — observable behavior, not implementation>

## Acceptance criteria
- [ ] <outcome a stranger could verify>
- [ ] <...>
- [ ] <...>

## Edge cases
## Off-limits
## Open questions
```

## Gate 1

After drafting each section, present it and **stop**:

> "Section `<name>` drafted. Acceptance criteria: N. Open questions: <list or none>.
> Approve, or what should change?"

Do not proceed to design until approved. On approval, set `Status: approved`.

Next: `create-plan`
