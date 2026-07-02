---
name: andon-cord
description: Design, install, or use hard-failing quality gates for AI coding workflows. Use when the user needs pre-commit hooks, lint/type/test/secret gates, ratcheting for legacy repos, gate failure recovery, or rules that stop agents from bypassing or weakening checks.
---

# Andon Cord

The andon cord is the automatic stop-the-line station. It catches mechanical
defects early and gives the agent a precise failure to self-correct.

## Gate principles

- A gate must fail hard. Warnings are not gates.
- A gate should print an error precise enough for the agent to fix.
- Start from the repo's existing commands and CI.
- For legacy repos with many existing violations, use a ratchet: gate new or
  modified code first, then tighten over time.
- Keep perishable tool versions and command flags in config or reference blocks,
  not in durable prose.

## Gate map

Produce or update a map like this:

```markdown
| Gate | Command | Defect class | Runs when | Failure action |
|---|---|---|---|---|
| Tests | [project command] | Behavior regressions | before PR / pre-push | Fix failing test or revise work order |
| Type check | [project command] | Type contract defects | before commit / CI | Fix exact type error |
| Lint/format | [project command] | Mechanical consistency | before commit | Apply formatter or fix lint |
| Secret scan | [project command] | Credential leakage | before commit / CI | Remove secret and rotate if exposed |
```

## Agent failure rule

When a gate fails:

1. Read the error.
2. Fix the exact defect.
3. Re-run the same gate.
4. Proceed only when green.

Never bypass hooks. Never use no-verify flags. Never edit lint, type, test, or
hook configuration just to make the current change pass. If config genuinely
needs to change, make that a separate, explicit change with its own reason.

## Setup posture

Do not install every possible tool on day one. Install the smallest gate set
that catches the repo's current leaks. A fast, reliable gate that runs every
time is better than a comprehensive gate nobody uses.

