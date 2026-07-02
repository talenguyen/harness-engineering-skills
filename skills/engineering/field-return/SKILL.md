---
name: field-return
description: 'Convert a production bug, incident, customer report, or escaped defect into an upstream harness improvement. Use when something reached production or review late and the user needs a regression test, new acceptance criterion, hook, monitoring note, or station-leak analysis.'
---

# Field Return

A field return is a defect that escaped the line. The fix is not complete until
the class of defect is moved upstream.

## Process

1. Reproduce or characterize the defect.
   - What failed?
   - Who observed it?
   - What was the expected behavior?
   - What was the actual behavior?

2. Classify the escape.
   - Work Order leaked if correctness was never specified.
   - The Part leaked if behavior was specified but not tested.
   - Andon Cord leaked if a mechanical gate should have failed.
   - Inspection leaked if judgment or spec drift was visible in review.
   - Field leaked if monitoring, rollback, or alerting failed to surface it.

3. Add the upstream catch.
   - Missing rule -> update the work order template or project rules.
   - Missing behavior check -> add a regression test.
   - Missing mechanical check -> add or tighten a gate.
   - Missed review concern -> add an inspection checklist item.
   - Missing observability -> add monitoring, alerting, or rollback notes.

4. Fix the defect.
   - Keep the fix as small as possible.
   - Verify the new upstream catch fails before the fix and passes after it when
     that is practical.

## Field return record

```markdown
# Field Return: [defect]

## Symptom
[What was observed]

## Expected behavior
[What should have happened]

## Escape analysis
- Station that should have caught it:
- Why it escaped:

## Upstream promotion
- New/updated work order rule:
- New/updated test:
- New/updated gate:
- New/updated inspection item:

## Fix evidence
- [commands, screenshots, logs, or review notes]
```

Do not stop at "bug fixed." The line gets better only when the escape becomes a
future catch.
