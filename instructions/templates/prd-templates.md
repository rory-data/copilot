---
description: "Product Requirements Document templates and user story formats"
applyTo: "**"
---

# PRD Templates

## Complete PRD Template

```markdown
---
title: PRD - [Project Title]
version: 1.0
date_created: [YYYY-MM-DD]
product_manager: [Name]
stakeholders: [List of key stakeholders]
---

# PRD: [Project Title]

## 1. Product Overview

### 1.1 Document Title and Version

- **PRD**: [Project Title]
- **Version**: 1.0
- **Date**: [YYYY-MM-DD]
- **Owner**: [Product Manager Name]

### 1.2 Product Summary

[Brief overview in 2-3 short paragraphs describing the product's purpose, target audience, and key value proposition]

## 2. Goals

### 2.1 Business Goals

- [Specific business objective 1]
- [Specific business objective 2]
- [Specific business objective 3]

### 2.2 User Goals

- [What users want to achieve 1]
- [What users want to achieve 2]
- [What users want to achieve 3]

### 2.3 Non-Goals

- [What this project explicitly will NOT do]
- [Scope boundaries and excluded features]
- [Future considerations outside current scope]

## 3. User Personas

### 3.1 Key User Types

- [Primary user type 1]
- [Primary user type 2]
- [Secondary user type 3]

### 3.2 Basic Persona Details

- **[Persona Name]**: [Description of user type, needs, pain points, and goals]
- **[Persona Name]**: [Description of user type, needs, pain points, and goals]

### 3.3 Role-Based Access

- **[Role Name]**: [Permissions, capabilities, and access levels]
- **[Role Name]**: [Permissions, capabilities, and access levels]

## 4. Functional Requirements

### [Feature Name] (Priority: High/Medium/Low)

[Specific requirements for this feature, including:]

- [Functional requirement 1]
- [Functional requirement 2]
- [Integration requirements]
- [Data requirements]

### [Feature Name] (Priority: High/Medium/Low)

[Additional feature requirements...]

## 5. User Experience

### 5.1 Entry Points & First-Time User Flow

- [How users discover and access the product]
- [Onboarding process and initial setup]
- [First-run experience requirements]

### 5.2 Core Experience

- **[Step Name]**: [Description of key user interaction]

  - How this ensures a positive user experience
  - Expected user behaviour and system response

- **[Step Name]**: [Description of subsequent key interaction]
  - User experience considerations and benefits

### 5.3 Advanced Features & Edge Cases

- [Complex user scenarios and how they're handled]
- [Error states and recovery processes]
- [Power user features and advanced workflows]

### 5.4 UI/UX Highlights

- [Key interface design requirements]
- [Accessibility considerations]
- [Mobile/responsive design needs]
- [Visual design principles]

## 6. Narrative

[Concise paragraph describing the end-to-end user journey, highlighting key benefits and positive outcomes users will experience]

## 7. Success Metrics

### 7.1 User-Centric Metrics

- [User engagement metric] - Target: [specific value]
- [User satisfaction metric] - Target: [specific value]
- [User retention metric] - Target: [specific value]

### 7.2 Business Metrics

- [Revenue/cost impact] - Target: [specific value]
- [Business process improvement] - Target: [specific value]
- [Market/competitive metric] - Target: [specific value]

### 7.3 Technical Metrics

- [Performance requirement] - Target: [specific value]
- [Reliability requirement] - Target: [specific value]
- [Scalability requirement] - Target: [specific value]

## 8. Technical Considerations

### 8.1 Integration Points

- [External system 1] - [Integration method and requirements]
- [External system 2] - [Integration method and requirements]
- [Internal system dependencies]

### 8.2 Data Storage & Privacy

- [Data types and storage requirements]
- [Privacy and compliance considerations]
- [Data retention and deletion policies]
- [Security requirements]

### 8.3 Scalability & Performance

- [Expected load and growth projections]
- [Performance requirements and SLAs]
- [Scalability approach and limitations]

### 8.4 Potential Challenges

- [Technical risk 1] - [Mitigation strategy]
- [Technical risk 2] - [Mitigation strategy]
- [Integration challenges and solutions]

## 9. Milestones & Sequencing

### 9.1 Project Estimate

- **Size**: [Small/Medium/Large] - [X weeks/months]
- **Complexity**: [Technical complexity assessment]

### 9.2 Team Size & Composition

- **Team Size**: [Number] people
- **Roles**: [Developer, Designer, QA, DevOps, etc.]
- **Key Skills**: [Required expertise and specializations]

### 9.3 Suggested Phases

- **Phase 1**: [MVP/Core features] ([X weeks])

  - [Key deliverable 1]
  - [Key deliverable 2]

- **Phase 2**: [Enhancement features] ([X weeks])

  - [Key deliverable 1]
  - [Key deliverable 2]

- **Phase 3**: [Advanced features] ([X weeks])
  - [Key deliverable 1]
  - [Key deliverable 2]

## 10. User Stories

### 10.1 [User Story Title]

- **ID**: GH-001
- **As a** [user type], **I want** [functionality] **so that** [benefit/value]
- **Acceptance Criteria**:
  - [ ] [Specific testable criteria 1]
  - [ ] [Specific testable criteria 2]
  - [ ] [Specific testable criteria 3]
- **Priority**: High/Medium/Low
- **Story Points**: [Estimation if using Agile]

### 10.2 [User Story Title]

- **ID**: GH-002
- **As a** [user type], **I want** [functionality] **so that** [benefit/value]
- **Acceptance Criteria**:
  - [ ] [Specific testable criteria 1]
  - [ ] [Specific testable criteria 2]
  - [ ] [Specific testable criteria 3]
- **Priority**: High/Medium/Low
- **Story Points**: [Estimation if using Agile]

[Continue for all user stories...]

---

## Appendices

### A. Wireframes and Mockups

[References to design documents]

### B. Technical Architecture

[References to technical design documents]

### C. Competitive Analysis

[Brief comparison with existing solutions]

### D. Glossary

[Definition of domain-specific terms]
```

## User Story Template

```markdown
### [Story ID]: [Descriptive Title]

**As a** [type of user]  
**I want** [some functionality]  
**So that** [some benefit is achieved]

**Acceptance Criteria:**

- [ ] Given [context/precondition], when [action], then [expected outcome]
- [ ] Given [context/precondition], when [action], then [expected outcome]
- [ ] Given [context/precondition], when [action], then [expected outcome]

**Priority:** [High/Medium/Low]
**Story Points:** [If using story point estimation]
**Dependencies:** [Other stories this depends on]
**Notes:** [Additional context or considerations]
```

## Epic Breakdown Template

```markdown
# Epic: [Epic Name]

## Overview

[Brief description of the epic and its business value]

## Success Criteria

- [ ] [Measurable outcome 1]
- [ ] [Measurable outcome 2]

## User Stories in This Epic

### Theme 1: [Group of related stories]

- **GH-001**: [Story title] - [Priority]
- **GH-002**: [Story title] - [Priority]
- **GH-003**: [Story title] - [Priority]

### Theme 2: [Group of related stories]

- **GH-004**: [Story title] - [Priority]
- **GH-005**: [Story title] - [Priority]

## Dependencies

- [External dependency 1]
- [External dependency 2]

## Risks & Mitigation

- **Risk**: [Description] - **Mitigation**: [Strategy]
- **Risk**: [Description] - **Mitigation**: [Strategy]
```

## GitHub Issue Template (from PRD)

```markdown
---
name: User Story
about: User story generated from PRD
labels: user-story, needs-refinement
---

## User Story

**As a** [user type]  
**I want** [functionality]  
**So that** [benefit]

## Acceptance Criteria

- [ ] [Testable criteria 1]
- [ ] [Testable criteria 2]
- [ ] [Testable criteria 3]

## Definition of Done

- [ ] Acceptance criteria met
- [ ] Code reviewed and approved
- [ ] Tests written and passing
- [ ] Documentation updated
- [ ] QA testing completed
- [ ] Deployed to staging

## Additional Context

[Any relevant context, mockups, or technical notes]

## Related Issues

- Related to: #[issue-number]
- Blocks: #[issue-number]
- Blocked by: #[issue-number]
```

## Quick PRD Checklist

Before finalising any PRD, ensure:

- [ ] **Clear Problem Statement**: What problem does this solve?
- [ ] **Defined Success Metrics**: How will we measure success?
- [ ] **User Personas Identified**: Who are we building for?
- [ ] **User Stories Complete**: All user interactions covered?
- [ ] **Technical Feasibility**: Engineering team consulted?
- [ ] **Resource Requirements**: Team size and timeline realistic?
- [ ] **Dependencies Identified**: What do we need from others?
- [ ] **Risk Assessment**: What could go wrong?
- [ ] **Scope Boundaries**: What's in and out of scope?
- [ ] **Review Process**: Who needs to approve this?
