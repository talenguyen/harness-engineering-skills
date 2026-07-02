---
name: tdd-plan-gate
description: Turn a work order into a TDD plan and stop for approval before implementation. Use when an AI coding agent needs a red-green-refactor plan, test coverage for acceptance criteria, public seams, implementation order, or a plan-review gate before writing code.
---

# TDD Plan Gate

The plan gate turns "correct" into machine-checkable feedback before the agent
edits implementation code.

## Inputs

Start from the work order. If no work order exists, reconstruct a small one
first or ask the user for the missing correctness rules.

## Plan format

```markdown
# TDD Plan: [task]

## Seams under test
- [Public interface or behavior boundary]

## Test plan
1. [ ] [Acceptance criterion or business rule] -> [expected observable result]

## Edge-case tests
1. [ ] [Edge case] -> [expected observable result]

## Off-limits coverage
- [Constraint] -> [how the plan or review will catch violations]

## Implementation order
1. Write failing test for [slice]
2. Implement the minimum code to pass
3. Run [relevant gate]
4. Repeat
```

## Review the plan before coding

Try to reject the first plan once. Look for:

- A business rule with no test.
- An edge case that only appears in prose.
- A test at the wrong seam or against internals.
- A tautological assertion that repeats the implementation.
- An off-limits rule that tests cannot catch and must be inspected later.

If you find a gap, revise the plan and show the rejection reason. If you cannot
find one, explicitly approve the plan and explain why.

## Implementation rules after approval

- One vertical slice at a time.
- Red before green.
- Minimal implementation per test.
- Do not add speculative behavior for future tests.
- Do not delete, weaken, or skip tests to pass.

Do not implement before approval unless the user explicitly overrides the gate.

