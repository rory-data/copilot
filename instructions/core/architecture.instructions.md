---
description: "Skill and subagent architecture patterns for structured, reusable workflows"
applyTo: "**"
---

# Skill and Subagent Architecture

## Pattern Overview

| Component | Role | How It Is Invoked |
|-----------|------|-------------------|
| **Skill** | Domain knowledge and conventions | Referenced in instructions or invoked by name |
| **Subagent** | Orchestrates a workflow using relevant skills | Launched via the task tool |
| **Task** | Single-responsibility work unit | Delegated to a subagent or executed inline |

## When to Use This Pattern

- Multi-step workflows requiring coordination across phases
- Domain-specific knowledge that should be injected consistently
- Sequential tasks with validation checkpoints between phases
- Reusable components that apply across multiple projects

## Why It Works

- **Progressive disclosure**: Context loaded only when needed — skill descriptions are lightweight;
  full content loads on invocation
- **Single execution context**: Subagent maintains state across phases
- **Clean separation**: Each component has a single clear responsibility
- **Reusability**: Skills shared across subagents and projects

## RPI Workflow (Research → Plan → Implement)

For non-trivial features, follow three phases with explicit validation between each:

### Phase 1 — Research
- Assess feasibility
- Identify dependencies, risks, and constraints
- Produce a GO/NO-GO verdict before proceeding

### Phase 2 — Plan
- Define user stories and acceptance criteria
- Specify technical architecture and interfaces
- Break implementation into phases, each completable within a focused session

### Phase 3 — Implement
- Execute phase-by-phase
- Validate (tests, linter, type checker) after each phase
- Commit upon task completion; do not batch across phases

## Directory Structure for RPI

```
docs/rpi/{feature-slug}/
  REQUEST.md          # Initial spec
  research/           # Feasibility + GO/NO-GO
  plan/               # User stories, UX, architecture
  implement/          # Execution records per phase
```

## Subagent Rules

- Subagents CANNOT invoke other subagents via bash — use the task tool with explicit parameters
- Keep subagent tasks focused and completable in a single context window
- Use lightweight models for worker agents; reserve deep-reasoning models for orchestration
