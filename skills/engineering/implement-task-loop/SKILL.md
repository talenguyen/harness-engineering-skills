---
name: implement-task-loop
description: "Implements ready tasks test-first (red→green), self-corrects against fail-hard hooks, and advances task status. Use after Gate 2 to build approved tasks one at a time."
---

# Implement Task Loop

Alternates with `deliver` per task: implement one → open its PR → human merges → next.

## Clean context per task

Each task is a clean room. Do not carry prior task transcripts forward.

- **Best:** dispatch one sub-agent per task, seeded with only that task file + its spec/plan
  + the code it touches. Controller keeps a one-line ledger (id → status → PR).
- **Fallback:** compact between tasks — drop working details, keep the ledger, re-read the
  next task's files fresh.

If a single task burns >~200k tokens or the session turns sluggish, stop — the task is too
vague or too big; split it.

## Step 0 — Sync (GitHub mode only)

For each task with `status: in_review`, check if its PR was merged or closed while no agent
was running. If merged → set `done`; if closed unmerged → set back to `todo` and note why.
Skip `todo`/`in_progress`/`done` tasks (no PR to check).

## Step 1 — Select ready tasks

Ready = `status: todo` AND every `depends_on` is `done`. If nothing is ready and nothing is
in progress → loop complete, hand to `deliver` for final exit.

## Step 2 — Branch + isolation

Create `task/<id>-<slug>` branch from the base branch **before the first edit**.
- Serial: work in the main tree.
- Parallel (only when all candidates are `parallel_safe`, `touches` don't overlap,
  concurrency ≤ 2-3): each task gets its own `git worktree`. Install project deps in the
  fresh worktree before running tests.

## Step 3 — Generate test plan (fresh read, before implementation)

Compact context (or use a sub-agent). Re-read the **spec section** + the task body from
scratch. Derive a test plan (test→behavior pairs) from the spec's acceptance criteria and
edge cases. Present the test plan for approval before writing any code.

## Step 4 — TDD red → green

1. Read the code the task touches. Plan what you'll change.
2. Set `status: in_progress`.
3. Write tests from the approved test plan. Run them; they must be **red** for the right reason.
4. Implement the minimum to turn them green.
5. If passing a test requires crossing an **off-limits** constraint → stop and surface it.

## Step 4 — Hooks (Andon Cord)

Run pre-commit gates. On failure: read the error, fix the exact issue, re-run. Never
`--no-verify`; never edit gate config to pass.

## Step 5 — Hand to deliver

Set `status: in_review`. Push the branch. Hand off to `deliver` for this task's PR.

While the PR awaits review, you may start the next independent ready task (own worktree) —
but this task isn't `done` until the human merges it.
