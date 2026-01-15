---
description: "Optimised specification creation leveraging core workflow and templates"
---

# Specification Generator

## Goal

Create comprehensive specifications following **Workflow Standards** and **Engineering Principles** (see `../instructions/core/`).

Use **Specification Templates** (see `../instructions/templates/specification-templates.md`) for consistent, AI-ready documentation.

## Quick Reference

- **Templates**: `../instructions/templates/specification-templates.md`
- **Workflow**: 6-phase loop from `../instructions/core/workflow-standards.instructions.md`
- **Quality**: Engineering principles from `../instructions/core/engineering-principles.instructions.md`
- **Tool Usage**: Efficient patterns from `../instructions/core/tool-configurations.instructions.md`

## Execution Strategy

### Phase 1: Analyse

1. **Understand Requirements**: Use EARS notation for clarity
2. **Assess Confidence**: Generate confidence score (0-100%)
3. **Gather Context**: Use `codebase` and `search` tools for understanding

### Phase 2: Design

Based on confidence score:

- **High (>85%)**: Use complete specification template
- **Medium (66-85%)**: Start with core sections, expand iteratively
- **Low (<66%)**: Focus on requirements gathering and research

### Phase 3-6: Implement, Validate, Reflect, Handoff

Follow structured workflow with comprehensive documentation.

## Specification Types

| Type               | Template Section           | Use Case                               |
| ------------------ | -------------------------- | -------------------------------------- |
| **Architecture**   | `spec-architecture-*.md`   | System design, component relationships |
| **Data**           | `spec-data-*.md`           | Data models, schemas, flows            |
| **Process**        | `spec-process-*.md`        | Workflows, procedures, standards       |
| **Tool**           | `spec-tool-*.md`           | Tool configurations, integrations      |
| **Infrastructure** | `spec-infrastructure-*.md` | Deployment, environments, operations   |

## Output Location

Save specifications in `/spec/` directory using naming convention:
`spec-[type]-[descriptive-name].md`
