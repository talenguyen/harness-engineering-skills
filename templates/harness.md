# Harness Engineering Line

## Rules (always in effect)

1. Plan before editing — read the relevant code and plan first, then implement.
2. Tests first — a failing test defines "done" before the implementation exists.
3. Never bypass hooks or CI.
4. Never weaken a gate to pass it — fix the code. If a gate is genuinely wrong, stop and ask the human.
5. Scope and acceptance criteria are human-owned. Stop for approval at Gate 1 (after the spec) and Gate 2 (after the task plan).
6. Confirm the Station 0 isolation checklist before any unattended run.

## Pipeline — load the skill for your current stage

| Stage | Load |
|---|---|
| Setup repo + install gates | `skills/setup-project.md` |
| Capture requirement → spec | `skills/intake-requirement.md` — Gate 1 |
| Design from approved spec | `skills/create-plan.md` |
| Break into TDD tasks | `skills/break-into-tasks.md` — Gate 2 |
| Implement one task (TDD + hooks) | `skills/implement-task-loop.md` |
| PR + human review + merge | `skills/deliver.md` |
| Escaped defect → upstream fix | `skills/field-returns.md` |

Order: setup → intake → plan → tasks → (implement ⇄ deliver, per task) → done.

## Project profile

<!-- project-profile:start -->
<!-- setup-project fills this in; install.sh preserves this block across re-installs. -->

**Detected stack:** _unset — run `skills/setup-project.md`_

**Station 0 isolation checklist:**
- [ ] Sandbox
- [ ] Least-privilege credentials
- [ ] No production / deploy / write-API access
- [ ] CI budget cap
- [ ] Rollback path tested

**Baseline owner:** _unset_
<!-- project-profile:end -->
