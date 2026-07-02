# Harness Engineering Skills

A portable guide and installable skill pack for AI coding agents, built around
[Running the Line](./running-the-line.md).

The premise is simple: your output is no longer the code. It is the line that
produces the code. A good line defines correctness before implementation,
tests the part, stops on mechanical defects, narrows human review, and feeds
production failures back upstream.

Install with:

```bash
npx skills@latest add talenguyen/harness-engineering-skills
```

This repo is meant to be grabbed by working engineers and adapted into their
own agent setup. It favors small, composable skills over a monolithic process.
The structure follows the public `mattpocock/skills` convention: a readable
README, a `.claude-plugin/plugin.json` manifest, a `skills/` tree, and one
`SKILL.md` per skill.

## What is here

```text
.claude-plugin/plugin.json Installer manifest for the skills CLI
running-the-line.md        The durable guide and mental model
skills/engineering/        Installable agent skills
templates/                 Copyable artifacts for real repos
evals/evals.json           First-pass eval prompts for skill testing
```

## Skill Map

### User-invoked

These are slash-command style skills. The human chooses when to run them.

| Skill | Use it when |
|---|---|
| [`setup-harness-engineering`](./skills/engineering/setup-harness-engineering/SKILL.md) | You want to prepare a repo for the line: rules, docs, gates, and review habits |
| [`running-the-line`](./skills/engineering/running-the-line/SKILL.md) | You have one concrete coding task and want the full spec -> plan -> gate -> review flow |

### Model-invoked

These are reusable station skills. The model may invoke them automatically when
the task fits.

| Skill | Use it when |
|---|---|
| [`work-order`](./skills/engineering/work-order/SKILL.md) | A task needs crisp business rules, acceptance criteria, edge cases, and off-limits |
| [`tdd-plan-gate`](./skills/engineering/tdd-plan-gate/SKILL.md) | A work order exists and the agent must produce a TDD plan before implementation |
| [`andon-cord`](./skills/engineering/andon-cord/SKILL.md) | You need hooks, checks, or ratcheted gates that fail hard and let agents self-correct |
| [`inspection`](./skills/engineering/inspection/SKILL.md) | A diff or PR needs review against the work order and human judgment calls |
| [`field-return`](./skills/engineering/field-return/SKILL.md) | A production bug or escaped defect must become an upstream test, rule, or gate |

## Quickstart

1. Install the repo:
   ```bash
   npx skills@latest add talenguyen/harness-engineering-skills
   ```
2. Read [Running the Line](./running-the-line.md) once. It explains the five
   stations and the cost model.
3. Run `/setup-harness-engineering` in a target repo to create or update agent
   rules, station docs, and the first gate map.
4. Pick one small real task. Invoke `/running-the-line`:
   "run this task down the line; start with the work order and do not implement
   until the TDD plan is approved."
5. Keep only what earns its place. If your repo leaks at Station 1, install only
   `work-order`. If it leaks at Station 3, install only `andon-cord`.

## Design Rules

- Durable reasoning belongs in the guide. Perishable commands, versions, flags,
  and tool names belong in clearly marked reference blocks or templates.
- No borrowed authority. If a claim needs a number, study, or vendor statistic,
  verify it before publishing or cut it.
- Skills should stop agents at the right moments. The most important stop is
  before implementation: the work order and TDD plan must be reviewable.
- A skill should make the workflow easier to run, not harder to understand.
  The user still owns what to build and what "correct" means.

## Non-goals

This is not a promise that agents can replace engineering judgment. It is not a
universal workflow for R&D, ambiguous product discovery, or architecture you do
not understand. The line is for well-defined parts. Exploratory work should be
done interactively first, then turned into work orders once the shape is known.

