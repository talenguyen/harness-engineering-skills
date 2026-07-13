---
name: break-into-tasks
description: "Converts a design plan into TDD task files with an acyclic dependency graph. Performs reject-first review and stops at Gate 2 for human approval. Use after a plan exists."
---

# Break Into Tasks — Gate 2

Input: `docs/plans/<slug>.md`.
Output: `docs/tasks/T-<NNN>.md` files (one per task).

## Core rule: tasks describe WHAT, not HOW

Do not put in a task body: function signatures, algorithm pseudocode, private helper names,
or internal data shapes.

Litmus test: could someone implement this a different reasonable way and still pass every
acceptance criterion? If not, you wrote *how*.

Exception: a genuine public contract (HTTP endpoint, CLI flags, library interface) goes
under `## Interface contract` — nothing internal.

## Procedure

1. Load the plan and its acceptance-criteria trace.
2. Slice into tasks — each produces a working, demoable increment.
3. Assign `id`, `section` (→ spec section it implements), `depends_on` (acyclic).
   - **Check dependency coverage:** every symbol/module a task uses must come from a task in
     its `depends_on`. A task that uses `portfolio_return()` must depend on the task that
     defines it. Also check for over-declared deps (kills parallelism for no reason).
4. Set `parallel_safe: false` if it touches files another task touches.
5. Write the body: context, business rules (behavioral), acceptance criteria (≥3,
   stranger-verifiable), edge cases, off-limits, test plan (test→behavior pairs), demo.
   For non-testable rules (config, migration, refactor), give a verification checklist
   instead of a test.
6. GitHub mode: mirror each task to an issue (idempotently — check before creating).
   Record the issue number in `issue:` frontmatter.
7. **Reject-first:** try to find one missing case in your own plan. Add it.
8. Present the task list + dependency graph. **Stop for Gate 2 approval.**

## Task file format

```markdown
---
id: T-003
title: <short objective>
section: <slug>/<section>
depends_on: [T-001, T-002]
status: todo | in_progress | in_review | done
parallel_safe: true
touches: [src/auth/*.ts]
issue:
---
## Task: <what>
### Business rules
### Acceptance criteria
### Edge cases
### Off-limits
### Test plan
### Demo
```

Status `blocked` is derived (= todo with an unfinished dep), never written.

## Done when

- Every task has valid frontmatter + behavioral body (no signatures/algorithms).
- Graph is acyclic; every dep id exists; dep coverage checked.
- Reject-first performed; Gate 2 approval received.

Next: `implement-task-loop`
