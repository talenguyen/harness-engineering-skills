---
name: setup-project
description: "Confirms isolation, detects the stack, and installs ratcheted fail-hard gates (format, lint, type-check, secrets). Use before any feature work or before the first unattended run."
---

# Setup Project

Do these steps in order. Do not start feature work until all are done.

## 1. Isolation checklist

Fill in the `harness.md` Project profile block. For each item, write **concrete evidence**
(not just "done") and date it today. Leave unchecked anything you cannot verify.

- [ ] Sandbox — where is the agent running?
- [ ] Least-privilege credentials — what scope, what tokens?
- [ ] No production / deploy / write-API access — confirm no write-capable key for payments, brokerage, messaging, databases, or deploy targets is reachable.
- [ ] CI budget cap — what ceiling?
- [ ] Rollback path tested — how do you revert? (If unsolved, say so.)

Do not run unattended until every box is checked.

## 2. Detect the stack

Look at manifests (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, etc.).
Record in `harness.md` Project profile:

```
**Detected stack:** <language> (<pkg manager>) · test: <runner> · gates: <tools>
**Baseline owner:** <person>
```

Prefer tools the repo already uses. Assign a real baseline owner.

## 3. Install fail-hard gates

Four categories — pick the idiomatic tool for this stack. Every gate must **fail hard** (not warn).

| Category | Behavior |
|---|---|
| Formatter | auto-fix + fail if anything changed |
| Linter | net-new/modified code only (ratchet) |
| Type-check | fail hard |
| Secret scan | fail hard, whole diff |

Use a gate runner (e.g. `pre-commit`). Pin real tags. Run `pre-commit install`.

For config examples per stack, see [reference/gate-examples.md](reference/gate-examples.md).

## 4. Ratchet existing violations

If the repo has history with existing violations:

1. Baseline them (lint suppress file, `.gitleaksignore`, etc.).
2. Gate only net-new and modified code.
3. Tighten one category per week.

## 5. Mirror gates in CI + enable branch protection

Run the same gates server-side on PRs. Enable branch protection on `main` (require PR,
require checks, disallow direct push). This is the real enforcement boundary — local hooks
are bypassable.

## Done when

- Every isolation box checked with evidence.
- `harness.md` has detected stack + baseline owner.
- A clean commit passes hooks; a net-new violation is blocked; baselined violations don't block.
- CI runs the same gates; branch protection requires them.

Next: `intake-requirement`
