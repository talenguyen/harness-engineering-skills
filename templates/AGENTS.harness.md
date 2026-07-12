<!-- harness:start -->
# Harness Engineering Line

This repo runs the **Harness Engineering line**. Before any task, read **`harness.md`**
and follow it as a hard contract — it holds the full operating manual (the five stations,
the skills index, and the pipeline). Load the skill module `harness.md` points you to for
your current stage.

**Non-negotiables (authoritative full text in `harness.md`):**
1. Plan before you edit — read code and plan first; no premature patching.
2. Tests first — a failing test states "done" before the implementation exists.
3. Never bypass hooks — no `--no-verify`, no skipping CI or enforced gates.
4. Never weaken a gate to pass it — don't loosen config or delete/skip tests; fix the code.
5. The Work Order is human-owned — scope, acceptance criteria, and trade-offs need human
   approval (Gate 1 after the spec, Gate 2 after the task plan).
6. Isolate before you automate — confirm the Station 0 checklist before any unattended run.

Verify the line at any time: `bash tools/check-line.sh`.
<!-- harness:end -->
