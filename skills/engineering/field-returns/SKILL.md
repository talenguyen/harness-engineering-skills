---
name: field-returns
station: "5"
description: "Classifies a defect that escaped to production or past review to the station that should have caught it, adds the missing upstream test/rule, and tracks metrics. Use when a defect slips past review or surfaces in production."
produces:
  - "a new test or rule added at the station that should have caught the defect"
  - "a 'which station leaked' note + a metrics entry"
feeds_back_to:
  - "setup-project (new hook/gate)"
  - "intake-requirement (missing spec rule/edge case)"
  - "break-into-tasks (missing test in the task template)"
---

# Field Returns — Station 5 (Production → Upstream)

The most expensive station, and the one you want to catch **nothing** — anything caught
here already cost the maximum. When a defect does reach here, the response is not just
"fix the bug." It's: **which upstream station should have caught this, and why didn't it?**
Then add the test or rule so that *class* of defect dies at its station forever after.
That feedback arrow is what makes the line get smarter over time instead of just busier.

> Prerequisite this station leans on: monitoring, error tracking, and a **tested rollback
> path**. Rollback is often unsolved for CDN configs, ASO pipelines, and LiveOps
> automations — confirm it exists (Station 0) rather than assuming this backstop works.

---

## Procedure

1. **Stabilize first.** Roll back or mitigate the live impact before analysis. (This
   requires the rollback path confirmed in Station 0.)
2. **Classify the responsible station.** Ask which station *should* have caught this
   class of defect — the leftmost one that could have:

   | The defect was... | Should have been caught at... | Upstream fix goes to... |
   |---|---|---|
   | the wrong thing built correctly | Station 1 — Work Order | `intake-requirement.md`: add the missing rule / acceptance criterion / edge case |
   | a logic or behavior bug | Station 2 — The Part | `break-into-tasks.md`: add the test-> behavior pair; add a held-out test |
   | lint / type / format / a leaked secret | Station 3 — Andon Cord | `setup-project.md`: add / tighten the hook, fail-hard |
   | spec drift or a bad judgment call | Station 4 — Inspection | strengthen the inspection checklist; usually means an upstream station also leaked |

3. **Add the check at that station** — a concrete, permanent artifact: a new test, a new
   acceptance criterion, a new (or tightened) hook rule. The point is that this exact
   class can't escape again.
4. **Fix the immediate bug** (via the normal line: spec the fix if needed, task it, TDD it,
   gate it). The upstream check should now fail against the un-fixed code — proving the
   catch moved left.
5. **Record the leak note + metric** (below). Never just close the incident silently.

---

## The "which station leaked" note

Append one entry per escaped defect to `docs/field-returns.md` (create it on first use):

```markdown
## <date> — <short defect title>
- Impact: <what broke, who saw it>
- Responsible station: <1 | 2 | 3 | 4>
- Why it escaped: <one sentence>
- Upstream check added: <the test / rule / criterion, with file path>
- Immediate fix: <link / commit>
```

---

## Metrics — treat the rollout as an experiment, not doctrine

Process discipline decays into ceremony if nobody watches it. Track a few numbers and
review them at **30 / 60 / 90 days**:

- **Plan-rejection rate** — how often the first task plan gets a real rejection at Gate 2.
  If this trends to zero, reject-first has become rubber-stamping.
- **Escaped-defect count** — defects caught at Station 4 or 5 (per station). This is the
  score you want falling as upstream checks accumulate.
- **Hook-bypass count** — times a gate was skipped or loosened. Target: zero. Any non-zero
  value is a hole in the enforcement boundary to investigate.

Keep them in `docs/field-returns.md` under a `## Metrics` heading, with a dated row at
each 30/60/90-day check-in.

```markdown
## Metrics
| Date | Plan-rejection rate | Escaped defects (S4 / S5) | Hook bypasses |
|------|---------------------|---------------------------|---------------|
| D+30 |                     |                           |               |
```

---

## Skipped gates are visible debt, not silence

If a gate was skipped under deadline pressure (spec approval rushed, reject-first not done,
a hook temporarily loosened), it must be **logged as debt** so it stays visible — add it to
the leak note or a `## Debt` section. A silently skipped gate is how the line quietly stops
being trusted.

---

## Self-check (dry-run validation)

- [ ] A mock escaped defect is classified to a single responsible station with a reason.
- [ ] A concrete upstream check (test / rule / criterion) is added at that station, and it
      would fail against the pre-fix code.
- [ ] A leak note is appended to `docs/field-returns.md`.
- [ ] A metrics entry exists (plan-rejection rate, escaped-defect count, hook-bypass count).
- [ ] Any skipped gate is logged as visible debt.

---

## Handoff

The upstream fix routes back into the relevant setup / intake / tasks skill so the next run
of the line catches this class of defect earlier. The line is now smarter than before.
