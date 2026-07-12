---
name: implement-task-loop
station: "2 + 3"
description: "Implements ready tasks test-first (red→green), self-corrects against fail-hard hooks, and advances task status. Use after Gate 2 to build approved tasks, enforcing the always-on non-negotiables."
requires:
  - "docs/tasks/<id>.md files with status: todo and an acyclic depends_on graph"
produces:
  - "implementation + tests per task; task status advanced to in_review"
enforces:
  - "`running-the-line` non-negotiables 1-4: plan-before-edit, tests-first, never-bypass-hooks, never-weaken-gate"
---

# Implement Task Loop — Station 2 (The Part) + Station 3 (Andon Cord)

Drive each ready task from red → green → hooks-green, updating status as you go. Tests catch
logic defects, hooks catch mechanical ones, and the agent self-corrects both without a human.

---

## Run each task in a clean context

Many tasks in one session cause two failures: the context window **fills up**, and earlier
tasks **pollute** the current one (stale assumptions, leaked names). Treat every task as a
clean room — the files are the memory (its `docs/tasks/<id>.md` + spec section + plan +
`harness.md`), so no task depends on the *transcript* of earlier tasks. Pick the strongest
option your runtime supports:

- **Sub-agent per task (best).** The controller stays tiny: read the map, pick a ready task,
  dispatch **one sub-agent** seeded only with that task file + its spec/plan + the code it
  touches (own worktree). It runs red→green→hooks→`in_review`→PR and returns a one-line
  summary (id, status, PR link). The controller records that line and never absorbs the
  sub-agent's working context.
- **Compact between tasks (fallback).** At each task's terminal state, drop its working
  details (diffs, test output) and keep a durable **ledger** — the map plus each task's
  `id → status → PR`. Re-read the next task's files fresh.

The controller keeps only that ledger, never accumulated transcripts. If a single task burns
more than ~200k tokens or the session turns sluggish, stop — the task was too vague or too
big; sharpen or split it. For a long task, commit work-in-progress and start a fresh session
that re-reads the map + task file + branch state; a clean context beats a stale one near capacity.

---

## Step 0 — Sync from GitHub first (GitHub mode only)

GitHub is the source of truth for merge state, and a human may have merged a PR while **no
agent was running**. Reconcile local task files with GitHub before selecting work, or a task
drifts (`in_review` forever) and you may re-implement something merged.

**Only check tasks with `status: in_review`** — the only ones with an open PR a human could
have merged. `todo` / `blocked` / `in_progress` have no PR; `done` is settled. If nothing is
`in_review`, skip this step.

For each `in_review` task, via its `issue:` number (or `T-<id>` label):

- PR **merged** → set `status: done`; clean up its worktree/branch.
- PR **closed unmerged** (rejected) → move back to `todo`/`in_progress` and note why.
- PR still **open** → leave as `in_review`.

```sh
gh pr list --search "T-002 in:title" --state merged --json number --jq '.[0].number'  # merged?
# if merged ⇒ set status: done in docs/tasks/T-002.md
```

(**Local mode:** skip — the files are already the source of truth.)

---

## Step 1 — Select ready tasks

A task is **ready** when its `status` is `todo` **and** every id in its `depends_on` is
`done`. A `todo` task with an unfinished dep is **blocked** — but that's a *derived* state you
compute here, not a status you write (the stored values are `todo | in_progress | in_review |
done`). If nothing is ready and nothing is in progress, the loop is complete — hand back to
`deliver.md` for final loop control.

---

## Step 2 — Serial vs parallel, and isolate each task

Default to **serial**. Parallelism usually moves the bottleneck onto the human (three PRs
land at once on one reviewer), so only fan out when *all* of these hold:

- every candidate task is `parallel_safe: true`, **and**
- their `touches` globs do **not** overlap (no shared files → no merge conflicts), **and**
- you stay under a fixed **concurrency cap** (e.g. 2–3), **and**
- the single-agent line has already been boringly reliable (parallel work piles up at Inspection).

If any condition fails, run serially — sequential is often faster because the bottleneck is
the human reviewer, not the agent. (This permits fan-out, which the source guide is cautious
about, but the four conditions keep it from creating a review pile-up.)

**Every task is built on its own branch `task/<id>-<slug>`, cut from the base branch at the
start of implementation** — never on `main`, never at delivery time.

- **Serial:** work on the task branch in the main working tree.
- **Parallel:** give each concurrent task its own **git worktree** for zero shared git state:

  ```sh
  git worktree add ../<repo>.wt/<id> -b task/<id>-<slug> <base-branch>
  ```

  Run the whole TDD + hooks loop *inside* that worktree. `touches` non-overlap prevents
  file-*content* conflicts; the worktree prevents git-*state* conflicts (HEAD / index /
  checkout) — the `touches` rule alone is not enough for genuinely concurrent agents.
  **Cold-start:** a fresh worktree shares the repo's git hooks (common git dir) and
  pre-commit's global cache — so don't re-run `pre-commit install` — but **not** the
  project's dependency/build state. Run install/build (`uv sync`, `npm install`) before
  tests and gates, or they fail spuriously. `deliver.md` pushes/opens the PR from the
  worktree, then removes it after merge.

---

## Step 3 — Per task: TDD red → green

Work from the task's mini Work Order body, on its own branch/worktree (Step 2):

1. **Plan before editing.** Read the code the task touches; restate what you'll change. No premature patching.
2. **Set `status: in_progress`** in the frontmatter.
3. **Write the tests first**, derived from the body: one per acceptance criterion, one per
   edge case, plus assertions that fail if an **off-limits** constraint is crossed. They must
   be **red** for the right reason (feature missing, not a typo).
4. **Implement the minimum** to turn them green — nothing beyond what the task needs.
5. **Green.** All the task's tests pass.

**Off-limits are hard constraints.** If passing a test seems to require crossing one
(forbidden file, banned dependency, frozen API), **stop and surface it** — never cross silently.

---

## Step 4 — Tests ≠ correctness (guard the intent)

Agents can satisfy a visible suite while missing its intent, and the gap grows on longer
tasks. The always-available guard is cheap: carry the question forward to Inspection —
*"did this satisfy the harness, or the intent?"*

Two **optional** techniques for non-trivial/safety-relevant work — use only if your harness
supports them, don't fake them:

- **Held-out tests.** If a separate reviewer/sub-agent writes the code, keep a few tests it
  can't see or edit (e.g. `tests/held_out/`, ignored by the implementing step) and run them
  at Inspection. In a single-agent session this offers little — rely on the question above.
- **Mutation testing** on critical paths — note *where* it's worth running; wire it in only
  if your stack has a mutation tool configured.

---

## Step 5 — Run the Andon Cord and self-correct

Run the pre-commit gates (format, lint net-new, type-check, secret scan) before committing.

- A hook failure prints a **precise** error. Read it, fix that exact thing, re-run, go green.
  This self-recovery — no human — is the difference between agents that finish and agents that stall.
- **Never** `--no-verify`; **never** edit hook/lint/type config or delete/skip a test to go
  green (non-negotiables 3 & 4). If a gate is genuinely wrong, stop and surface it.

---

## Step 6 — Hand this task to delivery (per task, not batched)

When *this* task's tests and hooks are green:

- Set `status: in_review`; in GitHub mode push the task branch and sync the status label.
- **Hand off to `deliver.md` for this task** — it opens the PR (Station 4, the human review
  gate). One finished task ⇒ one PR, immediately. Do **not** implement every task first and
  deliver once at the end.

While the PR awaits a human merge, you may start the next *independent* ready task (its own
worktree) — but this task is not `done`, and its dependants stay blocked, until a human merges it.

The loop is therefore one task at a time, alternating with `deliver.md`:

```text
pick ONE ready task → branch/worktree → tests red → implement → green → hooks green
   → status = in_review → deliver.md opens its PR → human merges → status = done
   → dependants unblock → next ready task
Exit only when no task is ready and none is in_review/awaiting-merge.
```

---

## Self-check (dry-run validation)

- [ ] GitHub mode: synced **only `in_review`** tasks from GitHub (merged PR → `done`) before selecting work.
- [ ] Correctly identifies ready vs blocked tasks from `depends_on` + `status`.
- [ ] Refuses to parallelize tasks with overlapping `touches` or any `parallel_safe: false`.
- [ ] Each task is on its own `task/<id>-<slug>` branch; parallel tasks each get their own worktree.
- [ ] Each task runs in a clean context; the controller keeps only a minimal ledger.
- [ ] Walks one task red → green: tests written first, failing for the right reason.
- [ ] Enforces off-limits; runs hooks and self-corrects without weakening any gate.
- [ ] Flips `status` to `in_review` and hands the task to `deliver.md` (does not batch).

---

## Handoff

When a task reaches `in_review`, proceed to **`deliver`** (Station 4), then loop
back here while ready tasks remain.
