---
name: intake-requirement
station: "1"
description: "Understands what the user actually wants — or sharpens a vague idea — through one-question-at-a-time dialogue, then captures it as a resumable spec map (an overview plus per-section Work Orders). Use when a new requirement, idea, or bug needs to become a spec; stops at Gate 1 for human approval before any design or code."
produces:
  - "docs/specs/<slug>/overview.md — the map: intent, goals, section index + status"
  - "docs/specs/<slug>/<section>.md — one mini Work Order per section"
gate: "GATE 1 — confirm the map first, then approve each section spec before it flows downstream"
blocks_until: "the human confirms the map, and approves each section spec"
---

# Intake Requirement — Station 1 (The Work Order)

The single most expensive defect in software is **building the wrong thing correctly** —
no test, hook, or reviewer downstream can catch it. So the one job of this station is
narrow and it is everything:

> **Understand what the user actually wants — or help them sharpen a vague idea — and
> write it down as something a stranger could verify.**

The deliverable is *shared understanding, captured* — not a document for its own sake. If
the idea is vague, your job is to make it concrete *with* the user, not to guess.

---

## Elicit before you draft

Getting intent right comes from questioning, not from drafting fast and hoping. Before you
write a spec:

- **Look facts up; ask only decisions.** If something is knowable from the codebase
  (current API shape, existing patterns, what already exists), find it yourself — don't
  spend a question on it. *Decisions* — scope, priorities, trade-offs, what "correct"
  means — are the human's. Put each decision to them.
- **One question at a time.** Asking several at once is bewildering. Prefer multiple-choice;
  open-ended is fine when it has to be.
- **Propose your recommended answer with every question.** People correct a draft far
  faster than they compose one from scratch. Lead with your pick and why.
- **Stay on the *what* and *why*** — purpose, constraints, success criteria. Not the *how*
  (that's Station 2). If you're reaching for data structures or function names, stop.
- **Every request gets this, regardless of how simple it looks.** "Too simple to need a
  spec" is exactly where unexamined assumptions cost the most. Keep the spec *short* for
  simple asks — never skip it.

---

## The map + sections model (never lose context in a long session)

This is the mechanic that makes a long brainstorming session resumable and safe. **The
files are the memory — not the chat.**

```text
docs/specs/<slug>/
  overview.md        # the MAP: intent, goals/non-goals, section index + status,
                     #          cross-cutting constraints, open questions
  <section>.md       # one mini Work Order per section
```

How it works:

1. **As soon as intent is clear, draft the map** (`overview.md`) — before diving into any
   section. It states the intent in plain language and breaks the work into sections.
2. **Confirm the map with the user** (right intent? right breakdown?) before detailing.
3. **Work sections one at a time.** Each finished section is written to its own file and
   its status updated in the map.
4. **Upward feedback:** if detailing a section changes the big picture (a new section, a
   changed goal, a cross-cutting constraint), **update `overview.md` immediately.** The
   map must never drift from reality — it is always authoritative and always current.

Because the map records every section's status, an agent (even a fresh session) can always
re-read `overview.md`, see exactly where things stand, and **resume, redraft one section,
or pause to implement a finished section and come back** — without relying on chat memory.

Section status values: `todo | drafting | drafted | approved | implemented`.

---

## Procedure

1. **Understand the intent.** Run the elicit loop above until you can state, in one or two
   sentences, what the user actually wants and why. Reflect it back and get confirmation.
2. **Draft the map → `overview.md`** (template below). **Decompose:** if the request spans
   independent subsystems, list each as a section; a simple feature is a single section.
   Present the map and confirm intent + breakdown before drafting any detail.
3. **For each section, in priority order:**
   1. Elicit that section's details (one-at-a-time, recommend answers, facts vs decisions).
   2. Draft `<section>.md` as a mini Work Order: testable business rules; ≥3
      stranger-verifiable acceptance criteria; edge cases; off-limits; open questions.
   3. **Find the expensive omission** — re-read asking "what would make this
      correct-looking but wrong?" (e.g. a refresh token with no rotation is a
      session-hijack hole). Fix it while it's still a sentence.
   4. **Self-review with fresh eyes:** scan for placeholders/TBD, contradictions,
      ambiguity (any requirement readable two ways → pick one and make it explicit), and
      scope. Fix inline.
   5. Update the section's status in the map; if the big picture shifted, update
      `overview.md` too.
   6. **Gate 1 (section):** present the section spec and stop for approval.
4. **Define *correct*, not *how*** throughout. Leave the solution design to Station 2.

---

## Resuming, pausing, interleaving

- **Resume:** read `overview.md` → find the first non-`approved` section → open its file
  (if `drafting`) or start it. The map tells you where you are.
- **Redraft:** any section can be reopened — set its status back to `drafting`, and update
  `overview.md` if the change affects the whole.
- **Interleave with implementation (allowed, guarded):** once a section is `approved`, you
  may pause intake, send *that section* down the pipeline (`create-plan` → …), and return
  to intake the next section later. Keep `overview.md` `Status: sections-in-progress` so a
  partial map is never mistaken for a finished one.

---

## `overview.md` template (the map)

```markdown
# Spec map: <title>

- Slug: <slug>
- Status: drafting            # drafting | sections-in-progress | complete
- Author: <human>
- Date: <yyyy-mm-dd>

## Intent
What the user actually wants, and why — 1-2 sentences, plain language.

## Goals / Non-goals
- Goal: <...>
- Non-goal: <explicitly out of scope>

## Sections
| # | Section (slug) | Purpose (one line) | Status |
|---|----------------|--------------------|--------|
| 1 | <section-slug> | <...>              | todo   |
| 2 | <section-slug> | <...>              | todo   |

## Cross-cutting constraints / off-limits
- <constraint that applies across sections>

## Open questions (map-level)
- <decisions still needed before the whole is settled>
```

## Section Work Order template (`docs/specs/<slug>/<section>.md`)

```markdown
# Spec: <title> — <section>

- Parent: docs/specs/<slug>/overview.md
- Status: drafting            # drafting | drafted | approved | implemented

## Context
Why this section exists, which area of the codebase it touches, and what user/system
behavior changes. (What and why — not how.)

## Summary
One or two sentences.

## Business rules
1. <testable rule>

## Acceptance criteria
- [ ] <observable outcome a stranger could verify>
- [ ] <observable outcome>
- [ ] <observable outcome>

## Edge cases
- <the thing that breaks at 2am>

## Off-limits
- <the constraint the agent must not cross>

## Open questions
- <resolve before approval>
```

---

## Worked example (vague idea → map → sections)

Human: *"add auth to the API."* (vague — intent unclear)

**Elicit** (one at a time, each with a recommended answer): stateless tokens or server
sessions? *(rec: stateless JWT)* · access-token lifetime? *(rec: 15 min)* · refresh
strategy? *(rec: single-use, rotating)* · where do refresh tokens live? *(a decision —
ask)*. Reflect back: *"You want stateless API auth with safe, rotating refresh."* Confirm.

**Map** — `docs/specs/api-auth/overview.md`:

```markdown
## Intent
Stateless token auth for the API, with safe refresh-token rotation.

## Sections
| # | Section (slug)   | Purpose                                  | Status |
|---|------------------|------------------------------------------|--------|
| 1 | access-tokens    | issue/verify short-lived access tokens   | todo   |
| 2 | refresh-rotation | single-use rotating refresh + reuse guard| todo   |
| 3 | auth-middleware  | protect routes, 401 never 500            | todo   |
```

**Section** — `docs/specs/api-auth/refresh-rotation.md`: rules (single-use; rotate on every
refresh; reuse revokes the family); acceptance criteria (old refresh stops working after
use; reuse revokes all user tokens); edge cases (two concurrent refreshes; clock skew);
off-limits (never store refresh tokens in localStorage).

> The catch: the user almost said "refresh token valid 7 days, no rotation" — a
> session-hijack hole. Caught while typing a sentence. Cost downstream: a security incident.

---

## Gate 1 — the hard stop(s)

- **Map gate:** *"Here's the map for `<slug>`: intent + N sections. Right intent and
  breakdown? Approve the map, or tell me what's off."* Don't draft section details until
  the map is confirmed.
- **Section gate:** *"Section `<name>` is drafted (`<path>`). Acceptance criteria: N. Open
  questions: `<list, or none>`. Approve, or change?"* A section flows downstream only once
  `approved`.

Don't skip a gate under deadline pressure — if you must, log it as visible debt and say so.
On approval, set the relevant `Status:` fields.

---

## Self-check (dry-run validation)

- [ ] You can state the user's true intent in 1-2 sentences and the user confirmed it.
- [ ] Questions were asked one at a time, each with a recommended answer; facts were
      looked up, not asked.
- [ ] `overview.md` exists as the map with a section index + per-section statuses, and is current.
- [ ] Each drafted section has ≥3 stranger-verifiable acceptance criteria and testable rules.
- [ ] The map was confirmed before section detail; each section halted at Gate 1.
- [ ] A fresh session could resume from `overview.md` alone (no reliance on chat history).
- [ ] The spec defines *correct*, not *how*.

---

## Handoff

On section approval, proceed to **`create-plan`** (Station 2 — design) for that
section — or for the whole map once all sections are `approved`. `create-plan` reads
`overview.md` for context plus the approved section spec(s) it should design for.
