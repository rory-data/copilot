---
description: "Optimised README generation leveraging project analysis and markdown standards"
---

# README Generator

## Goal

Create comprehensive, appealing README files following **Markdown standards** (see `../instructions/language/markdown.instructions.md`) and **Core Engineering Principles**.

## Quick Reference

- **Standards**: `../instructions/language/markdown.instructions.md`
- **Structure**: Professional open-source project format
- **Quality**: Clear, concise, actionable content
- **Style**: New Zealand English, minimal emoji usage

## Content Strategy

### Project Analysis Phase

1. **Technology Detection**: Analyse project structure and dependencies
2. **Feature Identification**: Extract core functionality and value proposition
3. **User Journey Mapping**: Understand installation and usage flow
4. **Documentation Assessment**: Review existing docs for context

### Structure Template

```markdown
# Project Name

[Brief, compelling description of what the project does]

## Features

- [Key feature 1 with benefit]
- [Key feature 2 with benefit]
- [Key feature 3 with benefit]

## Quick Start

[Minimal steps to get running - target <5 steps]

## Installation

[Detailed setup instructions]

## Usage

[Common use cases with examples]

## Configuration

[Environment setup and customization options]

## Contributing

[How to contribute - reference CONTRIBUTING.md if it exists]

## Support

[Where to get help - issues, discussions, etc.]
```

## Content Guidelines

### Writing Style

- **New Zealand English**: Spelling and grammar
- **Conversational Tone**: Refer to project naturally
- **Concise Language**: Essential information only
- **Active Voice**: Clear, direct instructions

### Technical Standards

- **GFM Syntax**: GitHub Flavored Markdown
- **Admonition Blocks**: Use GitHub's admonition syntax appropriately
- **Code Blocks**: Language-specific syntax highlighting
- **Proper Links**: Valid, accessible link references

### Inspiration Sources

Reference these high-quality READMEs for structure and tone:

- Azure Samples serverless projects
- Professional open-source projects
- Clear, minimal documentation examples

## Exclusions

DO NOT include these sections (dedicated files exist):

- `LICENSE` section
- `CONTRIBUTING` section
- `CHANGELOG` section
- Legal disclaimers or boilerplate

## Quality Checklist

- [ ] Clear value proposition in first paragraph
- [ ] Quick start section gets user running in <5 steps
- [ ] All code examples are tested and functional
- [ ] Links are valid and accessible
- [ ] Markdown validates correctly
- [ ] Images include proper alt text
- [ ] Project logo included if available

## Integration

- **Analysis Tools**: Use codebase analysis for comprehensive project understanding
- **Markdown Validation**: Apply markdown instructions for consistency
- **Engineering Standards**: Follow clean documentation principles
