---
name: markdown-conventions
description: Documentation and content creation standards for Markdown files. Use when writing, reviewing, or formatting Markdown documentation, README files, or technical content.
license: Proprietary. See parent repository LICENSE
---

# Markdown Content Conventions

## Quick Reference

- **Heading levels**: H2-H3 for structure (H1 auto-generated from title)
- **Line length**: 400 characters max for readability
- **Language**: New Zealand English spelling and grammar
- **Code blocks**: Use fenced blocks with language specification
- **Front matter**: YAML metadata at file start

## Content Rules

### 1. Headings

- Use `##` for H2 and `###` for H3
- Ensure hierarchical structure (don't skip levels)
- **Do not use H1** (`#`) - auto-generated from title
- Recommend restructuring if content requires H4
- Strongly recommend restructuring for H5+

```markdown
## Main Section (H2)

Content here...

### Subsection (H3)

More specific content...

### Another Subsection (H3)

Related content...
```

### 2. Lists

- Use `-` for bullet points
- Use `1.` for numbered lists
- Indent nested lists with two spaces
- Ensure proper spacing between list items

```markdown
- First item
- Second item
  - Nested item
  - Another nested item
- Third item

1. First step
2. Second step
3. Third step
```

### 3. Code Blocks

- Use triple backticks for fenced code blocks
- **Always specify language** after opening backticks
- Ensures proper syntax highlighting

````markdown
```python
def hello_world():
    print("Hello, World!")
```

```bash
npm install package-name
```
````

### 4. Links

- Use descriptive link text (not "click here")
- Ensure URLs are valid and accessible
- Use relative paths for internal links

```markdown
Good: See the [API documentation](./api-docs.md) for details.
Bad: Click [here](./api-docs.md) for docs.

Good: Read about [dependency injection patterns](https://example.com/di)
Bad: More info [here](https://example.com/di)
```

### 5. Images

- Include descriptive alt text for accessibility
- Use meaningful file names
- Provide context in surrounding text

```markdown
![Architecture diagram showing microservices communication](./diagrams/architecture.png)

![Database schema with relationships](./schema.png)
```

### 6. Tables

- Use `|` for column separators
- Include header row with alignment
- Ensure columns are properly aligned

```markdown
| Feature | Status | Priority |
| ------- | ------ | -------- |
| API     | Done   | High     |
| UI      | WIP    | Medium   |
| Tests   | Todo   | High     |
```

### 7. Line Length

- **Limit lines to 400 characters** maximum
- Break long lines at 80 characters for improved readability
- Use soft line breaks for paragraphs
- Avoid horizontal scrolling

### 8. Whitespace

- Use blank lines to separate sections
- One blank line between paragraphs
- Two blank lines before major headings (optional)
- Avoid excessive whitespace

## Front Matter

Include YAML front matter at the start of documentation files:

```yaml
---
title: Document Title
description: Brief description of content
author: Team Name
date: 2026-01-01
tags: [documentation, guide]
---
```

## Language Standards

- **Use New Zealand English** for spelling and grammar
  - Colour (not color)
  - Organise (not organize)
  - Analyse (not analyze)
  - Centre (not center)
- Maintain consistent terminology throughout documents
- Use active voice where possible
- Write clear, concise sentences

## Formatting Best Practices

### Emphasis

```markdown
Use _italics_ for emphasis.
Use **bold** for strong emphasis.
Use `code` for inline code or technical terms.
```

### Blockquotes

```markdown
> Important note or quote
> Can span multiple lines
```

### Horizontal Rules

```markdown
Use three or more hyphens, asterisks, or underscores:

---

Content after rule
```

### Task Lists (GitHub Flavored Markdown)

```markdown
- [x] Completed task
- [ ] Pending task
- [ ] Another pending task
```

## Documentation Structure

### README Files

```markdown
# Project Name

Brief description of the project.

## Features

- Feature 1
- Feature 2

## Installation

\`\`\`bash
installation commands
\`\`\`

## Usage

Basic usage examples.

## Contributing

Guidelines for contributors.

## License

License information.
```

### Technical Documentation

```markdown
# Document Title

## Overview

High-level summary.

## Prerequisites

Requirements before proceeding.

## Step-by-Step Guide

### Step 1: Initial Setup

Detailed instructions.

### Step 2: Configuration

Configuration details.

## Troubleshooting

Common issues and solutions.

## References

Links to related resources.
```

## Validation

Before finalizing documentation:

- [ ] Headings follow hierarchical structure (H2 → H3)
- [ ] No H1 headings (auto-generated from title)
- [ ] Code blocks specify language
- [ ] Links use descriptive text
- [ ] Images include alt text
- [ ] Tables are properly formatted
- [ ] Lines don't exceed 400 characters
- [ ] New Zealand English spelling used
- [ ] Front matter included (where required)
- [ ] No excessive whitespace

## Anti-Patterns to Avoid

❌ **Don't do this:**

```markdown
# Using H1 in content (H1 is for title only)

Click [here](link) for more info.

Code without language:
\`\`\`
code here
\`\`\`

![](image.png) # No alt text
```

✅ **Do this instead:**

```markdown
## Using H2 for Main Sections

See the [detailed API documentation](link) for more information.

\`\`\`python
def example():
return "with language specified"
\`\`\`

![API response structure diagram](image.png)
```

## Tools and Resources

- **Linters**: markdownlint, remark-lint
- **Formatters**: Prettier (with markdown plugin)
- **Preview**: VS Code Markdown Preview, Grip (GitHub flavored)
- **Spell Check**: Code Spell Checker (VS Code extension)

## Alignment with Core Principles

- **Clarity**: Well-structured, readable documentation
- **Consistency**: Standardized formatting across all documents
- **Accessibility**: Alt text, descriptive links, proper structure
- **Maintainability**: Easy to update and version control friendly

## References

- [Markdown Guide](https://www.markdownguide.org/)
- [GitHub Flavored Markdown Spec](https://github.github.com/gfm/)
- [CommonMark Spec](https://spec.commonmark.org/)
