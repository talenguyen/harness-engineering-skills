---
name: running-the-line
description: Run one concrete coding task through the Harness Engineering production line. Use when the user wants to "run the line", apply harness engineering, implement a feature or fix safely, or coordinate work order -> TDD plan -> gates -> inspection -> field feedback.
---

# Running the Line

Run one well-defined task through five stations:

1. Work Order: define correct before code exists.
2. The Part: produce and approve a TDD plan, then implement test-first.
3. Andon Cord: run hard-failing gates and self-correct failures.
4. Inspection: review against the work order and judgment calls.
5. Field Returns: record leaks and push the catch upstream.

## First decision: should this go on the line?

Use the line for parts: features, bug fixes, refactors with clear expected
behavior, and production regressions that can become tests.

Do not force exploratory work through the line. If the user does not know what
they want yet, switch to interactive exploration and produce a work order only
after the shape is clear.

## Station 1: Work Order

Before implementation, create or update a work order with:

- Task
- Business rules
- Acceptance criteria
- Edge cases
- Off-limits

Ask clarifying questions only when the answer changes correctness, security,
data handling, or public behavior. Otherwise make a conservative assumption and
label it.

Stop when the work order is reviewable. Do not implement yet.

## Station 2: TDD Plan Gate

Produce a TDD plan from the work order:

- One test for each business rule or acceptance criterion.
- Tests for edge cases that are likely to escape.
- A note on the public seam or interface each test observes.
- An implementation order: one failing test, minimal implementation, repeat.

Try to reject the first plan once. Look for one missing case, ambiguous seam, or
off-limits rule that is not covered. If you cannot find one, say why the plan is
approved.

Do not implement until the plan is approved.

## Station 3: The Part and Andon Cord

Implement one vertical slice at a time:

1. Write the failing test.
2. Make the smallest implementation pass.
3. Run the relevant gate.
4. Fix the exact failure.
5. Repeat.

Never bypass hooks. Never loosen lint, type, test, or hook configuration to make
a failure disappear. Never delete or weaken a test to get green.

## Station 4: Inspection

Review the diff against the work order:

- Does each acceptance criterion pass?
- Did the implementation cross any off-limits boundary?
- Are the judgment calls sound: security, data handling, public API behavior,
  migration risk, operational behavior?
- Does the diff shape suggest an upstream leak: many files, few tests, skipped
  gates, or speculative changes?

Do not spend review energy on lint, formatting, or type issues if the gates ran
and passed. If those issues appear in review, Station 3 is leaking.

## Station 5: Field Returns

If a defect escapes, do more than fix it:

- Identify which station should have caught it.
- Add the missing acceptance criterion, test, hook, review check, or monitoring
  note.
- Record the leak so the next task starts with a tighter line.

## Final report

End with:

- Work order path or summary
- TDD plan path or summary
- Gates run and evidence
- Inspection findings
- Any leaks promoted upstream
