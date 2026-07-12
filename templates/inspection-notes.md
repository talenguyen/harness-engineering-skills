# Inspection — <task id / PR>

Narrow Station 4 review. Stations 1–3 already killed the wrong-thing, the logic, and the
mechanical defects — do **not** re-read everything. Check only:

- [ ] **Acceptance criteria.** Every box in the task's Work Order ticked against real behavior.
- [ ] **Judgment calls.** Security boundaries, data handling, choices the spec left open.
- [ ] **Intent vs harness.** Did it satisfy the intent, or just the visible tests?
- [ ] **Diff shape.** Many files + trivial tests → spec drift. Large diff + no tests → Station 2 skipped.
- [ ] **AI smells.** Unexplained new deps, diff size vs coverage, CI actually passed.
- [ ] **Codebase health.** Duplication, complexity, architectural fit.

Verdict: <approve / request changes — with which upstream station leaked>
