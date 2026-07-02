# [Project Name] Agent Guide

> Fill this file only with repo-specific facts. Delete any line you cannot
> verify from the repo or from the project owner.

## Before Starting Work

- Read [doc path] for build/test commands.
- Read [doc path] for project-specific conventions.

## Stack

- [runtime/language discovered from repo]
- Package manager: [exact package manager; do not guess]
- Test runner: [exact test runner if known]

## Always

- Write or update tests before implementation when behavior changes.
- Run [exact test command] before review.
- Run [exact lint/type/format command] before review.
- Show command output as evidence.
- Keep changes scoped to the work order.

## Never

- Do not bypass hooks or use no-verify flags.
- Do not edit lint, type, test, or hook configuration just to make a task pass.
- Do not delete, weaken, or skip tests to get green.
- Do not print, edit, rename, delete, or commit secret files.
- Do not touch [repo-specific off-limits paths or APIs].

## When A Gate Fails

Read the exact error, fix that defect, re-run the same gate, and proceed only
when green. If the gate itself is wrong, stop and propose a separate config
change.

