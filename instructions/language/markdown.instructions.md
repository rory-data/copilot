---
description: "Documentation and content creation standards"
applyTo: "**/*.md"
---

# Markdown Standards

- **Language**: Use New Zealand English for spelling and grammar.
- **H1 usage**: H1 is reserved for document titles only — never use H1 for section headings
  within a document. Use H2 and below for sections.
- **Code blocks**: Use fenced code blocks for all code snippets. Always specify the language
  identifier after the opening backticks for syntax highlighting (e.g., ` ```python `, ` ```bash `).
- **Links**: Use descriptive link text — no "click here", "read more", or bare URLs. The link
  text should describe the destination.
- **Tables**: Use pipe tables with header rows. Align columns consistently.
- **Lists**: Use `-` for unordered lists. Keep list items parallel in grammatical structure.
- **Headings**: Use sentence case for headings (capitalise the first word only, plus proper nouns).
  Do not add trailing punctuation to headings.
- **Blank lines**: Separate sections, code blocks, and tables from surrounding content with a
  blank line above and below.
- **No bare HTML**: Do not embed raw HTML in Markdown unless unavoidable. Prefer Markdown syntax.
- **Line length**: Keep prose lines under 100 characters for readable diffs.

For comprehensive style guidance, invoke the `markdown-conventions` skill.
