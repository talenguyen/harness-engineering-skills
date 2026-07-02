# Harness Engineering Agent Rules

## Line discipline

- Start coding tasks with a work order unless one already exists.
- Produce a TDD plan and stop for approval before implementation.
- Implement one vertical slice at a time: failing test, minimal code, gate.
- Run the relevant gates before review.
- Review against the work order, not against vague intent.
- Promote escaped defects upstream as tests, rules, gates, or review checks.

## Always

- Respect the repo's package manager and existing commands.
- Show evidence for tests, type checks, lint, secret scan, and other gates.
- Keep changes scoped to the work order.
- Record assumptions when correctness depends on them.

## Never

- Do not bypass hooks or use no-verify flags.
- Do not edit lint, type, test, or hook configuration just to make a check pass.
- Do not delete, weaken, or skip tests to get green.
- Do not print, edit, rename, delete, or commit secret files.
- Do not implement before the plan gate unless the user explicitly overrides it.

## Project stack

- Language/runtime:
- Package manager:
- Test command:
- Type-check command:
- Lint/format command:
- Secret scan command:
- CI command:

## Gate failure loop

Read the error, fix the exact issue, re-run the same gate, and proceed only when
green. If the gate itself is wrong, stop and ask for a separate config change.

