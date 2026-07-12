# Engineering Skills

Harness Engineering skills for AI coding agents — a five-station "production line" that
turns intent into shipped, verified code and kills each defect at the cheapest station.
Install the whole set, or load only the station your workflow needs.

## Orchestrator

| Skill | Role |
|---|---|
| [`running-the-line`](./running-the-line/SKILL.md) | The manual: non-negotiables, station map, and the per-task loop |
| [`setup-project`](./setup-project/SKILL.md) | Station 0 + 3 — isolation checklist, stack detection, ratcheted fail-hard gates |

## Station skills

| Skill | Station | Role |
|---|---|---|
| [`intake-requirement`](./intake-requirement/SKILL.md) | 1 | Understand true intent → a resumable spec map + per-section Work Orders (Gate 1) |
| [`create-plan`](./create-plan/SKILL.md) | 2 | Design from an approved spec; trace every acceptance criterion; route R&D to conductor mode |
| [`break-into-tasks`](./break-into-tasks/SKILL.md) | 2 | TDD tasks with an acyclic dependency graph; GitHub/local choice (Gate 2, reject-first) |
| [`implement-task-loop`](./implement-task-loop/SKILL.md) | 2 + 3 | TDD red→green per task, own branch/worktree, clean context, Andon self-correction |
| [`deliver`](./deliver/SKILL.md) | 4 | Narrow inspection + one PR per task (human-merge gate); loop control |
| [`field-returns`](./field-returns/SKILL.md) | 5 | Classify an escaped defect → add the upstream check; track metrics |

Start with `running-the-line` for a full task, or `setup-project` to prepare a repo first.
Bootstrap a project with the files in [`../../templates`](../../templates).
