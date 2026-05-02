---
description: "Workflow guidance for task orchestration, parallel execution, and multi-perspective analysis"
applyTo: "**"
---

# Workflow and Task Orchestration

## Available Copilot Skills

Copilot skills provide specialised domain knowledge. Invoke the relevant skill before tackling
complex tasks in that domain:

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `engineering-workflow` | Structured planning | Complex features, architecture decisions |
| `python-conventions` | Python coding standards | Writing or reviewing Python code |
| `python-testing-patterns` | Test-driven development | New features, bug fixes, improving coverage |
| `golang-conventions` | Go coding standards | Writing or reviewing Go code |
| `docker-best-practices` | Container image quality | Writing or reviewing Dockerfiles |
| `markdown-conventions` | Documentation standards | Writing or reviewing Markdown files |
| `principal-data-engineer` | Data platform architecture | Pipeline design, data quality |
| `gang-of-four` | Design patterns | Factory, Observer, Strategy, Decorator and other GoF patterns |
| `code-smell-detector` | Code quality review | Reviewing PRs, refactoring, identifying technical debt |
| `comment-reviewer` | Comment quality | Reviewing or auditing code comments and docstrings |
| `crusty-old-engineer` | Architectural reality check | Evaluating architecture, tooling choices, "is this a good idea?" |
| `instructions-builder` | Copilot instruction authoring | Creating or refining instruction files and rules |
| `skill-creator` | Skill creation | Creating, improving, or evaluating Copilot skills |

## Immediate Skill Usage

No user prompt needed — proactively use skills when:

1. Complex feature request → use `engineering-workflow` skill
2. Writing or reviewing Python → use `python-conventions` skill
3. New feature with tests → use `python-testing-patterns` skill
4. Architectural decision → use `engineering-workflow` skill

## Parallel Task Execution

ALWAYS use parallel execution for independent operations:

```
# GOOD: Parallel execution
Launch 3 sub-agents simultaneously:
1. Security analysis of auth module
2. Performance review of cache layer
3. Type checking of utility functions

# BAD: Sequential when unnecessary
Run security analysis, THEN performance review, THEN type checking
```

## Multi-Perspective Analysis

For complex problems, run split-role analysis in parallel:

- Correctness reviewer
- Security expert
- Performance analyst
- Consistency checker

Synthesise findings before acting on them.

## Task Scoping

- Keep each sub-task completable within a focused session
- One task per session — if scope creep occurs, split into separate tasks
- Subagents cannot invoke other subagents via shell; use the task tool with explicit parameters
