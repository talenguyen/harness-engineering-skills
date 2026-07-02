---
name: work-order
description: 'Create or sharpen a Harness Engineering work order before any implementation. Use when a coding task needs business rules, acceptance criteria, edge cases, off-limits constraints, clarification questions, or a definition of correct that an AI coding agent can build and test against.'
---

# Work Order

A work order defines "correct" before code exists. It catches the most expensive
defect: building the wrong thing correctly.

## What to produce

Use this structure:

```markdown
# Work Order: [task]

## Task
[One sentence describing the part to build or fix.]

## Business rules
1. [Testable rule]

## Acceptance criteria
- [ ] [Observable outcome]

## Edge cases
- [Case that changes behavior, risk, or correctness]

## Off-limits
- [Constraint the implementation must not cross]

## Assumptions
- [Only if needed; label assumptions clearly]
```

## How to write it

- Make every business rule testable.
- Prefer observable outcomes over implementation guesses.
- Include off-limits when a "green" implementation could still be wrong.
- Separate durable rules from perishable commands, versions, flags, or vendor
  details.
- If a claim needs an external statistic or exact tool behavior, verify it
  before including it or remove it.

## Clarifying questions

Ask only questions that change correctness:

- Public behavior or API contract
- Security boundary
- Data ownership, retention, or migration
- Failure mode
- Edge case with meaningful user impact

If the question is low risk, make a conservative assumption and mark it.

## Stop condition

Stop when a stranger could verify the work from the work order. Do not drift
into solution design unless design constraints are part of correctness.
