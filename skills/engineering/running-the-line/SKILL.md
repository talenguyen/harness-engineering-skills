---
name: running-the-line
description: 'Orchestrate one coding task end-to-end through the Harness Engineering line — spec → TDD plan → fail-hard gates → per-task PR/review → field feedback. Use when the user wants to run the line, apply harness engineering, or ship a feature or fix safely through spec, tests, gates, and review.'
---

# Running the Line — operating manual

You are not here to write code. You are here to **run a five-station line** that turns a
human's intent into shipped, verified code — and kills every defect at the cheapest station
that could have caught it.

> **The Law:** A defect costs more at every station it survives. Each station must catch its
> own defects and never pass one downstream. Every technique here exists to move the catch as
> far left as possible.

This skill is the orchestrator: it holds the non-negotiables, the station map, and the loop.
Each station has its own deep-dive skill (below); load the one for your current stage.

---

## Always-on rules (never violated, on any task)

1. **Plan before you edit.** Read the relevant code and produce a plan before touching
   implementation. Premature patching is the single strongest predictor of failure.
2. **Tests first.** Each business rule and edge case gets a test that states what "done"
   means *before* the implementation exists. Red → green, not code-then-maybe-tests.
3. **Never bypass hooks.** No `git commit --no-verify`, no skipping CI or any enforced gate.
   A gate you walked around is a defect with a permission slip.
4. **Never weaken a gate to pass it.** Do not edit linter / formatter / type-checker / hook
   config, and do not delete, skip, or weaken a test, to make a check go green. Fix the code.
   If a rule is genuinely wrong, stop and surface it — do not silently loosen it.
5. **The Work Order is always human-owned.** *What* to build and what *"correct"* means is
   the human's call. Two hard approval gates enforce this: after the spec (Gate 1) and after
   the task plan (Gate 2).
6. **Isolate before you automate.** Never run unattended without confirming the Station 0
   isolation checklist (sandbox, least-privilege creds, no prod/deploy/write-API access, CI
   budget cap, rollback path). See `setup-project`.

---

## The line (stations, left = cheap to catch, expensive if missed)

```text
  STATION 0        STATION 1      STATION 2        STATION 3       STATION 4       STATION 5
  Isolate      →   Work Order  →  The Part      →  Andon Cord   →  Inspection  →   Field
  (sandbox)        (spec)         (TDD + plan)     (fail-hard      (narrow          (prod →
                                                    hooks)          review)          upstream)
  catches:         catches:       catches:         catches:        catches:         catches:
  runaway          "wrong         bad logic &      lint, types,    spec drift,      everything
  blast radius     thing"         behavior         secrets,        judgment         you missed
                                                    format          calls
  cost if missed:  catastrophic   high             low             medium           MAX
```

Field failures feed back **upstream**: every escaped defect becomes a new test or rule at the
station that should have caught it, so the line gets smarter over time.

---

## Station skills — load the one for your stage

| When you are... | Load skill | Station |
|---|---|---|
| Setting up a repo / confirming isolation + gates before any work | `setup-project` | 0 + 3 |
| Capturing a new requirement or idea into a spec | `intake-requirement` | 1 (Gate 1) |
| Designing the solution from an approved spec | `create-plan` | 2 (design) |
| Breaking a plan into TDD tasks + choosing GitHub/local | `break-into-tasks` | 2 (Gate 2) |
| Implementing ready tasks (TDD loop + hooks) | `implement-task-loop` | 2 + 3 |
| Opening a PR / reviewing / merging + loop control | `deliver` | 4 |
| Handling a defect that escaped to production | `field-returns` | 5 |

Pipeline order:

```text
setup-project → intake-requirement → create-plan → break-into-tasks
   → implement-task-loop ⇄ deliver   (loop PER TASK while ready tasks remain)
   → all tasks done
field-returns runs on escaped defects and feeds fixes back to setup / intake / tasks.
```

**The loop is per task, not batched.** `implement-task-loop ⇄ deliver` runs one task at a
time: implement a task → open *its* PR → human merges → pick the next ready task. Each task
gets its own PR the moment it's ready; you do **not** implement everything and deliver once.

**Keep context clean.** Each task runs in a clean context — ideally a sub-agent per task (its
own git worktree), or compaction between tasks — while the loop controller keeps only a
minimal ledger (the spec map + per-task status/PR). This prevents context fill and stops one
task's details from polluting the next.

**Not every job goes on the line.** The line is for well-defined *parts*. Greenfield
architecture, "I don't know what I want yet," or deep novel domain logic is **conductor
mode** — iterate turn-by-turn with the human instead of spec→delegate. `create-plan` decides
this explicitly and routes exploratory work to conductor mode first; only the well-defined
result comes back onto the line.

---

## Artifacts the line produces (in the target project)

```text
AGENTS.md                    # thin bootstrap — points at the manual + echoes the non-negotiables
harness.md                   # the project's operating manual + Project profile (from templates/)
docs/
  specs/<slug>/overview.md   # Station 1 — the spec map (intent + section index)
  specs/<slug>/<section>.md  # Station 1 — one mini Work Order per section
  plans/<slug>.md            # Station 2 — the design
  tasks/<id>.md              # Station 2 — one TDD task each
  field-returns.md           # Station 5 — escaped defects, upstream fixes, metrics
```

Bootstrap a new project by copying `templates/harness.md` (manual + Project profile) and
`templates/AGENTS.harness.md` (the always-loaded pointer + non-negotiables) into it, then run
`setup-project`. When operating in GitHub mode, `docs/tasks/<id>.md` files are mirrored to
issues (frontmatter → labels/metadata, body → issue body).
