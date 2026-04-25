---
name: engineering-workflow
license: Proprietary
description: >
  Structured engineering workflow for complex projects: the 6-phase loop (Analyse → Design →
  Implement → Validate → Reflect → Handoff), confidence-based execution strategy, EARS
  requirements notation, and decision records. Use this skill whenever the user wants to plan a
  feature, architect a system, write requirements, or needs structured guidance for any
  non-trivial engineering task. Also invoke when asked to produce PRDs, user stories,
  specifications, technical designs, implementation task breakdowns, or decision records —
  templates for all of these are in references/templates.md.
---

# Engineering Workflow

A structured approach to complex engineering work. Apply this workflow whenever requirements need
clarification, documentation needs producing, or work spans multiple phases.

## The 6-Phase Loop

| Phase | Goal | Key Activities |
|-------|------|----------------|
| **1. Analyse** | Understand the problem | Read code/docs/logs; write EARS requirements; generate confidence score |
| **2. Design** | Plan the solution | Technical design in `design.md`; error handling matrix; testing strategy; `tasks.md` |
| **3. Implement** | Build it | Code in small increments; dependencies first; update task status in real time |
| **4. Validate** | Verify correctness | Run automated tests; test edge cases; verify performance; document execution traces |
| **5. Reflect** | Improve | Refactor for maintainability; update docs; identify and log technical debt |
| **6. Handoff** | Package for review | Executive summary; pull request with changelog; finalise workspace |

Never skip phases — each builds on the previous.

## Confidence-Based Execution

Before designing, generate a Confidence Score (0–100%) based on how well requirements are understood:

| Score | Indicators | Strategy |
|-------|------------|----------|
| **High (>85%)** | Clear requirements, familiar tech, similar work exists | Full implementation plan, skip PoC |
| **Medium (66–85%)** | Some unknowns, unfamiliar tech, 1–2 unclear dependencies | PoC/MVP first with clear success criteria |
| **Low (<66%)** | Ambiguous requirements, major tech unknowns, significant rework risk | Research phase first, then re-analyse |

## EARS Requirements Notation

Use EARS format when writing requirements in `requirements.md`:

- **Ubiquitous**: `THE SYSTEM SHALL [behavior]`
- **Event-driven**: `WHEN [trigger] THE SYSTEM SHALL [behavior]`
- **State-driven**: `WHILE [state] THE SYSTEM SHALL [behavior]`
- **Unwanted behavior**: `IF [condition] THEN THE SYSTEM SHALL [response]`
- **Optional**: `WHERE [feature included] THE SYSTEM SHALL [behavior]`

Each requirement must be: testable, unambiguous, necessary, feasible, traceable.

## Core Artifacts

For complex projects, maintain these files in the project root or session workspace:

- `requirements.md` — EARS requirements and acceptance criteria
- `design.md` — Technical architecture and implementation considerations
- `tasks.md` — Detailed, trackable implementation plan

## Decision Records

Document significant architectural and technical choices:

```markdown
### Decision — [date]: [brief title]

**Decision**: What was decided
**Context**: Situation requiring the decision and the data driving it
**Options**: Alternatives considered with brief pros/cons
**Rationale**: Why the chosen option is superior, with trade-offs stated explicitly
**Impact**: Consequences for implementation, maintainability, and performance
**Review**: Conditions or schedule for reassessing this decision
```

## Tool Selection by Phase

| Phase | Primary Focus | Tools |
|-------|--------------|-------|
| Analyse | Research and reading | grep, glob, view, bash, web_fetch |
| Design | Mapping dependencies | view, grep, ask_user, create (plan files) |
| Implement | Code changes | edit, create, bash |
| Validate | Testing and verification | bash (run tests), view (check output) |
| Reflect | Code review | view, edit, grep |
| Handoff | Summarise and commit | bash (git), create (PR description) |

## Troubleshooting Protocol

When encountering errors or blockers:

1. **Re-analyse** — revisit requirements and constraints
2. **Re-design** — update the technical design and dependencies
3. **Re-plan** — adjust the implementation task list
4. **Retry** — re-execute with corrected parameters
5. **Escalate** — if issues persist after retries, surface blockers to the user

## Templates

See `references/templates.md` for complete, ready-to-use templates:

- `requirements.md` — EARS requirements with NFRs, constraints, and success criteria
- `design.md` — Technical design with architecture, APIs, error handling, testing strategy
- `tasks.md` — Phased implementation tasks with dependencies and acceptance criteria
- PRD (Product Requirements Document) — full template with goals, personas, user stories
- User story and epic breakdown templates
- Action documentation and decision record templates
