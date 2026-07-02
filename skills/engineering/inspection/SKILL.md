---
name: inspection
description: 'Review an AI-generated diff or PR against a Harness Engineering work order. Use when the user asks for code review, PR inspection, spec-fit review, acceptance criteria verification, judgment-call review, or leak analysis after gates ran.'
---

# Inspection

Inspection is not redoing lint by hand. It is the final check for spec fit,
boundaries, and judgment calls after upstream stations have done their job.

## Required Inputs

Prefer:

- Work order
- TDD/verification plan
- Diff or PR
- Gate output

If any are missing, mark that as a review risk. Do not silently reconstruct
everything and pretend confidence is high.

## Review Order

1. Acceptance criteria: pass/fail/unknown with evidence.
2. Off-limits: respected/violated/unknown.
3. Test quality: meaningful seams, not tautological, not trivial coverage theater.
4. Judgment calls: security, data handling, API behavior, migrations, failure modes.
5. Diff shape: many files, config edits, deleted tests, speculative changes.
6. Gate evidence: commands run, commands missing, failures ignored.
7. Leak analysis: which upstream station should catch each finding next time.

## Agent PR Smells

Call these out directly:

| Signal | Likely leak |
|---|---|
| Many files edited, tests trivial | Work order/spec drift |
| Large diff, no tests or checks | Plan gate skipped |
| Config changed to pass | Andon cord bypass |
| Deleted/softened tests | Agent optimized for green |
| Fix touches callers instead of root cause | Missing architectural constraint |
| PR summary lacks test evidence | Feedback/review package weak |

## Output Format

Lead with findings:

```markdown
## Findings
- [Severity] [file:line] [Issue]
  Evidence:
  Why it matters:
  Upstream station that should catch next time:

## Acceptance Criteria
- [ ] [criterion] - [pass/fail/unknown] - [evidence]

## Gate Evidence
- [command] - [passed/failed/not run]

## Residual Risk
- [what remains uncertain]
```

If there are no findings, say so and name any missing evidence.
