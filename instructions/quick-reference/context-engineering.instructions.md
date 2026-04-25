---
description: "Context engineering guide covering instruction loading hierarchy, progressive disclosure, and structure"
applyTo: "**"
---

# Context Engineering

## Instruction File Loading in Monorepos

### Ancestor Loading (Upward)

- Copilot walks UP the directory tree when loading instructions
- All ancestor instruction files are loaded at session start
- Root-level instructions are always available

### Descendant Loading (Downward)

- Subdirectory instruction files use lazy loading
- Loaded only when you interact with files in those directories
- Sibling directories never cross-load

### Recommended Structure

| File                                                        | Contents                                                |
| ----------------------------------------------------------- | ------------------------------------------------------- |
| `.github/copilot-instructions.md`                           | Repo-wide conventions, coding standards, commit formats |
| `~/.copilot/instructions/copilot-instructions.md`           | Global instructions for all sessions                    |
| `~/.copilot/instructions/core/*.instructions.md`            | Domain-specific rules (security, testing, etc.)         |
| `~/.copilot/instructions/language/*.instructions.md`        | Language-specific conventions                           |
| `~/.copilot/instructions/quick-reference/*.instructions.md` | Quick-reference guides                                  |

### Key Rules

- Keep root instruction files concise — longer files reduce adherence
- Instructions in parent directories propagate to all subdirectories automatically
- Denial rules in parent instructions cannot be overridden by child instructions

## Skills and Progressive Disclosure

### Loading Behaviour

- Skill descriptions load into context automatically at startup (lightweight)
- Full skill content loads only when the skill is invoked (on-demand)
- This keeps startup context lean while making expertise available when needed

### Discovery

Skills are discovered from (in priority order):

1. Enterprise-managed skills (highest priority)
2. Personal skills (`~/.copilot/skills/`)
3. Project skills (`.copilot/skills/`)

### Best Practices

- Do not front-load all context at session start — reveal it progressively
- Feature-specific subagents should carry only the skills relevant to their task
- Use skill descriptions as lightweight context; full content on demand only

## Context Window Discipline

```
0%  ──────────────────────── 50% ─────────────── 80% ──── 100%
│  Safe for all tasks        │  Caution: complex │  Avoid  │
│                            │  multi-file work  │  here   │
```

- Below 50%: safe for any task
- 50%–80%: suitable for single-file edits, documentation, simple fixes
- Above 80% (the "dumb zone"): avoid complex refactoring or multi-file features

Compact manually at ~50% rather than waiting for automatic compaction.
