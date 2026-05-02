---
description: "Git workflow including commit message format, PR process, and feature implementation workflow"
applyTo: "**"
---

# Git Workflow

## Commit Message Format

Use [Conventional Commits](https://www.conventionalcommits.org/) format strictly:

```
<type>(<scope>): <subject>

<optional body>
```

Common types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`

```
feat(ingestion): add Iceberg write path for bronze layer
fix(dag_release_audit): handle missing feature flag key gracefully
refactor(cli): extract argument parsing into dedicated module
chore: bump polars to 1.x
```

Subject line rules:
- Imperative mood — "add" not "added"
- 72 characters maximum
- No trailing period
- The body (separated by a blank line) explains _why_, not _what_ — the diff shows what

## Commit Atomicity

A commit should represent one logical change — independently reviewable, independently revertable,
and independently deployable.

The practical test: if the commit message requires the word "and" to be accurate, the commit likely
bundles two changes. Examples:

- Refactoring a function and fixing a bug in it → two commits
- Adding a new pipeline stage with its tests → one commit (logically coupled)
- Updating a dependency and fixing the downstream breakage → one commit (change and required consequence)

Tests belong in the same commit as the behaviour they cover. A commit adding behaviour followed by
a commit adding tests is almost always the wrong split.

## Commit Granularity

The right unit is a **coherent behaviour**, not a code structure boundary. Method-per-commit is
too granular — intermediate commits represent incomplete, broken code with no independent value.

Natural boundaries when building a module:

1. Scaffold — module file, class definition, `__init__` imports, nothing implemented
2. Each meaningful capability with its tests
3. Substantial error handling or edge cases, if worth isolating
4. Integration wiring — connecting the module into the broader system

## Staging Discipline

Use `git add -p` (patch mode) rather than `git add .`. Patch mode forces a review of exactly what
is being committed and makes atomic commits achievable even when multiple changes were made in one
editing session.

## Commit Size

No hard line on diff size, but a useful heuristic: if a reviewer cannot meaningfully review it in
a single sitting, it is too large. For broad refactors, prefer a series of preparatory commits
(rename, extract, move) followed by the substantive change. This keeps individual commits
reviewable and makes `git bisect` tractable when tracking down regressions.

## Pull Request Workflow

When creating PRs:

1. Analyse the full commit history (not just the latest commit)
2. Use `git diff <base-branch>...HEAD` to review all changes
3. Write a comprehensive PR summary covering motivation, changes, and impact
4. Include a test plan with specific scenarios to verify
5. Push with `-u` flag if the branch is new

## Pre-PR Quality Checks

Run these against the full branch diff (`git diff <base-branch>...HEAD`) before opening a PR.
`roborev` handles general code review automatically post-commit; these checks cover specialised
dimensions that complement it.

- **Code smells**: invoke `code-smell-detector` on changed source files. Fix Critical and Major
  findings before opening the PR.
- **Comment quality**: invoke `comment-reviewer` on any files where comments or docstrings were
  added or modified. Fix Critical issues; address Rot Risk items where practical.
- **Silent failures**: if catch blocks or fallback logic were added, apply the 5-question cascade
  from `code-smell-detector` Smell #8.

## Feature Implementation Workflow

### 1. Plan First

- Identify dependencies and risks
- Break the feature into phases
- Confirm scope before writing code

### 2. TDD Approach

- Write tests first (RED)
- Implement to pass tests (GREEN)
- Refactor (IMPROVE)
- Verify 80%+ coverage

### 3. Commit and Push

- Follow the commit message format and atomicity rules above
- Commit immediately upon task completion; do not batch across features
