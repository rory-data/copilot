---
description: "Quick-reference workflow guide covering key rules, patterns, and decision frameworks"
applyTo: "**"
---

# Workflow Quick Reference

## Core Workflow Rules

1. **Always start with plan mode** for non-trivial tasks
2. **Keep instruction files concise** — adherence decreases as length increases
3. **Manual compaction at ~50% context** — do not wait for automatic compaction
4. **Keep sub-tasks completable within 50% context**
5. **Commit immediately upon task completion**
6. **Direct Copilot > complex orchestration** for smaller, self-contained tasks

## Architecture Pattern: Skill → Subagent → Task

| Component | Role |
|-----------|------|
| **Skill** | Domain knowledge injected at task start |
| **Subagent** | Orchestrates a workflow using relevant skills |
| **Task** | Single-responsibility unit of work |

## RPI Workflow (Research → Plan → Implement)

```
/rpi:research  →  feasibility analysis + GO/NO-GO
/rpi:plan      →  user stories, architecture, phase breakdown
/rpi:implement →  phase-by-phase execution with validation checkpoints
```

## Model Selection

| Tier | Use For |
|------|---------|
| Fast | Worker agents, frequent invocations, simple generation |
| Standard | Main dev work, orchestration, complex coding |
| Deep reasoning | Architecture decisions, research, maximum reasoning |

## Context Engineering

- Ancestor instruction files load automatically (upward walk from project root)
- Descendant instruction files load lazily on file access
- Root instructions propagate to all subdirectories

## Debugging

- Run terminal commands as background tasks for better log visibility
- Provide screenshots for visual issues
- Check linter and type checker output before reporting completion

## Task Tracking

Use SQL task tracking for multi-step work:

```sql
INSERT INTO todos (id, title, description) VALUES
  ('feature-x', 'Implement feature X', 'Description of the work');

UPDATE todos SET status = 'in_progress' WHERE id = 'feature-x';
UPDATE todos SET status = 'done' WHERE id = 'feature-x';
```

## Community Terminology

| Term | Meaning |
|------|---------|
| **Context bloat** | Too much context loaded at once |
| **Context rot** | Stale or outdated context |
| **Dumb zone** | Last 20% of context window — avoid complex work here |
| **Progressive disclosure** | Load context on demand, not up front |
| **Token burn** | Wasted tokens on irrelevant context |
| **Slot machine method** | Retrying the same approach hoping for a different result — do not do this |
