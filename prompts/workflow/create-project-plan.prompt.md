---
description: "Optimised GitHub project planning leveraging core engineering principles"
---

# GitHub Project Planning Generator

## Goal

Generate comprehensive GitHub project plans following **Workflow Standards** and **Engineering Principles** (see `../instructions/core/`).

Apply Agile methodology with INVEST criteria and value-based prioritization.

## Quick Reference

- **Workflow**: 6-phase loop from `../instructions/core/workflow-standards.instructions.md`
- **Quality**: Engineering principles for maintainable project structure
- **Templates**: Use specification templates for planning artifacts
- **Tools**: Efficient project management patterns

## Agile Work Item Hierarchy

```
Epic (Milestone) → Feature → Story/Enabler → Task
```

### Definitions

- **Epic**: Large business capability spanning multiple features
- **Feature**: Deliverable user-facing functionality within an epic
- **Story**: User-focused requirement delivering independent value
- **Enabler**: Technical infrastructure supporting stories
- **Task**: Implementation-level work breakdown

## INVEST Criteria Application

All work items must be:

- **Independent**: Can be developed separately
- **Negotiable**: Flexible scope and implementation
- **Valuable**: Delivers clear business or technical value
- **Estimable**: Effort can be reasonably estimated
- **Small**: Completable within sprint timeframe
- **Testable**: Clear acceptance criteria and validation

## Planning Phases

### Phase 1: Analyse

1. **Requirements Mapping**: Extract from PRD and technical specifications
2. **Dependency Analysis**: Identify blocking relationships
3. **Scope Assessment**: Generate confidence scores for work items

### Phase 2: Design

1. **Work Breakdown**: Apply INVEST criteria to create optimal work items
2. **Priority Matrix**: Business value vs. effort assessment
3. **Sprint Planning**: Logical grouping and sequencing

### Phase 3-6: Implementation Planning

Create comprehensive GitHub project with:

- Epic/Feature/Story hierarchy
- Dependency linking
- Priority assignment
- Automated tracking workflows

## Output Deliverables

1. **GitHub Project Setup**: Kanban board with work item templates
2. **Issue Templates**: Standardized formats for each work item type
3. **Automation Rules**: Status transitions and dependency tracking
4. **Metrics Dashboard**: Progress tracking and velocity measurement

## Integration Requirements

**Prerequisites**:

- Feature PRD from `create-prd.prompt.md`
- Technical specifications
- Architecture documentation
