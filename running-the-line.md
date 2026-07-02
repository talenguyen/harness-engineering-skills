# Running the Line
## A shipping engineer's guide to agentic coding

> **The whole idea in one line:** Your output is no longer the code. It's the line that produces the code — and a good line kills every defect at the station that made it, before it can travel downstream and get expensive.

---

## Read this first

If you only take three things from this guide, take these:

1. The bottleneck is moving from writing code to defining correctness and verifying output.
2. The highest-leverage change is not "better prompting." It's adding quality gates before code reaches review or production.
3. Start with one real repo and one small task. You do not need a grand rollout to get value.

This guide is organized in four moves: the shift, the five stations, one concrete walkthrough, and a first-week rollout.

---

## 1. The jump already happened

For thirty years the hard part of software was *production*: turning a clear idea into correct syntax. That part is much cheaper now. In many everyday tasks, an agent writes the function faster than you can describe it cleanly.

So the bottleneck didn't slowly drift. It jumped — once, hard — from *making* the code to *being sure the code is right and was the right thing to make at all.* Generation got cheaper. Verification, judgment, and direction matter more.

This isn't just a forecast. Recent research and industry reporting point in the same direction: humans still provide the goals, constraints, and judgment, while agents increasingly handle the mechanical execution. The split is already becoming *you decide, it builds.*

This breaks the two reflexes most engineers still run on:

- **Babysitting** — prompt, watch, correct, prompt again. You've just made yourself a slower autocomplete. The agent is fast; you became the bottleneck by refusing to leave the room.
- **Vibe merging** — prompt, glance, ship. Fast until a defect you never inspected surfaces in production three weeks later wearing a customer's name.

Both fail for the same reason: they treat the agent's output as the thing to manage. The thing to manage is the *line the output travels down.*

---

## 2. You run the line, you don't make the part

This guide uses production-line names because they carry the logic well:

| Term | Meaning here |
|------|--------------|
| **Work Order** | The written spec that defines "correct" before anything is built |
| **The Part** | The implementation, tested against the work order |
| **Andon Cord** | A stop-the-line quality gate; in software, this is usually hooks and automated checks |
| **Inspection** | Final human review before shipping |
| **Field Returns** | Production failures that should feed back into upstream checks |

A factory manager does not hand-weld each chassis. They design the line, put a quality check at every station, and then watch the gauges. If welds start failing, they don't grab a torch — they fix the welding station so the next thousand welds come out right.

That is your job now. Not "write the auth code." Build the line that turns a spec into shipped auth code, with a check at every station, so you can hand off the next ten features without hovering.

There is exactly one law that makes a line good:

> **A defect gets more expensive at every station it survives. So each station must catch its own defects — and never pass one downstream.**

This isn't a metaphor for inspiration. It's a cost function you can feel:

| Where the same class of bug gets caught | What it costs you |
|------------------------------------------|-------------------|
| In the spec, before any code | a sentence rewritten |
| By a test, as the agent writes it | the agent fixes itself, 30 seconds, no human |
| By a pre-commit hook | 8 seconds, agent reads the error and retries |
| In PR review | your afternoon, a round-trip, context reload |
| In production | three weeks later, an incident, a customer, your weekend |

Every technique in this guide exists for one reason only: **move the catch as far left on that table as possible.** When you understand that, you stop memorizing layers and start asking the only question that matters — *which station is letting defects through, and how do I make it catch them itself?*

The payoff is counterintuitive and it's the whole point: **the goal is not to remove human review. It's to shrink human review until it's small enough to stay rigorous.** Rubber-stamping happens because there are 400 lines to read and 380 of them are lint and boilerplate. When the line has already killed all 380, you're left reviewing the 20 lines of actual judgment — and you'll actually read them.

---

## 3. The five stations

The line has five stations, ordered by what a defect costs if it escapes. Build them left to right; the leftmost stations are cheap and catch the most expensive defects.

```text
  WORK ORDER  →   THE PART   →  ANDON CORD  →  INSPECTION  →  FIELD
   (spec)         (tests)        (hooks)        (PR review)    (prod)
   catches:       catches:       catches:       catches:       catches:
   "wrong         logic &        lint, types,   spec drift,    everything
    thing"        behavior       secrets,       judgment       you missed
                  defects        formatting     calls
   cost if        cost if        cost if        cost if        cost:
   escaped:       escaped:       escaped:       escaped:        max
   catastrophic   high           low            medium
```

### Station 1 — The Work Order (spec)

This is the most powerful station and the one everyone skips. It catches the single most expensive defect in software: **building the wrong thing correctly.** No test, no hook, no reviewer downstream can catch that — by the time they see it, the wrong thing is already built.

The mechanic is cheap: before any handoff, write business rules, acceptance criteria, edge cases, and what's off-limits. Not a paragraph of vibes — a checklist a stranger could verify.

```markdown
## Task: [what]
### Business rules
1. [testable rule]
### Acceptance criteria
- [ ] [observable outcome]
### Edge cases
- [the thing that breaks at 2am]
### Off-limits
- [the constraint the agent must not cross]
```

**Realistic adoption:** start by writing the acceptance criteria *only* — three checkboxes. That alone catches most "wrong thing" defects. Add edge cases and off-limits once you've been burned a couple of times (you will be, and the burn tells you exactly what to add).

**Failure mode:** the spec becomes a novel. If it's longer than the code would be, you're hand-welding with extra steps. The spec defines *correct*; it doesn't design the solution. Leave the *how* to the agent.

### Station 2 — The Part (TDD + the plan)

A test is "correct" made machine-checkable. That's the entire reason TDD fits agents: instead of "build the feature" (the agent guesses what done means), each test states exactly what done means, and the agent gets a green/red signal on every step.

The high-leverage move isn't "write tests." It's **make the agent produce a test plan from your business rules and stop — before it writes a line of implementation.** This is your cheapest, highest-value intervention in the entire flow, for a concrete reason: agents that read and plan before editing succeed far more than agents that patch first (the research puts this correlation at the top of the list). The plan-approval gate *forces* that behavior on every task.

```text
You:    here are the business rules. Produce a TDD plan as a checklist.
        Do NOT implement yet.
Agent:  [checklist of test → behavior pairs]
You:    [read it in 60 seconds — this is the leverage point]
        "missing the concurrent-refresh case. add it. then implement."
```

**Realistic adoption:** you do not need 90% coverage. You need a test for each business rule and each edge case in the work order. That's it. Coverage theater is a different failure mode.

**Failure mode:** rubber-stamping the plan. The plan is cheap to skim and feels like progress, so people wave it through. Counter it with one habit — **try to reject the first plan.** Find one missing case. If you genuinely can't, the spec was good. If you can, you just caught a high-cost defect for the price of reading a checklist.

### Station 3 — The Andon Cord (pre-commit hooks)

On a Toyota line any worker can pull a cord that stops everything when they see a defect. Your pre-commit hooks are that cord, except automatic: lint, format, type-check, secret-scan. They catch the cheap, mechanical defects in seconds so neither you nor the agent wastes a single thought on them.

Install a local gate runner that fits your stack. `pre-commit` is one common option because it makes it easy to bundle linting, formatting, and secret scanning behind one command.

```yaml
# Example only — pin real versions before using this in a repo
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/biomejs/pre-commit
    rev: v0.6.1
    hooks: [{id: biome-check, args: ['--write']}]
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.24.2
    hooks: [{id: gitleaks}]
```

Secret scanning is one example; the same station holds any mechanical, automatable gate — dependency vulnerability checks, static analysis for common security anti-patterns, format enforcement. If a tool can say "fail" with a message precise enough for the agent to self-fix, it belongs here.

**The one rule that makes hooks work:** any check you expect the agent to self-correct must fail hard, not warn softly. Agents walk past warnings. A warning is a defect with a permission slip.

**Why this station is what makes hands-off possible:** a hook failure prints a precise error. The agent reads it, fixes that exact thing, re-runs, goes green — with no human in the loop. This self-recovery is the difference between agents that finish and agents that stall. A fast, clear, automatic loop is worth more than a smarter model.

**Realistic adoption — the part every other guide lies about:** you *cannot* flip every rule to "error" on day one in a real repo. You'll block every commit on 2,000 pre-existing violations and the team will rip the hooks out by lunch. Do the **ratchet** instead:
1. Run the linter, baseline the existing violations, ignore them.
2. Gate only *net-new and modified* code (most tools support exactly this).
3. Tighten one rule category per week as you clean up.

The line tightens over a month, not in an afternoon. Anyone who tells you to start with all-errors has never done it on a codebase with history.

**Failure mode:** the agent, stuck on a stubborn check, "fixes" it by editing `biome.json`, loosening `tsconfig`, or deleting the failing test. It optimized for green by dismantling the station. One line in your agent rules closes this: *never modify the linter/type/hook config or delete a test to make a check pass.*

### Station 4 — Inspection (PR review)

Final inspection. But here's the shift: because the three upstream stations already killed the wrong-thing, the logic, and the mechanical defects, **inspection is no longer "read everything."** It's narrow and high-value: *does this match the work order, and are the judgment calls sound?*

What you actually review now:
- Does it satisfy each acceptance-criteria checkbox? (You have the list. Check it off.)
- The handful of decisions only a human can judge — security boundaries, data handling, the choices the spec couldn't fully pin down.
- The diff *shape*: many files touched but trivial tests = spec drift; large diff with no tests = skipped Station 2. The shape tells you which upstream station leaked.

You are not re-reading lint. The line caught lint. That's the deal.

**Failure mode:** if you find yourself reading 400 lines line-by-line, an upstream station is leaking — push the catch back left instead of compensating with heroic review.

### Station 5 — Field Returns (production)

The most expensive station, and the one you want to catch *nothing*, because anything caught here already cost the maximum. Monitoring, error tracking, the ability to roll back. When a defect does reach here, the response isn't just "fix the bug" — it's **"which upstream station should have caught this, and why didn't it?"** Then you add the test or the rule so that class of defect dies at its station forever after. That feedback arrow — field failure becomes an upstream check — is what makes the line get smarter over time instead of just busier.

---

## 4. Find your leak

Don't build all five stations at once. Find the one leaking the most expensive defects *this week* and fix that station first.

| If your agent's PRs keep... | The leak is at... | Build this first |
|------------------------------|-------------------|------------------|
| solving a different problem than you meant | Work Order | acceptance-criteria checklists |
| being logically wrong but plausible-looking | The Part | TDD plan-approval gate |
| breaking lint/types/style, leaking secrets | Andon Cord | pre-commit hooks (ratcheted) |
| forcing you into hours of line-by-line review | Inspection (upstream leak) | tighten Stations 1–3 |
| surprising you in production | Field → upstream | add the missing test/rule, then ask why it escaped |

Most teams shipping with agents today leak hardest at **Station 1**. They have hooks and tests but hand off vague intent, so the agent builds the wrong thing correctly — the most expensive defect, caught last. Start there.

---

## 5. One part down the line

Watch a real task — JWT auth — travel the line, and notice what each station catches and what it would have cost downstream.

**Station 1 — Work Order.** You write: access token 15 min, refresh token single-use and rotates, protected routes return 401 never 500, reused refresh token revokes everything. *Catch: you almost wrote "refresh token valid 7 days" with no rotation — a session-hijack hole. Caught it while typing a sentence. Cost downstream: a security incident.*

**Station 2 — The Part.** Agent produces the plan: ten test→behavior pairs. You skim, reject once: "add concurrent-refresh." Agent adds it, then implements test-first. *Catch: the concurrency race — caught by reading a checklist for 60 seconds. Cost downstream: an intermittent prod bug nobody can reproduce.*

**Station 3 — Andon Cord.** On commit, the type-checker fails: a token compared as the wrong type. Agent reads the error, fixes it, re-runs green. *No human involved. Cost if it had shipped: a 401 storm.*

**Station 4 — Inspection.** You don't re-read every JWT library call. You check the four acceptance boxes and verify one real judgment call: does the token design match your threat model, storage model, and key-management setup? Five minutes. Approve.

**Station 5 — Field.** Nothing. Which is the point.

In a healthy version of this flow, human input can get this small: one spec, one plan-rejection, then review against the work order. The line does the rest.

---

## 6. The honest section

Things most guides won't tell you, because they're selling the dream.

**The line is for parts, not for R&D.** Assembly lines make well-defined things. Greenfield architecture, "I don't know what I want yet," deep novel domain logic — these do not go on the line. Hand-weld those in real-time with the agent (conductor mode), *then* put the well-defined implementation on the line. Forcing exploratory work through spec→plan→gate just adds ceremony to confusion.

**Parallel agents usually move the bottleneck onto you, not off you.** Three agents on three branches sounds like 3x throughput. But review is your bottleneck, and now three PRs land at once on one human. You didn't scale output; you built a queue at the most expensive station. Run parallel agents only on genuinely independent work (no shared files, no merge conflicts), and only once a single-agent line is boringly reliable. Otherwise sequential is often faster, because the bottleneck is you.

**The work order is always yours.** This is the permanent floor. You can automate production, testing, gating, even review-of-mechanics. You cannot automate *deciding what's worth building and what "correct" means.* That's not a current-model limitation that goes away next release — it's the residue of the bottleneck's jump. As the line gets better, more of your time concentrates here, not less. If that sounds like less engineering, it's the opposite: it's the part that was always the actual engineering.

**A line you don't understand is a liability.** If you can't read the code the agent wrote well enough to judge it, you don't have a line — you have a faster way to ship things you can't defend. The line amplifies a competent engineer. It does not replace the competence.

**Where this goes next.** This manual describes the line where human judgment still gates the important stations. That's the honest state of mid-2026: evals are improving, but for non-trivial work they're not yet a clean replacement for review. The line is *designed* to evolve. As eval suites mature, the stations narrow:

- Station 3 is already fully automated (hooks, no human).
- Station 2 next: eval-driven plan approval replaces human skim-and-reject for well-understood task types.
- Station 4 after that: eval-gated review where the human spot-checks only what the eval flags, not every PR.
- Station 1 — last, and likely always partly human: deciding what to build and what "correct" means.

The stations don't disappear; they automate inward. If you're reading this after the tools can reliably do one of those steps: tighten accordingly. If not, the manual above is how to ship with confidence today.

---

## 7. Your real first week

Not a four-week theory plan. One task, one real repo, this week.

```text
Day 1 — Pick a live repo with working tests (not greenfield).
        Write a 5-line agent rules file: stack, "tests first",
        "never bypass hooks", "never edit hook/lint config to pass".
Day 2 — Install a gate runner such as pre-commit. Hooks: linter
        (net-new only — ratchet),
        gitleaks. Any enforced gate should fail hard. Commit something
        trivial, watch it gate.
Day 3 — Take one small real task. Write acceptance criteria only (3 boxes).
        Hand off: "produce a TDD plan, don't implement yet."
Day 4 — Reject the first plan once (find one missing case). Approve.
        Let the agent implement + gate + open a PR. If the line is healthy,
        you should be able to step away for a while.
Day 5 — Review against your 3 boxes, nothing else. Merge.
        Write down the one thing that leaked, and which station should
        have caught it. That note is your week-2 backlog.
```

Done when: you handed off a task, stepped away, and came back to a PR that passed its own gates and matched its own spec. That's the line running. Everything after is widening it.

---

## Reference card

```text
THE LAW:  a defect costs more at every station it survives.
          build each station to kill its own defects.

STATIONS (left = cheap to catch, expensive if missed):
  1 Work Order    spec + acceptance criteria   catches "wrong thing"
  2 The Part      TDD plan → approve → tests    catches bad logic
  3 Andon Cord    pre-commit hooks, fail hard   catches mechanics
  4 Inspection    review spec-fit + judgment    catches drift
  5 Field         monitoring → feed back upstream

HANDOFF:  "Read the rules. Here's the spec. Produce a TDD plan.
           Do NOT implement yet."
          → reject once → "approved, implement test-first,
            run hooks, open a PR."

WHEN IT LEAKS:  push the catch left. don't compensate with heroics.
WHEN NOT TO:    R&D, unknown design, work that shares files across agents.
ALWAYS YOURS:   the work order. what to build and what 'correct' means.
```

---

## Notes on evidence

This guide makes a strategic argument, not a claim that one tool or one workflow universally wins. The main empirical points behind it are narrower:

- Adoption of AI coding tools is real, but productivity gains are uneven and context-dependent.
- Security failures in AI-built or AI-assisted apps are common enough that quality gates matter.
- Human judgment still matters most where the work is ambiguous: defining the task, checking boundaries, and deciding whether the output should ship.

If you publish this externally, add citations for any concrete numbers you keep.
