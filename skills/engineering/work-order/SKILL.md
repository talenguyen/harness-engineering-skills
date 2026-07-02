---
name: work-order
description: 'Create or sharpen a Harness Engineering work order before implementation. Use when a coding task needs context, business rules, acceptance criteria, edge cases, off-limits constraints, verification expectations, or clarification before an AI coding agent can safely build.'
---

# Work Order

A work order defines "correct" before code exists. It should be short enough to
use and specific enough that a stranger can verify the result.

## Do Not Write Vague Specs

Reject prompts like "build auth", "fix the dashboard", or "clean this up" until
they are converted into observable rules. A vague work order produces either
babysitting or vibe merging.

## Work Order Format

```markdown
# Work Order: [task]

## Context
[Why this task exists, where in the repo it lives, and what user/system behavior changes.]

## Business Rules
1. [Testable rule]

## Acceptance Criteria
- [ ] [Observable outcome]

## Edge Cases
- [Case that changes behavior, security, data handling, or failure mode]

## Off-Limits
- [Boundary the implementation must not cross]

## Verification
- Tests:
- Commands:
- Manual/inspection checks:

## Assumptions
- [Only if needed; label assumptions clearly]
```

## Clarifying Questions

Ask questions only when the answer changes:

- Public behavior or API contract
- Security boundary
- Data ownership, retention, migration, or rollback
- Failure mode
- Edge case with meaningful user impact
- Which existing behavior must remain unchanged

For low-risk unknowns, make a conservative assumption and mark it.

## Quality Checks

Before handing off to implementation, verify:

- Every business rule has an acceptance criterion or verification check.
- Every edge case either has a planned test or an inspection check.
- Off-limits are concrete enough to review.
- The work order does not prescribe implementation details unless they are
  correctness constraints.
- Commands, versions, and tool flags are either discovered from the repo or
  clearly marked as examples.

Stop if "correct" is still not verifiable.
