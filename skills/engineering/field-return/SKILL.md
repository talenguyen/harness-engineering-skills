---
name: field-return
description: 'Convert a production bug, incident, customer report, or escaped defect into an upstream harness improvement. Use when something reached production or review late and the user needs a regression test, new work-order rule, gate, inspection check, or escape analysis.'
---

# Field Return

A field return is proof that the line leaked. The fix is incomplete until the
class of defect is moved upstream.

## Process

1. Characterize the escape.
   - Symptom
   - Expected behavior
   - Actual behavior
   - Scope and impact
   - Reproduction path or strongest available evidence

2. Identify the first station that could have caught it.
   - Work Order: correctness was never specified.
   - Plan/Test: specified behavior had no test/check.
   - Andon Cord: mechanical gate was missing, weak, or bypassed.
   - Inspection: judgment issue was visible but missed.
   - Field: monitoring/rollback/alerting failed to surface it quickly.

3. Promote the catch upstream before or alongside the fix.
   - New work-order rule
   - Regression test or characterization test
   - Gate or ratchet rule
   - Inspection checklist item
   - Observability or rollback note

4. Fix and prove.
   - Show the new catch failing before the fix when practical.
   - Apply the smallest fix.
   - Show the catch passing after the fix.

## Record Format

```markdown
# Field Return: [defect]

## Symptom
[What was observed]

## Expected vs Actual
- Expected:
- Actual:

## Escape Analysis
- First station that could have caught it:
- Why it escaped:

## Upstream Promotion
- Work order update:
- Regression test/check:
- Gate update:
- Inspection update:

## Fix Evidence
- [commands, logs, links, or review notes]
```

Do not stop at "bug fixed." The line gets better only when the escape becomes a
future catch.
