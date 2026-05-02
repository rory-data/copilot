---
description: "Git workflow including commit message format, PR process, and feature implementation workflow"
applyTo: "**"
---

# Git Workflow

## Commit Message Format

```
<type>: <description>

<optional body>
```

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

- Write detailed commit messages following the format above
- Use conventional commits format consistently
- Atomic commits — one logical change per commit
- Commit immediately upon task completion; do not batch across features
