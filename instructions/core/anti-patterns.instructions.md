---
description: "Anti-pattern prevention rules to avoid common AI-assisted coding mistakes"
applyTo: "**"
---

# Anti-Pattern Prevention Rules

## Hallucination Guard

- Before calling any API or method, verify it exists in the project's dependencies or documentation.
- If unsure whether a module or function exists, check `requirements.txt`, `pyproject.toml`, or equivalent first.
- Never invent import paths. Always verify with `find` or `grep` before importing.

## Grounded Claims

When asserting risks, failure modes, or trade-offs:

- Anchor the claim in a specific, verifiable source (postmortem, canonical reference, widely cited
  incident) when one exists
- If no strong source exists, frame the claim explicitly as experiential: "in practice…" or "a
  common failure mode is…"
- Do not present vague warnings as established fact

## Scope Discipline

- Implement ONLY what was explicitly requested.
- Do not add "nice to have" features, extra endpoints, or bonus functionality.
- If something additional seems needed, ask first — do not implement it.
- When fixing a bug, fix ONLY that bug. Do not refactor surrounding code unless asked.

## Completion Integrity

- Never claim "done" or "complete" unless ALL tests pass.
- After implementation, always run the test suite and report actual results.
- If tests fail, report the failures honestly — do not mark the task as complete.
- Include the actual test output in your completion report.

## Loop Prevention

- If you've attempted the same approach 3 times without progress, STOP and report the blocker.
- Do not repeat the same command expecting different results.
- If a fix doesn't work, try a fundamentally different approach instead of iterating.

## Context Discipline

- Do not load files "just in case". Load only what is needed for the current task.
- When reading errors, extract the relevant information — do not keep full stack traces in context.
- One task per session. If scope creep occurs, suggest splitting into separate tasks.

## Drift Prevention

- Re-read the task requirements before claiming completion.
- If implementation has diverged from the plan, acknowledge and explain why.
- Linter and type checker results override your confidence — fix all reported issues.
