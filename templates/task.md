---
id: T-003
title: <short objective>
section: <slug>/<section>    # the spec section this task implements (docs/specs/<slug>/<section>.md)
depends_on: [T-001, T-002]
status: todo                 # todo | in_progress | in_review | done  (blocked is DERIVED, never written)
parallel_safe: true          # false if it touches files another task touches
touches: [src/auth/*.ts]     # file globs — used for the parallel overlap check
issue:                       # GitHub mode only: the mirrored issue number (e.g. 42)
---

## Task: [what]

### Business rules
1. [testable rule]

### Acceptance criteria
- [ ] [observable outcome]

### Edge cases
- [the thing that breaks at 2am]

### Off-limits
- [the constraint the agent must not cross]

### Demo
- [what can be shown working after this task]
