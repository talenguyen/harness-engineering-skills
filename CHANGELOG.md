# Changelog

Design decisions and rationale for the Harness Engineering line, newest first. Keep entries
short: *what changed* and *why*. This exists so the skill set evolves deliberately instead of
accreting ad-hoc.

## Consolidation pass
- **Consolidated the skills; no rules removed.** Deduplicated statements that had drifted
  across files (per-task loop, clean-context, conductor mode now single-sourced with 1-line
  pointers), merged `implement-task-loop`'s duplicate "How the loop runs" section into Step 6,
  and cut reactive/patch-note phrasing. *Why:* turn-by-turn additive edits had started to
  accrete; this restores a designed, single-source-of-truth shape. Worked examples and
  multi-stack gate configs were **kept** — they're reference that belongs in the skill.

## Borrowed from the "v2" tactical guide
- **`break-into-tasks`: "Write what, not how" guard.** Task bodies are Work Orders (behavior),
  not implementation specs — no signatures/algorithms/private names/internal shapes as
  requirements; `## Interface contract (fixed)` escape hatch for real public seams; litmus
  test. *Why:* tasks were drifting into `how`, becoming unreadable and over-constraining.
- **`create-plan`:** design detail is guidance (not copied into task requirements) + a
  "when TDD doesn't fit → verification checklist" table.
- **`intake-requirement`:** added a `Context` field to the section Work Order.
- **`implement-task-loop`:** runaway-token flag (>~200k = spec too vague) + context-reset pattern.
- **`deliver`:** PR body gains "How to test" + docs/breaking-change checklist; doc-sync step
  after merge; two review smells (fix touches callers not root cause; multiple approaches in one file).
- **`setup-project`:** AGENTS.md authoring principles (only non-inferable, human-written, lean).

## Core line
- **Five stations + two human gates** (spec, TDD plan), from `running-the-line.md`. *Why:* move
  each defect's catch as far left as possible.
- **AGENTS.md (thin pointer) + harness.md (manual) split.** *Why:* AGENTS.md is the file most
  likely to already exist in a target repo; a small marker-delimited bootstrap block merges
  with near-zero conflict, while the full manual lives in harness.md.
- **Project profile block** in harness.md, preserved by `install.sh` across upgrades. *Why:*
  re-installing skills must not clobber a project's detected stack / Station 0 checklist / owner.
- **map + sections intake.** Resumable spec (overview map + per-section Work Orders); files are
  the memory. *Why:* long brainstorming sessions must survive context loss and be resumable.
- **Per-task branch + git worktree isolation.** *Why:* `touches` non-overlap prevents content
  conflicts; a worktree prevents git-state conflicts for genuinely concurrent tasks.
- **Per-task PR + human-merge gate** (no self-merge by default). *Why:* one PR per task keeps
  review small and rigorous; shipping stays human-owned (rule 5).
- **GitHub task↔issue sync.** `Closes #<issue>` auto-closes on merge; a Station-0 sync step
  reconciles `in_review` tasks from GitHub for async web-UI merges.
- **Clean-context strategy** (sub-agent per task, or compact between tasks; minimal ledger).
- **`tools/check-line.sh` self-check.** Validates wiring (skills referenced, cross-refs resolve,
  AGENTS.md points to harness.md, scaffold + field-returns present) and task frontmatter +
  acyclic `depends_on` graph.
