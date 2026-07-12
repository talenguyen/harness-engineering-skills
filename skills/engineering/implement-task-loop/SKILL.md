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

Pick up ready tasks and drive each one from red to green to hooks-green, updating status
as you go. This is where TDD and the Andon Cord do their work: tests catch logic defects,
hooks catch mechanical ones, and the agent self-corrects both without a human.

---

## Run each task in a clean context (avoid fill + cross-task pollution)

Running many tasks in one long session causes two failures: the context window **fills up**,
and earlier tasks' details **pollute** the current one (stale assumptions, leaked names,
focusing on the wrong files). Prevent both by treating every task as a **clean room**. The
files are the memory — a task's `docs/tasks/<id>.md`, its spec section(s), the plan, and
`harness.md` hold everything needed — so no task should depend on the *transcript* of
previous tasks.

Pick the strongest option your runtime supports:

1. **Best — delegate each task to a sub-agent.** The loop controller stays tiny: read the
   map, pick a ready task, and dispatch **one sub-agent per task**, seeded only with that
   task file + its referenced spec/plan + the code it touches (in its own worktree). The
   sub-agent runs red→green→hooks→`in_review` (through the PR via `deliver.md`) and returns a
   **short summary** (task id, final status, PR link, notes). The controller records that one
   line and moves on — it never absorbs the sub-agent's working context. Cleanest isolation,
   and it pairs naturally with per-task worktrees.
2. **Fallback — compact between tasks.** If sub-agents aren't available, at each task's
   terminal state (PR opened / merged) **compact**: drop that task's working details (diffs,
   test output, exploration) and keep only a durable **ledger** — the map/overview plus each
   task's `id → status → PR`. Start the next task by re-reading *its* files fresh.

Either way: the **loop controller keeps only a minimal ledger**, never accumulated
transcripts; and when you start a task, read **its** task file as the single source of truth
— do not carry another task's decisions forward from memory.

Two more habits:

- **Flag a runaway task.** If a single task burns more than ~200k tokens (or the session
  starts feeling sluggish/repetitive), stop — that usually means the task was too vague or
  too big. Sharpen or split it rather than pushing a bloated context further.
- **Context-reset pattern.** When a task runs long, commit work-in-progress to its branch,
  start a **fresh session**, and let it re-read the spec map + task file + branch state and
  continue. A clean context outperforms a stale one near capacity — prefer many short
  sessions over one marathon.

---

## Step 0 — Sync from GitHub first (GitHub mode only)

GitHub is the source of truth for merge state, and a human may have merged a PR while **no
agent was running**. So before selecting work, reconcile the local task files with GitHub —
otherwise a task drifts (`in_review` forever) and you may re-implement something merged.

**Only check tasks with `status: in_review`.** Those are the only ones with an open PR a
human could have merged. `todo` / `blocked` / `in_progress` have no PR yet, and `done` is
already settled — checking them is wasted `gh` calls. So if nothing is `in_review`, skip
this step entirely.

For each `in_review` task, using its `issue:` number (or the `T-<id>` label):

- PR **merged** → set `status: done` in `docs/tasks/<id>.md`; clean up its worktree/branch.
- PR **closed without merging** (rejected) → move it back to `todo`/`in_progress` and note why.
- PR still **open** → leave as `in_review`.

```sh
# reconcile the in_review tasks only, e.g. T-002 (mirrored issue #2):
gh pr list --search "T-002 in:title" --state merged --json number --jq '.[0].number'  # merged?
# if merged ⇒ set status: done in docs/tasks/T-002.md
```

Only after this sync do you select ready tasks. (**Local mode:** skip — the files are
already the source of truth.)

---

## Step 1 — Select ready tasks

A task is **ready** when its `status` is `todo` **and** every id in its `depends_on` is
`done`. Everything else is `blocked` (leave it). If nothing is ready and nothing is in
progress, the loop is complete — hand back to `deliver.md` for final loop control.

---

## Step 2 — Decide serial vs parallel (guardrailed)

Default to **serial**. Parallelism usually moves the bottleneck onto the human (three PRs
land at once on one reviewer), so only fan out when *all* of these hold:

- every candidate task has `parallel_safe: true`, **and**
- their `touches` globs do **not** overlap (no shared files → no merge conflicts), **and**
- you stay under a fixed **concurrency cap** (e.g. 2–3), **and**
- you warn about the review bottleneck: parallel work piles up at Inspection, so only do
  it once the single-agent line has been boringly reliable.

If any condition fails, run serially. Sequential is often faster because the bottleneck
is the human reviewer, not the agent.

> Documented tradeoff vs the source guide: "Running the Line" recommends sequential-first
> and parallel only once the line is reliable. This skill permits fan-out but gates it
> behind the four conditions above so it can't create a review pile-up by accident.

### Isolate each task on its own branch (worktree when parallel)

Every task is built on its own branch `task/<id>-<slug>` cut from the base branch — never
directly on `main` — and this branch is created **at the start of implementation**, not at
delivery.

- **Serial:** work on the task branch in the main working tree.
- **Parallel:** give each concurrent task its own **git worktree**, so there is zero shared
  git state (separate working tree + branch + index):

  ```sh
  git worktree add ../<repo>.wt/<id> -b task/<id>-<slug> <base-branch>
  ```

  Run the entire TDD + hooks loop *inside* that worktree. This is what makes real
  concurrency safe: the `touches` guardrail prevents file-*content* conflicts, and the
  worktree prevents git-*state* conflicts (HEAD / index / checkout). The `touches`
  non-overlap rule alone is **not** enough for genuinely concurrent agents — they would
  still share one HEAD/branch without a worktree.

  **Cold-start caveat:** a fresh worktree *shares* the repo's installed git hooks (they
  live in the common git dir) and pre-commit's global tool cache — so you do **not** re-run
  `pre-commit install`. But it does **not** share the project's dependency/build state
  (`node_modules`, `.venv`, `target/`, compiled artifacts). Run the project's install/build
  in the new worktree (e.g. `uv sync`, `npm install`) **before** tests and gates, or they
  fail spuriously. `deliver.md` pushes and opens the PR from that worktree, then removes it
  after the merge.

---

## Step 3 — Per task: TDD red → green

For each selected task, work from its mini Work Order body. **Before the first edit, put
the task on its own branch (and worktree, if parallel) per Step 2**, so every edit, test,
and commit happens on `task/<id>-<slug>`, never on `main`.

1. **Plan before editing.** Read the code the task touches. Restate what you'll change.
   No premature patching.
2. **Set `status: in_progress`** in the task frontmatter.
3. **Write the tests first**, derived from the body:
   - one test per **acceptance criterion**,
   - one test per **edge case** (the 2am breakers),
   - assertions that would fail if an **off-limits** constraint were crossed.
   Run them — they must be **red** for the right reason (feature missing, not typo).
4. **Implement the minimum** to turn the tests green. Nothing beyond what the task needs.
5. **Green.** All the task's tests pass.

### Off-limits enforcement
Treat the task's Off-limits list as hard constraints. If satisfying a test seems to
require crossing one (touching a forbidden file, adding a banned dependency, changing a
frozen API), **stop and surface it** — do not cross it silently.

---

## Step 4 — Tests ≠ correctness (guard the intent)

Agents can satisfy the letter of a visible suite while missing its intent, and the gap
grows on longer tasks. The always-available guard is cheap: **carry the explicit question
forward to Inspection — *"did this satisfy the harness, or the intent?"*** — it is not only
a Station 2 concern.

Two **optional, advanced** techniques for non-trivial or safety-relevant work — use them
only when the harness actually supports them, and skip them otherwise rather than pretending:

- **Held-out tests.** If a separate reviewer or sub-agent writes the implementation, keep a
  few tests it doesn't see or edit and run them at Inspection. Concrete convention: put them
  in `tests/held_out/` and have the implementing step ignore that path; reveal at Station 4.
  In a single-agent session this offers little — don't fake it; rely on the Inspection
  question above instead.
- **Mutation testing** on critical paths (a mutation tool flips small code changes and
  checks a test fails) — note *where* it's worth running; wire it in only if your stack has
  a mutation tool configured.

---

## Step 5 — Run the Andon Cord and self-correct

Run the pre-commit gates (format, lint net-new, type-check, secret scan) before committing.

- A hook failure prints a **precise** error. Read it, fix that exact thing, re-run, go
  green. This self-recovery loop needs **no human** and is the difference between agents
  that finish and agents that stall.
- **Never** `--no-verify`. **Never** edit hook/lint/type config or delete/skip a test to
  go green (`running-the-line` non-negotiables 3 & 4). If a gate seems genuinely wrong, stop and surface it.
- A fast, clear, automatic feedback loop is worth more than a smarter model — keep it fast.

---

## Step 6 — Hand this task to delivery NOW (per task, not batched)

When *this* task's tests are green and its hooks pass:

- Set the task `status: in_review`.
- In GitHub mode, push the task's branch and sync the status label.
- **Immediately hand off to `deliver.md` for THIS task** — it opens the PR (Station 4, the
  human review gate). One finished task ⇒ one PR, right now.

> **Do NOT implement every task first and deliver once at the end.** Each task goes to its
> own PR the moment it reaches `in_review`. With 15 tasks you get ~15 PRs over time, not one
> batch at the finish.

While that PR waits for the human to merge, you may return to Step 1 and start the next
*independent* ready task (its own worktree) — but this task is not `done`, and its
dependants stay blocked, until a human merges its PR.

---

## How the loop runs — one task at a time (implement ⇄ deliver)

The pipeline alternates between this skill and `deliver.md` **per task**. Read it as a cycle,
not a batch:

```text
   ┌─────────────────────────── next ready task ───────────────────────────┐
   │                                                                        │
pick ONE ready task ─► branch/worktree ─► tests red ─► implement ─► green   │
   ─► hooks green ─► status = in_review                                     │
        │                                                                   │
        └─► deliver.md: open THIS task's PR ─► human reviews & MERGES ──────┘
                                               │
                                               └─► status = done ─► dependants unblock

Exit only when no task is READY and none is in_review/awaiting-merge → line done.
```

Per task, in order: `implement → in_review → PR → (human merge) → done`, then the next
ready task. `deliver.md` runs once **per task**, not once at the end.

---

## Self-check (dry-run validation)

- [ ] GitHub mode: synced **only `in_review`** tasks from GitHub (merged PR → `done`) before
      selecting work — didn't waste calls on todo/blocked/in_progress/done.
- [ ] Correctly identifies ready vs blocked tasks from `depends_on` + `status`.
- [ ] Refuses to parallelize tasks with overlapping `touches` or any `parallel_safe: false`.
- [ ] Each task is on its own `task/<id>-<slug>` branch; parallel tasks each run in their
      own git worktree (no shared git state).
- [ ] Each task runs in a clean context (a sub-agent per task, or compact between tasks); the
      loop controller keeps only a minimal ledger, not prior tasks' transcripts.
- [ ] Walks one task red → green: tests written first and failing for the right reason.
- [ ] Enforces off-limits (stops rather than crossing a forbidden constraint).
- [ ] Runs hooks and self-corrects a mechanical error without weakening any gate.
- [ ] Notes hidden/held-out tests + mutation on critical paths.
- [ ] Flips `status` to `in_review` and unblocks dependants.

---

## Handoff

When a task reaches `in_review`, proceed to **`deliver`** (Station 4) to open a
PR / review + merge, then loop back here while ready tasks remain.
