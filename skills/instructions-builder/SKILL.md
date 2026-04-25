---
name: instructions-builder
description: Create, organise, and refine custom instruction files for GitHub Copilot using a modular, rules-based approach. Invoke this skill whenever the user wants to write Copilot instructions, add rules to an existing instructions directory, convert team conventions into instruction files, scope instructions to specific file types or directories, migrate rules from another AI tool into Copilot format, or asks how to structure a .github/copilot-instructions.md or instructions/ directory. Also invoke when someone says "create instructions for X", "write a rule for Y", "add a Copilot rule about Z", or "how should I organise my Copilot instructions".
---

# Instructions Builder

This skill creates and organises GitHub Copilot instruction files. These files tell Copilot
how to behave in a codebase — coding conventions, architecture rules, tooling preferences,
and workflow guidance.

## Instruction File Format

Every instruction file is a Markdown file with YAML front matter:

```markdown
---
description: "What this file governs — shown to the model as context"
applyTo: "**"   # glob pattern: which files trigger this instruction
---

# Topic Name

...rules...
```

**`applyTo` patterns:**
- `"**"` — load unconditionally for every interaction
- `"**/*.py"` — only when working with Python files
- `"src/**/*.ts"` — only for TypeScript files inside src/
- `"tests/**"` — only when working in the tests directory
- `"**/*.md"` — only for Markdown files

Rules without a path (or with `"**"`) load for every session. Path-scoped rules
load only when Copilot is working with matching files, keeping context lean.

## Directory Structure

```
instructions/                          # or .github/ for repo-level
├── copilot-instructions.md            # global, always-on behavioural rules
├── core/                              # cross-cutting standards
│   ├── coding-style.instructions.md
│   ├── error-handling.instructions.md
│   └── security.instructions.md
├── language/                          # language-specific, path-scoped
│   ├── python.instructions.md         # applyTo: "**/*.py"
│   └── typescript.instructions.md     # applyTo: "**/*.ts"
└── domain/                            # project-specific topics
    ├── api-design.instructions.md
    └── testing.instructions.md
```

**Naming convention:** Use `<topic>.instructions.md` for all instruction files.
Use descriptive names — `testing.instructions.md`, not `test.md`.

## Workflow

### Step 1 — Understand What to Capture

Before writing anything, interview the user:

1. **What do you want Copilot to know or do differently?** (coding style, library preferences,
   architecture patterns, workflow rules, etc.)
2. **Which file types or directories should these rules apply to?** (global vs. scoped)
3. **Do you have existing rules, READMEs, or conventions to convert?** (paste them in)
4. **Is this a new instructions directory, or adding to an existing one?** (check for overlap)

Look at the existing `instructions/` directory if one is present before proposing new files —
avoid duplicating what's already there.

### Step 2 — Propose File Organisation

Split the rules into topics. Each file should:
- Cover exactly **one topic** — testing, error handling, API design, etc.
- Be **concise**: 20–80 lines is ideal; 150 lines maximum
- **Not duplicate** content already in `copilot-instructions.md` or other instruction files

**When to combine vs. split:** Group tightly related rules together. Don't create separate files
for each library — group them by concern instead. For example, Python conventions for Pydantic,
SQLAlchemy, and Ruff belong in a single `python.instructions.md` file scoped to `**/*.py`, not
in `pydantic.instructions.md`, `sqlalchemy.instructions.md`, and `linting.instructions.md`. One
file per technology is usually too granular; one file per language or domain concern is right.

Propose an outline to the user before writing:

```
I'll create three files:
1. core/error-handling.instructions.md  — applyTo: "**"
2. language/python.instructions.md      — applyTo: "**/*.py"
3. domain/api-design.instructions.md    — applyTo: "src/api/**"
```

Confirm the outline before writing the files.

### Step 3 — Write the Instruction Files

For each file:

1. Open with the YAML front matter block (description + applyTo)
2. Use a single H1 heading matching the topic
3. Write rules as **concrete, verifiable instructions** — not principles
4. Group related rules under H2 headings
5. Include code examples for non-obvious rules

#### Content quality checklist

- [ ] Every rule can be verified: "Use Pydantic for input validation" ✓ vs. "Write good code" ✗
- [ ] No redundancy with other instruction files
- [ ] Code examples use the project's primary language
- [ ] `applyTo` scope is as tight as it can be while still covering the relevant files
- [ ] File is under 150 lines

### Step 4 — Scope Review

After writing, review each file's `applyTo`:

- Should this rule apply everywhere, or only to certain files?
- Rules about Python type hints → `"**/*.py"`, not `"**"`
- Rules about commit messages → `"**"` (no file filter helps here)
- Rules about React component patterns → `"src/components/**/*.tsx"`

Tighter scoping keeps context lean. Rules that always apply should genuinely
need to apply everywhere.

## Content Patterns

### DO: Concrete, actionable rules

```markdown
## Error Handling

- Raise `ValueError` for invalid inputs; raise `RuntimeError` for unrecoverable states
- Never use bare `except:`; always catch specific exception types
- Use `logging.exception()` inside `except` blocks to preserve stack traces
```

### DON'T: Vague principles (already in engineering-principles.instructions.md)

```markdown
## Error Handling

- Write clean, readable code
- Handle errors properly
- Follow best practices
```

### DO: Scoped language rules

```markdown
---
applyTo: "**/*.py"
description: "Python-specific conventions and tooling"
---

## Imports

- Use `from __future__ import annotations` for forward references
- Group imports: stdlib → third-party → local, separated by blank lines
- Never use wildcard imports (`from module import *`)
```

### DO: Code examples for non-obvious rules

```markdown
## Repository Pattern

Always inject the repository as a dependency — never instantiate it inside a service:

```python
# CORRECT
class OrderService:
    def __init__(self, orders: OrderRepository) -> None:
        self.orders = orders
```

## Migrating from Another Tool

When converting rules from Claude Code (`.claude/rules/`), VS Code snippets, or
team READMEs:

1. Group the source content by topic
2. Identify which rules need scoping (language-specific, directory-specific)
3. Remove anything already covered by the global instruction file
4. Rewrite as imperative instructions ("Use X", "Never Y") rather than descriptions
5. Add code examples where the rule isn't obvious from the text

**Migration is an enrichment opportunity.** Don't just reformat — the source file is likely
terse notes or bullet points. This is your chance to add CORRECT/WRONG code examples, group
rules under H2 headings, and add context that helps Copilot apply the rule correctly. A
migrated file with 10 well-illustrated rules is more valuable than a reformatted copy of the
original.

**Claude Code frontmatter → Copilot frontmatter:**
```yaml
# Claude Code (.claude/rules/python.md)
---
paths:
  - "**/*.py"
---

# Copilot (instructions/language/python.instructions.md)
---
description: "Python-specific conventions"
applyTo: "**/*.py"
---
```

**Enrichment example** — a terse source rule becomes a useful instruction:

```
# Before (terse note):
- Always parameterize SQL queries

# After (enriched instruction):
## SQL Safety

Always parameterize queries — never use f-strings or `.format()` to build SQL:

```python
# CORRECT
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# WRONG
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")  # SQL injection risk
```
```
