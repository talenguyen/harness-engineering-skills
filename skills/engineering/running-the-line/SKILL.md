---
name: running-the-line
description: 'Run one concrete coding task through the Harness Engineering production line. Use when the user wants to run the line, apply harness engineering, implement a feature or fix safely, or coordinate spec -> TDD plan -> gates -> PR/review -> field feedback.'
---

# Running the Line

Run one well-defined task through the line without babysitting and without blind
merge. The point is not to produce ceremony; it is to make the agent read first,
plan before editing, validate frequently, recover from errors, and leave a
reviewable trail.

## Preflight: Should This Go On The Line?

Use the line for implementation tasks with observable correctness:

- Feature with business rules
- Bug fix with reproducible behavior
- Refactor where behavior must stay unchanged
- Production defect that can become a regression test

Do not force the line onto exploratory product/design work, greenfield
architecture choices, or tasks in repos where tests/build cannot run. For those,
first produce a short discovery note or Week 0 readiness backlog.

## Step 1: Read The Harness

Before writing a work order or code, inspect:

- Root agent rules: `AGENTS.md` or `CLAUDE.md`
- Build/test docs referenced by those rules
- Relevant source files and existing tests
- Existing PR template or CI/gate config

If the repo has no useful harness, say so. Do not pretend the task is ready for
hands-off execution.

## Step 2: Work Order

Create or update a work order with:

- Context: why this task exists and what area it touches
- Business rules
- Acceptance criteria
- Edge cases
- Off-limits
- Verification expectations

Ask only questions that change correctness, security, data handling, public
behavior, or rollback risk. Otherwise state a conservative assumption.

Stop here if correctness is still ambiguous.

## Step 3: TDD Plan Gate

Generate a plan before implementation:

- Testable business rules -> tests
- Edge cases -> tests or explicit inspection checks
- Non-testable changes -> verification checklist
- Public seams under test
- Implementation order: one vertical slice at a time

Try to reject the first plan once. If a rule has no test or inspection check,
the plan is not ready. Do not implement until the plan is approved or the user
explicitly overrides the gate.

## Step 4: Implement With Recovery

For each slice:

1. Write the failing test or verification check.
2. Implement the minimum change.
3. Run the narrow relevant command.
4. Fix the exact failure.
5. Repeat.

Never bypass hooks. Never weaken tests or config to get green. If a gate itself
is wrong, stop and make that a separate proposed config change.

## Step 5: PR/Review Package

Before final review, produce evidence:

- Commands run and result
- Acceptance criteria status
- Off-limits check
- Any docs/conventions updated because behavior changed
- Known residual risk

If the agent touched many files, added trivial tests, skipped gates, changed
config, or deleted tests, call out the upstream leak instead of burying it.

## Step 6: Field Feedback

If something leaks into review or production, update the line:

- Missing rule -> work order or agent rules
- Missing behavior check -> regression test
- Missing mechanical check -> gate
- Missed judgment call -> inspection checklist
- Missing observability -> monitoring/rollback note

## Final Output

End with:

- Work order summary/path
- TDD plan summary/path
- Gate evidence
- Review package
- Field feedback or next line-tightening action
