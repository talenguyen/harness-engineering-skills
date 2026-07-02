---
name: setup-harness-engineering
description: 'Set up a real repository to use Harness Engineering. Use when the user wants a useful AI-agent workflow, AGENTS.md/CLAUDE.md rules, build/test/lint command docs, quality gates, or a readiness audit before letting agents code.'
---

# Setup Harness Engineering

This skill prepares a repo for guarded agent handoff. It must not spray generic
files into the project. A useful harness is made of repo facts, working commands,
and explicit boundaries. Empty templates are worse than no setup because they
teach the agent to trust meaningless docs.

## Non-negotiable Quality Bar

Do not create placeholder files. Do not create `docs/harness/` just because this
skill is named harness engineering. Do not write an AGENTS.md full of generic
rules like "follow best practices" or blank fields like `Test command:`.

Only write an artifact when it contains project-specific information discovered
from the repo or explicitly provided by the user.

If the repo lacks enough facts to build a harness, produce a Week 0 readiness
report instead of pretending setup is complete.

## Step 1: Audit Before Editing

Inspect the repo and collect evidence. Prefer existing files and commands over
guesses.

Look for:

- Existing agent rules: `AGENTS.md`, `CLAUDE.md`, `.cursor/rules`, `.github/copilot-instructions.md`
- Package/build files: `package.json`, `pnpm-lock.yaml`, `bun.lock`, `Cargo.toml`,
  `pyproject.toml`, `go.mod`, `pom.xml`, `build.gradle`, `Makefile`, `justfile`
- Test/lint/type commands from scripts, CI, Make/Just targets, README
- Existing gates: `.pre-commit-config.yaml`, Husky hooks, lint-staged, CI workflows
- Project conventions the agent cannot infer: package manager, banned APIs,
  generated files, env/secret rules, architectural boundaries, test seams
- Current failure modes: broken tests, missing commands, noisy lint, no CI, no
  testable seams

Do not run slow or mutating commands during audit unless the user asked. It is
fine to read config and run harmless list/version commands.

## Step 2: Classify Readiness

Classify the repo honestly:

| Level | Meaning | Setup action |
|---|---|---|
| Week 0 | Tests/lint/build are missing or cannot be identified | Produce readiness report and first fixes; do not claim harness is installed |
| Basic | Commands exist, but no agent rules or gates | Add concise rules and command docs |
| Guarded | Rules and some gates exist | Tighten failure policy, fill missing command docs, add gate map |
| Handoff-ready | Spec -> TDD plan -> gates -> PR is already plausible | Add only the missing workflow glue |

If the repo is Week 0, the correct output is a prioritized setup backlog, not a
pile of files.

## Step 3: Design Progressive Disclosure

Root agent rules should stay short. They should point to sub-docs only when
those sub-docs contain useful details.

Preferred layout:

```text
AGENTS.md or CLAUDE.md          <= 60 lines; stack, always/never, hook policy
docs/build-commands.md          exact commands and when to run them
docs/testing-patterns.md        only if repo-specific test seams/patterns exist
docs/conventions.md             only if there are counterintuitive repo rules
docs/permissions.md             only if ask-first/allowed actions need clarity
```

Do not create all four docs by default. Create the smallest useful set.

## Step 4: Artifact Standards

An artifact is allowed only if it meets the relevant standard.

### Agent Rules

Write or patch `AGENTS.md`/`CLAUDE.md` only with facts like:

- Package manager and exact commands discovered from repo config
- "Always" rules tied to repo behavior
- "Never" rules that prevent known agent damage
- Hook failure loop
- Links to sub-docs that exist and contain useful facts

Reject:

- Architecture summaries the agent can infer by reading code
- Empty stack fields
- Generic advice
- Tool commands not verified from the repo

### Build Commands Doc

Create this only if you can list exact commands and their source.

Required table:

```markdown
| Purpose | Command | Source | Notes |
|---|---|---|---|
| Test | pnpm test | package.json | Fast enough for local use |
```

### Gate Map

Create a gate map only from real commands or clearly marked missing gates.

```markdown
| Gate | Command | Exists? | Runs when | Defect caught | Failure action |
|---|---|---|---|---|---|
```

Missing gates belong in the report/backlog, not as fake config.

## Step 5: Present Patch Plan Before Writing

Before editing, show:

- Readiness level
- Facts discovered, with file sources
- Files proposed for create/update
- Files deliberately not created and why
- Commands to verify after edits

If the user explicitly asked for autonomous setup, proceed after presenting the
plan. Otherwise ask for confirmation before writing.

## Step 6: Edit Conservatively

When editing:

- Preserve existing user rules.
- Update an existing agent-rules section in place rather than appending duplicates.
- Keep root rules short.
- Prefer patching existing docs over creating parallel docs.
- Do not delete old meaningless harness files unless the user explicitly asks;
  instead list cleanup recommendations.

## Final Output

End with a concrete report:

```markdown
## Harness Readiness
[Week 0 / Basic / Guarded / Handoff-ready]

## Files Changed
- [path] - [why this file earns its place]

## Gate Map
- [real command] - [what it catches] - [status]

## Not Created
- [file] - [why it would have been empty or premature]

## Week 0 / Next Backlog
- [specific missing command, gate, convention, or testability issue]

## First Task To Run
[small real task that can exercise the line]
```
