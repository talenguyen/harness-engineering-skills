---
name: deliver
description: "Opens one PR per task for human review, runs a narrow inspection, and controls the implement-deliver loop. Use when a task reaches in_review with green tests and hooks."
---

# Deliver

Runs **once per task**. One finished task → one PR → human merges → next.

## What to do

1. Push the task branch. Open a PR with `Closes #<issue>` in the body (from the task's
   `issue:` field) so the issue auto-closes on merge. See [reference/pr-body.md](reference/pr-body.md) for the body template.
2. Confirm CI is green.
3. **Stop.** Do not self-merge. Wait for the human to review and merge.
4. After the human merges:
   - Set `status: done`.
   - If all tasks for the same `section:` are now done, set that section's `Status: implemented` in the spec map.
   - Update any docs/conventions the task changed.
   - Clean up worktree/branch if applicable.
5. Recompute readiness → if ready tasks remain, return to `implement-task-loop`; otherwise report done.

If not present at merge (human merged async), `implement-task-loop` Step 0 reconciles on next run.

## Constraints

- Never push to `main`. Task already has its own branch.
- Stage specific files. Flag anything that looks like a secret.
- No `--no-verify`, no force-push, no self-merge (unless auto-merge is explicitly enabled).
- Local mode: produce a review summary, wait for human approval, then merge.

## Inspection checklist (what the reviewer checks)

- [ ] Acceptance criteria from the task — ticked against actual behavior.
- [ ] Judgment calls (security, data handling, choices the spec left open).
- [ ] Intent vs harness — did it satisfy intent, or just the visible tests?
- [ ] Diff shape (many files + trivial tests = drift; large diff + no tests = skipped TDD).
- [ ] AI smells (unexplained deps; fix touches callers not root; multiple approaches in one file).
- [ ] Codebase health (duplication, complexity, fit).
