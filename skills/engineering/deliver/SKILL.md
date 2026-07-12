---
name: deliver
station: "4"
description: "Runs the narrow Station 4 inspection against acceptance criteria, opens a PR (GitHub) or does a local review + merge, and controls the pipeline loop. Use when a task reaches in_review with green tests and hooks."
requires:
  - "a task with status: in_review, green tests and green hooks"
produces:
  - "(GitHub) a PR on the task branch, awaiting human review + merge; (local) a review summary for the human"
  - "on human merge/approval: task status advanced to done; loop continues or exits"
enforces:
  - "git-safety: never push to main, specific staging, flag secret files, no force-push, no self-merge"
---

# Deliver — Station 4 (Inspection) + Loop Control

> **Runs once per task.** The moment a single task reaches `in_review`, this skill opens
> that task's PR — it does **not** wait for all tasks to be implemented. One task ⇒ one PR.
> After the human merges, control returns to `implement-task-loop.md` for the next ready task.

Final inspection, then merge, then loop. The shift at this station: because Stations 1–3
already killed the wrong-thing, the logic, and the mechanical defects, **inspection is no
longer "read everything."** It's narrow and high-value — *does this match the Work Order,
and are the judgment calls sound?* If you're reading 400 lines line-by-line, an upstream
station is leaking; push the catch left instead of compensating with heroic review.

> Don't assume green-and-small means safe: published studies show agent PRs are rejected /
> fail meaningfully more often than human PRs. Budget review time accordingly.

---

## Narrow inspection checklist (what you actually review now)

- [ ] **Acceptance criteria.** Tick every box in the task's Work Order body against the
      actual behavior. You have the list — check it off.
- [ ] **Judgment calls.** The handful of decisions only a human can judge: security
      boundaries, data handling, and the choices the spec couldn't fully pin down.
- [ ] **Intent vs harness.** Did it satisfy the *intent*, or just the visible tests?
      Check the held-out tests / mutation notes from Station 2.
- [ ] **Diff shape.** Many files touched but trivial tests → spec drift. Large diff with
      no tests → Station 2 was skipped. The shape tells you which upstream station leaked.
- [ ] **AI-specific smells.** Unexplained new dependencies, diff size vs test coverage,
      whether CI actually passed (not just local hooks), a fix that changes **callers**
      instead of the root cause (architectural reasoning gap), and **multiple competing
      approaches in one file** (the agent was guessing / looping).
- [ ] **Codebase health.** Duplication, complexity, architectural fit — these erode even
      when every step's tests passed, and no earlier station catches them. This is the
      Inspection-only question.

You are **not** re-reading lint. The line already caught lint. That's the deal.

---

## GitHub mode

Follow git-safety throughout: stage specific files (not `git add -A`), flag any file that
looks like it holds secrets before staging, never push to `main`/`master`, never
force-push unless explicitly asked, keep hooks (no `--no-verify`).

1. The task is already on its own branch `task/<id>-<slug>` (created at the start of
   implementation — in its own worktree if it ran in parallel). Never commit onto `main`.
2. Stage the specific files for this task and commit on that branch. Let the hooks run.
3. Push with upstream tracking: `git push -u origin task/<id>-<slug>`.
4. Open a PR with the CLI (`gh pr create`), concise title (< 70 chars), structured body.
   **The body MUST include a `Closes #<issue>` line** for the task's mirrored issue (from
   the task's `issue:` frontmatter, or found via the `T-<id>` label). This is what makes
   GitHub **auto-close the issue when the PR merges** — including when a human merges later
   in the web UI with no agent present. Without it, issues stay open and drift.

```markdown
Closes #<issue>

## What & why
<what changed, and why it was needed — 1-3 sentences>

## How to test
<manual steps a reviewer can run to verify: commands / requests / expected results>

## Tested
<tests added/run; note any held-out tests / mutation on critical paths>

## Acceptance criteria (from docs/tasks/<id>.md)
- [x] <criterion 1>
- [x] <criterion 2>

## Checklist
- [x] No secrets exposed
- [ ] Docs/spec updated if behavior or conventions changed
- [ ] Breaking changes documented (or "none")

## Blockers / follow-ups
<anything deferred, or "none">
```

5. Confirm CI (the server-side gates) is green — that's the real enforcement boundary,
   not the local hooks.
6. **STOP — the PR is the human Inspection gate.** Do **not** merge your own PR. Wait for
   the human to review and merge it. **Default policy: always wait for a human merge.** Only
   merge without waiting if the human has explicitly enabled auto-merge for this project.
   (This is rule 5 — the Work Order, and the decision to ship, stay human-owned.)
7. **After the human merges:** set the task `status: done` (the `Closes #<issue>` line
   already closed the mirrored issue on merge), and if the task ran in a worktree, clean up —
   `git worktree remove <path>` and delete the merged local branch. Then go to loop control.
   **If you were not present at the merge** (human merged async in the web UI), you can't run
   this live — the **Sync step** at the top of `implement-task-loop.md` reconciles it on the
   next run (marks the task `done` from GitHub's merge state, cleans up the worktree/branch).

While a PR is awaiting review you may start the next *independent* ready task (in its own
worktree), but a task is **never** `done` until its PR has been merged by a human.

---

## Local mode

1. Produce a **review summary**: the diff shape, the acceptance-criteria checklist ticked
   against behavior, and the judgment calls that need a human eye.
2. **Wait for the human** to review — this is the Inspection gate. Do not self-merge past
   unresolved judgment calls.
3. On approval, merge to the main line and set the task `status: done`.

---

## Loop control (this skill drives the pipeline's loop)

After the **human** merges a task's PR (GitHub mode) or approves the local review — never
before:

1. Set the task `status: done` (sync the GitHub issue/label in GitHub mode).
2. **Close the spec section if complete.** The task's `section:` names the spec section it
   implements. If **all** tasks with that `section:` are now `done`, set that section's
   `Status: implemented` in `docs/specs/<slug>/<section>.md` and update its row in the map's
   Sections table — so `overview.md` always reflects what's actually shipped.
3. **Doc sync (easy to skip).** If the task introduced a new convention, changed behavior,
   or broke something, update the project's agent rules (`AGENTS.md` / `harness.md`), the
   relevant `docs/*`, and `CHANGELOG` **now** — stale docs make the next task build against
   the wrong picture.
4. Recompute readiness: any task whose `depends_on` are now all `done` becomes ready.
5. **If ready tasks remain → return to `implement-task-loop`** for the next one.
6. **If no tasks are ready and none in progress → the line is done.** Report completion.

```text
in_review ─► inspect (narrow) ─► merge ─► status=done
                                            │
                    ready tasks remain? ────┼─ yes ─► implement-task-loop.md
                                            └─ no  ─► DONE (report)
```

---

## Self-check (dry-run validation)

- [ ] The PR / review-summary checklist maps 1:1 to the task's acceptance criteria.
- [ ] Inspection stayed narrow (criteria + judgment + shape + smells + health), not a
      line-by-line lint re-read.
- [ ] GitHub mode: task's own branch used (never main), `-u` push, structured PR body, CI confirmed.
- [ ] GitHub mode: did **not** self-merge — waited for the human to merge (unless auto-merge
      was explicitly enabled); worktree removed + branch cleaned up after merge.
- [ ] Local mode: waited for human approval before merge.
- [ ] git-safety honored (specific staging, secret-file flag, no force-push, hooks kept).
- [ ] When a section's last task merged, its spec section `Status: implemented` and the map's
      Sections row were updated (spec map matches what shipped).
- [ ] Loop returns to `implement-task-loop.md` while ready tasks remain; exits when all `done`.

---

## Handoff

- More ready tasks → **`implement-task-loop`**.
- All tasks `done` → the line has run. If a defect later escapes to production, load
  **`field-returns`** (Station 5).
