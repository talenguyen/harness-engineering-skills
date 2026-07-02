---
name: inspection
description: Review an AI-generated diff or PR against a Harness Engineering work order. Use when the user asks for code review, PR inspection, spec-fit review, review against acceptance criteria, or a check for judgment calls after automated gates have run.
---

# Inspection

Inspection is final human review after upstream stations have done their work.
It should be narrow and rigorous: spec fit plus judgment calls.

## Inputs

Prefer these inputs:

- Work order
- TDD plan
- Diff or PR
- Gate output

If the work order is missing, reconstruct the likely acceptance criteria and
mark that as a review risk.

## Review order

1. Verify each acceptance criterion.
2. Check each off-limits constraint.
3. Review judgment calls: security, data handling, public behavior, migration
   risk, operational failure modes, and architecture boundaries.
4. Inspect diff shape: files touched, tests added, speculative changes, deleted
   tests, config changes, and gate bypasses.
5. Identify which upstream station leaked for each issue.

Do not lead with style comments if gates should have caught them. If style,
format, or type defects are present after gates supposedly ran, call out Station
3 as leaking.

## Output format

Lead with findings:

```markdown
## Findings
- [Severity] [file:line] [Issue]
  Evidence:
  Why it matters:
  Upstream station that should catch next time:

## Acceptance criteria
- [ ] [criterion] - [pass/fail/unknown]

## Gate evidence
- [gate] - [passed/failed/not run]

## Residual risk
- [what remains uncertain]
```

If there are no issues, say that clearly and name the remaining test or review
gap, if any.

