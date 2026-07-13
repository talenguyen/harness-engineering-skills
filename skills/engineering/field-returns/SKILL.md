---
name: field-returns
description: "Classifies an escaped defect to the station that should have caught it, adds the missing upstream check, and logs it. Use when a defect slips past review or surfaces in production."
---

# Field Returns

A defect reached production (or slipped past review). Respond in this order:

## 1. Stabilize

Roll back or mitigate the live impact first.

## 2. Classify the responsible station

| The defect was... | Should have been caught at | Upstream fix |
|---|---|---|
| Wrong thing built correctly | Station 1 (spec) | Add the missing rule/AC/edge case to the spec section |
| Logic or behavior bug | Station 2 (tasks/tests) | Add a test that catches it |
| Lint / type / format / leaked secret | Station 3 (gates) | Add or tighten the hook |
| Spec drift / bad judgment call | Station 4 (inspection) | Strengthen the checklist; usually an upstream station also leaked |

## 3. Add the upstream check

A concrete artifact (test, rule, hook, AC) at the responsible station so this class of
defect can't escape again. The check should **fail** against the pre-fix code — that proves
it moved left.

## 4. Fix the bug

Route the fix through the normal line (spec if needed → task → TDD → gate → PR).

## 5. Log it

Append an entry to `docs/field-returns.md`:

```markdown
## <date> — <title>
- Impact: <what broke>
- Responsible station: <1-4>
- Why it escaped: <one sentence>
- Upstream check added: <file/path>
- Fix: <commit/PR>
```

Track metrics at 30/60/90 days: plan-rejection rate, escaped-defect count, hook-bypass count.
If a gate was skipped under pressure, log it as visible debt — not silence.
