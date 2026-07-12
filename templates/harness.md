# Harness Engineering Line — project manual

Copy this file into a project (with `AGENTS.harness.md` → the project's `AGENTS.md`) to run
the line here. `AGENTS.md` is the always-loaded pointer; this file holds the authoritative
non-negotiables and this project's profile. The full station procedures live in the
Harness Engineering skills (`running-the-line` orchestrator + the seven station skills).

> **The Law:** A defect costs more at every station it survives. Each station must catch its
> own defects and never pass one downstream. Every technique moves the catch left.

---

## Always-on rules (never violated, on any task)

1. **Plan before you edit.** Read the code and plan before implementing; no premature patching.
2. **Tests first.** A failing test states what "done" means before the implementation exists.
3. **Never bypass hooks.** No `--no-verify`, no skipping CI or any enforced gate.
4. **Never weaken a gate to pass it.** Don't loosen config or delete/skip tests; fix the code.
   If a rule is genuinely wrong, stop and surface it.
5. **The Work Order is human-owned.** Scope, acceptance criteria, and trade-offs need human
   approval — Gate 1 (after the spec) and Gate 2 (after the task plan).
6. **Isolate before you automate.** Confirm the Station 0 checklist below before any
   unattended run.

## The line

```text
Station 0 Isolate → 1 Work Order → 2 The Part (TDD+plan) → 3 Andon Cord (fail-hard hooks)
   → 4 Inspection (narrow review) → 5 Field (prod → upstream)
```

Run one task at a time through `implement-task-loop ⇄ deliver` (per task: implement → its
own PR → human merge → next). Load the station skill for your stage; start with
`running-the-line` or `setup-project`.

---

## Project profile

<!-- project-profile:start -->
<!-- setup-project fills this in. -->

**Detected stack.** _Unset — run `setup-project` to detect and record it here (language,
package manager, test runner, gate tools)._

**Station 0 isolation checklist** — _verify each box yourself, record concrete evidence next
to it, and date it with today's date. Do not pre-check a box you have not verified._
- [ ] Sandbox —
- [ ] Least-privilege credentials —
- [ ] No production / deploy / write-API access —
- [ ] CI budget cap —
- [ ] Rollback path (tested) —

**Baseline owner:** _Unset — assign a person; an unowned lint/type baseline drifts._
<!-- project-profile:end -->
