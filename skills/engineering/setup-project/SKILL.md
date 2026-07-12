---
name: setup-project
station: "0 + 3"
description: "Sets up a repo's safety and quality gates: the Station 0 isolation checklist plus ratcheted, fail-hard pre-commit hooks and server-side enforcement. Use when initializing a project or before the first unattended agent run, ahead of any feature work."
produces:
  - "harness.md 'Detected stack' section filled in"
  - ".pre-commit-config.yaml (or stack-equivalent) with fail-hard, ratcheted gates"
  - "CI workflow running the same gates + branch protection note"
  - "Station 0 isolation checklist, signed off"
blocks_until: "isolation checklist is satisfied; you may not run unattended before that"
---

# Setup Project — Station 0 (Isolate) + Station 3 (Andon Cord)

Two jobs, both before any feature work:

1. **Station 0 — Isolate before you automate.** The whole "hand off and step away"
   model assumes the agent physically *cannot* do catastrophic damage while unwatched.
   That assumption gets a gate, not a hope.
2. **Station 3 — The Andon Cord.** Install fast, fail-hard local gates (format, lint,
   type-check, secret scan) so mechanical defects die in seconds without a human. Back
   them with server-side CI + branch protection, which is the *real* enforcement boundary.

> A hook is fast local feedback. A hook is **not** the enforcement boundary — `git commit
> --no-verify` skips it and an agent can edit the hook config. The boundary is server-side.

---

## Step 1 — Station 0 isolation checklist (blocks unattended runs)

Confirm every box before letting an agent run without a human watching. If any box
can't be checked, **build it before Station 3**, not after. Record the result in the
**Project profile** section of `harness.md` (the `<!-- project-profile -->` block). If the
project has no `harness.md` yet, copy `templates/harness.md` into the project root first
(and `templates/AGENTS.harness.md` into `AGENTS.md`) — they ship the manual plus an empty
Project profile.

**Honesty rule — this checklist is worthless if faked:**
- **Verify each box yourself before checking it.** A checked box you didn't confirm is
  worse than an empty one — it manufactures false confidence. If you can't verify it, leave
  it unchecked and say so.
- **Write concrete evidence next to each box** (the actual sandbox, the exact credential
  scope, the command you used to confirm rollback), not "done".
- **Date the checklist with today's real date.** A stale date signals it was never reviewed.

- [ ] **Sandbox.** Agent runs in a container / VM / ephemeral workspace, not on a box
      with ambient access to anything that matters.
- [ ] **Least-privilege credentials.** Tokens are scoped to exactly this repo/task.
      No org-admin tokens, no shared prod creds sitting in the environment.
- [ ] **No direct production / write access.** No prod database connection string, no
      deploy keys, no `kubectl` context pointing at prod — and, for anything that acts on
      the world (payments, brokerage/trading, messaging), no *write*-capable API key
      reachable from the agent's shell. Read-only is the default; confirm it.
- [ ] **CI budget cap.** A hard ceiling on CI minutes / job count so a runaway loop
      can't rack up unbounded jobs (or spend).
- [ ] **Rollback path exists and is tested.** You can revert a merge/deploy. For CDN
      configs, ASO pipelines, LiveOps automations, and similar, rollback is often *not*
      solved — confirm it explicitly rather than assuming it.

> Why this is Station 0 and not optional: a real 2025 incident had an AI coding agent
> delete a production database mid-task, reportedly ignoring an explicit freeze. Isolation
> is the gate that makes every later "step away" instruction safe.

---

## Step 2 — Detect the stack

Never assume the stack. Detect it, then write it into the **Project profile** section of
`harness.md`.

1. Look for manifests: `package.json`, `pyproject.toml` / `requirements.txt`, `go.mod`,
   `Cargo.toml`, `pom.xml` / `build.gradle`, `composer.json`, `Gemfile`, etc.
2. Identify: language(s), package manager, test runner, and existing lint/format/type tools.
3. Prefer tools the repo already uses over introducing new ones.
4. Record findings in the `harness.md` Project profile block (replace the placeholder):

```markdown
**Detected stack.** TypeScript (pnpm) · test: vitest · gates: biome (format+lint),
tsc --noEmit (types), gitleaks (secrets).
```

Assign the **baseline owner** in the same block (a real person, not "TBD") — an unowned
baseline drifts and quietly stops being trusted.

### Writing the project's agent rules (`AGENTS.md`) — keep it lean and human-owned

`AGENTS.md` is loaded every session, so it costs tokens every time and gets ignored when it
bloats. Write it by hand and keep it short:

- **Only what the agent cannot infer from the code.** Custom build/test commands,
  counterintuitive patterns, and permission boundaries are gold. Architecture overviews and
  "follow best practices" are noise — the agent can read the code.
- **Write it yourself, don't have the agent generate it.** An agent can't describe what it
  doesn't know; auto-generated rules tend to help less than none at all.
- **Keep it lean.** Rule quality degrades as the file grows; push detail into referenced
  docs (`@docs/build-commands.md`, `@docs/conventions.md`) via progressive disclosure.
- **Ask for evidence, not screenshots.** Require test output / grep results as proof of a
  claim, not images — screenshots waste context.
- **Update it when the agent repeats a mistake** — that's the signal a rule is missing.

---

## Step 3 — Choose gate tools (language-agnostic, one per category)

Every stack needs the same four gate categories. Pick the idiomatic tool for the
detected stack. Any check you expect the agent to self-correct **must fail hard, not
warn** — agents walk past warnings.

| Category | What it catches | Rule |
|---|---|---|
| Formatter | style / whitespace churn | auto-fix + fail if it had to change anything |
| Linter | bug-prone patterns, anti-patterns | **net-new + modified code only** (ratchet) |
| Type-check | type mismatches | fail hard |
| Secret scan | committed credentials | fail hard, whole-diff |

Optional same-station gates when available: dependency-vulnerability scan, static
security analysis (SAST). If a tool can emit a precise "fail" the agent can self-fix
from, it belongs here.

---

## Step 4 — Install the runner with a fail-hard config

A local gate runner (e.g. `pre-commit`) bundles all four categories behind one command.
`pre-commit` itself runs on Python, so on a pure JS/TS repo it's a new runtime dependency —
budget ~30–60 min on setup day. Pin every tool to a real released tag; check each repo's
latest tag before copying a version.

### Example — TypeScript / JavaScript

```yaml
# .pre-commit-config.yaml  (pins are placeholders — look up current tags)
repos:
  - repo: https://github.com/biomejs/pre-commit
    rev: vX.Y.Z
    hooks:
      - id: biome-check          # format + lint
        args: ["--write"]
  - repo: https://github.com/gitleaks/gitleaks
    rev: vX.Y.Z
    hooks:
      - id: gitleaks
  - repo: local
    hooks:
      - id: tsc
        name: type-check
        entry: pnpm tsc --noEmit
        language: system
        pass_filenames: false
```

### Example — Python

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: vX.Y.Z
    hooks:
      - id: ruff                 # lint  (add --fix for autofix)
      - id: ruff-format          # format
  - repo: https://github.com/gitleaks/gitleaks
    rev: vX.Y.Z
    hooks:
      - id: gitleaks
  - repo: local
    hooks:
      - id: mypy
        name: type-check
        entry: mypy .
        language: system
        pass_filenames: false
```

### Example — Go

```yaml
repos:
  - repo: https://github.com/golangci/golangci-lint
    rev: vX.Y.Z
    hooks:
      - id: golangci-lint        # lint + vet (fail hard)
  - repo: https://github.com/gitleaks/gitleaks
    rev: vX.Y.Z
    hooks:
      - id: gitleaks
  - repo: local
    hooks:
      - id: gofmt
        name: format check
        entry: gofmt -l -w .
        language: system
        pass_filenames: false
```

Install so it runs on every commit: `pre-commit install`.

---

## Step 5 — Ratchet, don't big-bang (the part other guides skip)

You **cannot** flip every rule to "error" on day one in a repo with history. You'll block
every commit on thousands of pre-existing violations and the team rips the hooks out by
lunch. Do the ratchet instead:

1. **Baseline** existing violations and ignore them (most linters emit a baseline/suppress
   file, e.g. a lint baseline, `.gitleaksignore`, `# type: ignore` audit list).
2. **Gate net-new and modified code only.** `pre-commit` already runs on changed files
   by default; keep linter scope to the diff.
3. **Tighten one rule category per week** as you clean up legacy.
4. **Assign a baseline owner** and review the baseline monthly — it drifts as code is
   moved and renamed, and an unowned baseline quietly stops being trusted. Record the
   owner in `harness.md` (alongside the Detected stack).

The line tightens over a month, not an afternoon. Anyone who says start with all-errors
has never done it on a codebase with history.

---

## Step 6 — Server-side enforcement (the real boundary)

Local hooks are bypassable. Mirror the *same* gates in CI and turn on branch protection
so nothing local can route around them.

```yaml
# .github/workflows/gates.yml  (illustrative)
name: gates
on: [pull_request]
jobs:
  gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pipx run pre-commit run --from-ref origin/${{ github.base_ref }} --to-ref HEAD
      # plus: test runner, and any gate not expressible as a pre-commit hook
```

Then, in branch protection for `main`: require this check to pass, require PR review,
disallow direct pushes, and (ideally) disallow force-push. **CI + branch protection is
the enforcement boundary; hooks are the fast feedback that keeps the agent unblocked.**

---

## Anti-pattern to forbid explicitly

Some harnesses, stuck on a stubborn check, "fix" it by editing `biome.json`, loosening
`tsconfig`, deleting the failing test, or committing with `--no-verify`. That is
optimizing for green by dismantling the station. This is already banned by harness.md
non-negotiables 3 and 4 — restate it in review if you see it.

---

## Self-check (dry-run validation)

Run against a throwaway repo that has pre-existing violations:

- [ ] Station 0 checklist is filled in; no unchecked box remains before unattended runs.
- [ ] `harness.md` Detected stack section is populated (stack, tools, baseline owner).
- [ ] A trivial clean commit **passes** the hooks.
- [ ] A commit that adds a **net-new** violation (bad format / lint error / type error /
      a fake secret) is **blocked** with a precise message.
- [ ] Pre-existing (baselined) violations do **not** block commits.
- [ ] CI runs the same gates and `main` branch protection requires them.

If all boxes hold, Station 0 + 3 are live.

---

## Handoff

Gates are up and the sandbox is confirmed. Proceed to **`intake-requirement`**
(Station 1) to turn the first requirement into a Work Order.
