---
description: "Common workflow patterns and decision-making frameworks"
applyTo: "**"
---

# Workflow Standards

## Quick Reference

- **6-Phase Loop**: Analyse → Design → Implement → Validate → Reflect → Handoff
- **Confidence-Based Execution**: High (>85%) = full implementation, Medium (66-85%) = PoC first, Low (<66%) = research first
- **Confidence Indicators**: High = clear requirements + familiar tech; Medium = some unknowns; Low = multiple major unknowns
- **Never Skip Steps**: Each phase must be completed before proceeding

## Core Artifacts

For complex work, maintain these:

- **`requirements.md`**: User stories and acceptance criteria in EARS notation
- **`design.md`**: Technical architecture and implementation considerations
- **`tasks.md`**: Detailed, trackable implementation plan

For detailed documentation templates (Action Documentation, Decision Records, etc.), see `instructions/templates/`.

## Execution Workflow (6-Phase Loop)

### Phase 1: ANALYSE

**Never skip. Reduce ambiguity.**

**Objective**: Understand the problem and produce clear, testable requirements.

**Key Activities**:

- Read all provided code, documentation, tests, logs
- Define requirements in EARS notation: `WHEN [condition], THE SYSTEM SHALL [behavior]`
- Identify dependencies, constraints, edge cases
- Map data flows and system interactions
- Generate Confidence Score (0-100%) based on requirement clarity

**Confidence Scoring Guidance**:

- **High (>85%)**: Requirements are clear and documented; technology stack is familiar; similar implementations exist; success criteria are unambiguous
- **Medium (66-85%)**: Some requirements need clarification; some technology is unfamiliar; 1-2 unknown dependencies; success criteria partially clear
- **Low (<66%)**: Requirements are ambiguous or incomplete; major technology unknowns; multiple dependencies unclear; significant risk of rework

**Critical Constraint**: Do not proceed until all requirements are documented.

### Phase 2: DESIGN

**Adaptive execution based on Confidence Score.**

**Confidence-Based Strategy**:

- **High (>85%)**: Comprehensive implementation plan, skip PoC
- **Medium (66-85%)**: Prioritise PoC/MVP with clear success criteria
- **Low (<66%)**: Research phase, then re-analyse

**Key Activities**:

- Document technical design in `design.md`
- Create error handling matrix
- Define unit testing strategy
- Create implementation plan in `tasks.md`

**Critical Constraint**: Do not implement until design is complete and validated.

### Phase 3: IMPLEMENT

**Production-quality code according to design.**

**Key Activities**:

- Code in small, testable increments
- Implement from dependencies upward
- Follow language conventions (see language/ directory)
- Add meaningful comments (explain WHY, not WHAT)
- Update task status in real time

### Phase 4: VALIDATE

**Verify implementation meets all requirements.**

**Key Activities**:

- Execute automated tests with full documentation
- Perform manual verification if necessary
- Test edge cases and error handling
- Verify performance metrics
- Document execution traces

### Phase 5: REFLECT

**Improve codebase and documentation.**

**Key Activities**:

- Refactor for maintainability
- Update all project documentation
- Identify and document improvements
- Validate success criteria
- Create technical debt issues

### Phase 6: HANDOFF

**Package work for review and deployment.**

**Key Activities**:

- Generate executive summary
- Prepare pull request with changelog
- Finalise workspace and archive artifacts
- Document transition to next task

## Tool Selection by Phase

| Phase         | Primary Tool Group  | Secondary Tools | Focus                                      |
| ------------- | ------------------- | --------------- | ------------------------------------------ |
| **Analyse**   | Research + Analysis | -               | Understand requirements, identify unknowns |
| **Design**    | Analysis            | Research        | Map dependencies, plan approach            |
| **Implement** | Implementation      | Analysis        | Code changes, build, execution             |
| **Validate**  | Validation          | Implementation  | Testing, verification, edge cases          |
| **Reflect**   | Analysis            | Implementation  | Code review, documentation                 |
| **Handoff**   | Research            | -               | Summarise, document, prepare PR            |

## EARS Notation Reference

**Easy Approach to Requirements Syntax** - Standard format:

- **Ubiquitous**: `THE SYSTEM SHALL [expected behavior]`
- **Event-driven**: `WHEN [trigger event] THE SYSTEM SHALL [expected behavior]`
- **State-driven**: `WHILE [in specific state] THE SYSTEM SHALL [expected behavior]`
- **Unwanted behavior**: `IF [unwanted condition] THEN THE SYSTEM SHALL [required response]`
- **Optional**: `WHERE [feature is included] THE SYSTEM SHALL [expected behavior]`

Each requirement must be: **Testable**, **Unambiguous**, **Necessary**, **Feasible**, **Traceable**

## Decision Record Examples

When documenting significant architectural choices or trade-offs:

**Example 1: Technology Choice**

```markdown
### Decision - 2024-11-16: Use PostgreSQL over MongoDB

**Decision**: Adopt PostgreSQL as primary data store

**Context**: Building financial transaction system requiring ACID guarantees and complex relational queries. Team has PostgreSQL experience; MongoDB was considered for flexibility but risks data consistency.

**Options**:

- PostgreSQL (chosen): Strong ACID, mature ecosystem, team expertise
- MongoDB: Flexible schema, but lack of transactions in older versions
- Firebase: Managed service but vendor lock-in and cost concerns

**Rationale**: ACID compliance is non-negotiable for financial data. PostgreSQL's mature tooling and team expertise reduce risk.

**Impact**: Requires schema design upfront but eliminates consistency issues; training overhead minimal.

**Review**: Assess in Q2 2025 if scaling needs emerge.
```

**Example 2: Architectural Pattern**

```markdown
### Decision - 2024-11-16: Repository pattern for data access

**Decision**: Implement repository pattern for database abstraction

**Context**: Refactoring data layer to improve testability. Current code directly calls database functions scattered across codebase.

**Options**:

- Repository pattern (chosen): Clean separation, easier mocking, centralised queries
- Direct DAO calls: Simpler implementation but tighter coupling
- ORM only: Less control over queries, heavier dependency

**Rationale**: Improves testability (can mock repositories) and maintains flexibility for future database changes.

**Impact**: Initial +2 days implementation, long-term maintenance improvement. Enables parallel feature development with mocked data.

**Review**: Evaluate test coverage improvement in next sprint.
```

## Troubleshooting Protocol

When encountering errors or blockers:

1. **Re-analyse**: Revisit requirements and constraints
2. **Re-design**: Update technical design and dependencies
3. **Re-plan**: Adjust implementation plan
4. **Retry execution**: Re-execute with corrected parameters
5. **Escalate**: If issues persist after retries

**Never proceed with unresolved errors or ambiguities.**
