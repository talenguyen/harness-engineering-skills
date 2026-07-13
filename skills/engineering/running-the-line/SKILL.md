---
name: running-the-line
description: "Orchestrates one coding task end-to-end through the Harness Engineering line — spec → TDD plan → fail-hard gates → per-task PR/review → field feedback. Use when the user wants to run the line or ship a feature safely through spec, tests, gates, and review."
---

# Running the Line

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
| Setup repo + install gates | `setup-project` |
| Capture requirement → spec | `intake-requirement` — Gate 1 |
| Design from approved spec | `create-plan` |
| Break into TDD tasks | `break-into-tasks` — Gate 2 |
| Implement one task (TDD + hooks) | `implement-task-loop` |
| PR + human review + merge | `deliver` |
| Escaped defect → upstream fix | `field-returns` |

Order: setup → intake → plan → tasks → (implement ⇄ deliver, per task) → done.

## Bootstrap a new project

Copy `templates/harness.md` into the project root (the operating manual + project profile),
and `templates/AGENTS.harness.md` into its `AGENTS.md`. Then run `setup-project`.
