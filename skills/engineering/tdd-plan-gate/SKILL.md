---
name: tdd-plan-gate
description: 'Turn a work order into an implementation plan that validates behavior before coding. Use when an AI coding agent needs a TDD plan, public seams, test coverage for acceptance criteria, non-TDD verification checklist, or a stop-for-approval gate.'
---

# TDD Plan Gate

The plan gate forces read-first and validate-first behavior. It is the cheapest
moment to catch a missing case because no implementation exists yet.

## Inputs

Start from a work order. If it does not exist, create or sharpen one first. Also
read existing tests so the plan uses the repo's real testing style and public
seams.

## Decide The Verification Mode

Not every task is TDD-shaped.

| Task | Plan style |
|---|---|
| Business behavior | TDD: one failing test per rule/edge case |
| Bug fix | Regression test first, then fix |
| Refactor | Existing tests stay green; add characterization test only for risky behavior |
| Config/build change | Verification checklist with exact commands |
| Migration | Before/after state, rollback check, data safety checks |
| Prototype/exploration | Throwaway branch, explicit merge gate later |

## Plan Format

```markdown
# Plan: [task]

## Seams Under Test
- [Public interface or behavior boundary, discovered from repo]

## Test / Verification Plan
1. [ ] [Rule or edge case] -> [test/check] -> [expected observable result]

## Off-Limits Coverage
- [Constraint] -> [test, gate, or inspection check that catches violation]

## Implementation Order
1. Read [files/tests]
2. Write failing test/check for [slice]
3. Implement minimum change
4. Run [exact command]
5. Repeat
```

## Reject The First Plan Once

Try to find one gap:

- Rule with no test/check
- Edge case left in prose
- Test aimed at internals instead of public behavior
- Tautological assertion
- No command evidence
- Off-limits not covered by tests, gates, or inspection
- Too many slices bundled together

If there is a gap, show the rejection reason and revise. If there is no gap,
say why the plan is ready.

Do not implement before the plan is approved unless the user explicitly
overrides the gate.
