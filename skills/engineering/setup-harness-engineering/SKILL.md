---
name: setup-harness-engineering
description: Set up a repository to use the Harness Engineering line. Use when the user wants to install or configure harness engineering skills, create agent rules, add station docs, map quality gates, or prepare a repo for AI coding agents.
---

# Setup Harness Engineering

Use this skill to prepare an existing repository for agentic coding without
turning the workflow into ceremony. The goal is a small line the user can run on
one real task this week.

## Setup sequence

1. Inspect the repo before proposing changes.
   - Find the stack, package manager, test command, lint command, type-check
     command, existing CI, and existing agent rules.
   - Prefer existing project commands over inventing new ones.
   - If commands are missing, record the gap instead of pretending a gate exists.

2. Add or update agent rules.
   - If `AGENTS.md`, `CLAUDE.md`, or another agent rules file exists, preserve it
     and add a scoped "Harness Engineering" section.
   - If no rules file exists, create one from `templates/AGENTS.harness.md` if
     that template is available.
   - Include the core safety rules: tests first, never bypass hooks, never edit
     lint/type/hook config to make a check pass, never delete or weaken tests to
     get green.

3. Create a docs location for station artifacts.
   - Default to `docs/harness/` unless the repo already has a better docs
     convention.
   - Suggested files: `work-order.md`, `tdd-plan.md`, `inspection-notes.md`,
     `field-return.md`.

4. Define the first gate map.
   - For each gate, record: command, defect class caught, when it runs, and what
     the agent should do on failure.
   - Start with the gates the repo can already run.
   - For noisy legacy repos, recommend a ratchet: gate only new or modified code
     first, then tighten categories over time.

5. Run a harmless verification if possible.
   - Prefer commands that already exist and do not mutate unrelated files.
   - If a command fails, report whether it is a pre-existing repo issue or a
     setup issue.

## Output

End with:

- Files changed
- Gate map
- Known gaps
- The first task the user should run through the line

Keep setup small. A working two-gate line beats a grand plan nobody uses.
