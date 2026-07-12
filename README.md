# Harness Engineering Skills

A portable guide and installable skill pack for AI coding agents, built around
[Running the Line](./running-the-line.md).

The premise is simple: your output is no longer the code. It is the line that produces the
code. A good line defines correctness before implementation, tests the part, stops on
mechanical defects, narrows human review, and feeds production failures back upstream.

Install with:

```bash
npx skills@latest add talenguyen/harness-engineering-skills
```

This repo is meant to be grabbed by working engineers and adapted into their own agent
setup. It favors small, composable skills over a monolithic process, following the public
`mattpocock/skills` convention: a readable README, a `.claude-plugin/plugin.json` manifest,
a `skills/` tree with one `SKILL.md` per skill, and copyable `templates/`.

## What is here

```text
.claude-plugin/plugin.json  Installer manifest for the skills CLI
running-the-line.md         The durable guide and mental model
skills/engineering/         Installable agent skills (1 orchestrator + setup + 6 stations)
templates/                  Copyable artifacts: project manual, AGENTS bootstrap, spec map,
                            work order, plan, task, inspection notes, field-return log
evals/evals.json            First-pass eval prompts for skill testing
```

## Skill map

### Orchestrator + setup

| Skill | Use it when |
|---|---|
| [`running-the-line`](./skills/engineering/running-the-line/SKILL.md) | You have one concrete task and want the full flow: spec → TDD plan → gates → per-task PR → field feedback |
| [`setup-project`](./skills/engineering/setup-project/SKILL.md) | You want to prepare a repo: isolation checklist, stack detection, ratcheted fail-hard gates |

### Station skills

| Skill | Station | Use it when |
|---|---|---|
| [`intake-requirement`](./skills/engineering/intake-requirement/SKILL.md) | 1 | Understand what the user truly wants → a resumable spec map + per-section Work Orders (Gate 1) |
| [`create-plan`](./skills/engineering/create-plan/SKILL.md) | 2 | Design from an approved spec, trace every acceptance criterion, route R&D to conductor mode |
| [`break-into-tasks`](./skills/engineering/break-into-tasks/SKILL.md) | 2 | TDD tasks with an acyclic dependency graph; GitHub/local choice (Gate 2, reject-first) |
| [`implement-task-loop`](./skills/engineering/implement-task-loop/SKILL.md) | 2 + 3 | TDD red→green per task, own branch/worktree, clean context, Andon self-correction |
| [`deliver`](./skills/engineering/deliver/SKILL.md) | 4 | Narrow inspection + one PR per task (human-merge gate); loop control |
| [`field-returns`](./skills/engineering/field-returns/SKILL.md) | 5 | Turn an escaped defect into the upstream test/rule that would have caught it |

## Quickstart

1. Install: `npx skills@latest add talenguyen/harness-engineering-skills`.
2. Read [Running the Line](./running-the-line.md) once — the five stations and the cost model.
3. Bootstrap a target repo: copy `templates/harness.md` into it and `templates/AGENTS.harness.md`
   into its `AGENTS.md`, then run `setup-project` to detect the stack and confirm isolation.
4. Pick one small real task and run `running-the-line`: "run this down the line; start with the
   spec and do not implement until the TDD plan is approved."
5. Keep only what earns its place. Leaking at Station 1? Lean on `intake-requirement`. Leaking
   at Station 3? Lean on `setup-project`'s gates.

## Design rules

- Durable reasoning belongs in the guide; perishable commands/versions/tool names belong in
  clearly marked reference blocks or templates.
- No borrowed authority. If a claim needs a number or study, verify it before publishing or cut it.
- Skills should stop agents at the right moments. The most important stops are the two human
  gates: after the spec, and after the TDD task plan.
- The user still owns what to build and what "correct" means.

## Non-goals

Not a promise that agents replace engineering judgment, and not a workflow for R&D, ambiguous
product discovery, or architecture you do not understand. The line is for well-defined parts;
explore interactively first (conductor mode), then put the known shape on the line.
