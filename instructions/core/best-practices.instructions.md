---
description: "Best practices for planning, context management, commits, and avoiding common pitfalls"
applyTo: "**"
---

# Best Practices

## 1. Plan First, Always

- Use plan mode for any non-trivial task
- Break complex features into phases
- Validate feasibility before beginning implementation

## 2. Instruction File Management

- Keep `copilot-instructions.md` concise — adherence drops when it becomes too long
- Use separate instruction files in `instructions/` for domain-specific rules
- Personal overrides belong in local settings, not shared instruction files

## 3. Progressive Disclosure

- Load context on demand, not up front
- Skill descriptions load automatically (lightweight); full content loads only on invocation
- Feature-specific subagents carry only the skills relevant to their task

## 4. Simplicity First

Vanilla Copilot without complex orchestration is the right choice for smaller tasks.
Only introduce subagents and skills when the task genuinely benefits from them.
