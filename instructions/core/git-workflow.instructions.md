---
description: "Git workflow including commit message format, PR process, and feature implementation workflow"
applyTo: "**"
---

# Git Workflow

## Commit Message Format

```
<type>: <description>

<optional body>

Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

## Pull Request Workflow

When creating PRs:

1. Analyse the full commit history (not just the latest commit)
2. Use `git diff <base-branch>...HEAD` to review all changes
3. Write a comprehensive PR summary covering motivation, changes, and impact
4. Include a test plan with specific scenarios to verify
5. Push with `-u` flag if the branch is new

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

### 3. Code Review

- Request review after writing code
- Address CRITICAL and HIGH issues before merging
- Fix MEDIUM issues when possible within scope

### 4. Commit and Push

- Write detailed commit messages following the format above
- Use conventional commits format consistently
- Atomic commits — one logical change per commit
- Commit immediately upon task completion; do not batch across features
