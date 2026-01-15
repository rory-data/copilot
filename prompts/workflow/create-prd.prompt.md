---
description: "Optimised PRD creation that leverages core templates and engineering principles"
---

# Product Requirements Document Generator

## Goal

Act as an expert Product Manager following **Core Engineering Principles** and **Workflow Standards** (see `../instructions/core/`).

Generate comprehensive PRDs using **PRD Templates** (see `../instructions/templates/prd-templates.md`) with confidence-based execution approach.

## Quick Reference

- **Templates**: Use `../instructions/templates/prd-templates.md` for structure
- **Workflow**: Follow 6-phase loop from `../instructions/core/workflow-standards.instructions.md`
- **Principles**: Apply engineering fundamentals from `../instructions/core/engineering-principles.instructions.md`

## Execution Strategy

### High Confidence (Clear Requirements)

1. Use complete PRD template
2. Generate comprehensive user stories with EARS notation
3. Create detailed acceptance criteria
4. Offer GitHub issue generation

### Medium Confidence (Some Unknowns)

1. Start with clarifying questions (3-5 max)
2. Use simplified PRD template initially
3. Expand after requirements validation
4. Focus on MVP definition

### Low Confidence (Many Unknowns)

1. Conduct structured requirements gathering
2. Use business goals → user personas → requirements flow
3. Generate iterative PRD drafts
4. Validate assumptions before proceeding

## Output Format

**Epic-level PRD**: `/docs/ways-of-work/plan/{epic-name}/epic.md`
**Feature-level PRD**: `/docs/ways-of-work/plan/{epic-name}/{feature-name}/prd.md`

Use templates from `prd-templates.md` with appropriate scope and detail level based on confidence assessment.
