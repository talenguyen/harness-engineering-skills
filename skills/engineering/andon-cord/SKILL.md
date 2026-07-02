---
name: andon-cord
description: 'Design, install, or use hard-failing quality gates for AI coding workflows. Use when the user needs repo-specific test/lint/type/secret gates, pre-commit hooks, ratcheting for legacy violations, or agent recovery rules that prevent bypassing checks.'
---

# Andon Cord

The andon cord is the stop-the-line mechanism. It only works when gates are real
commands, fail hard, and produce errors the agent can act on.

## Audit Existing Gates First

Before recommending new tooling, inspect:

- Package scripts, Make/Just targets, CI workflows
- Existing pre-commit/Husky/lint-staged config
- Test runner, type checker, linter/formatter, secret scanner
- Known noisy or legacy checks

Do not invent commands. If a command is missing, mark it missing and recommend a
next step.

## Gate Map

Produce a map based on discovered facts:

```markdown
| Gate | Command | Source | Exists? | Runs when | Defect caught | Failure action |
|---|---|---|---|---|---|---|
```

Every existing command needs a source file. Every missing gate needs a reason it
matters.

## Ratchet Strategy

For legacy repos, do not flip every rule to error on day one. Recommend a
ratchet:

1. Baseline existing violations.
2. Gate new or modified code first.
3. Tighten one category at a time.
4. Keep the command fast enough that agents will run it repeatedly.

## Agent Recovery Rule

When a gate fails:

1. Read the exact error.
2. Fix the specific defect.
3. Re-run the same gate.
4. Proceed only when green.

Never use no-verify flags. Never edit lint/type/test/hook config to make the
current task pass. Never delete or weaken tests to get green. If a config change
is legitimate, split it into a separate explicit change with its own rationale.

## Output

End with:

- Gate map
- Missing gates
- Recommended ratchet
- Exact agent rules to add
- Commands verified, or commands not run and why
