---
description: "Coding style rules covering immutability, file organisation, error handling, and input validation"
applyTo: "**"
---

# Coding Style

> Code examples use Python for illustration. For Python-specific conventions, invoke the python-conventions skill.

## Immutability (CRITICAL)

ALWAYS create new objects, NEVER mutate in place. Every state change returns a new value; the
original is unchanged. This eliminates a whole class of bugs from shared mutable state and makes
behaviour easier to reason about. For Python implementations (`{**obj, "key": value}`,
`dataclasses.replace`) see the `python-conventions` skill.

## File Organisation

MANY SMALL FILES > FEW LARGE FILES:

- High cohesion, low coupling
- 200–400 lines typical, 800 lines maximum
- Extract utilities from large modules
- Organise by feature/domain, not by type

## Error Handling

ALWAYS handle errors comprehensively. Catch specific exception types, log with context, and
re-raise as a domain-appropriate error using `raise ... from exc` to preserve the chain.
Never swallow exceptions silently. For Python implementations see the `python-conventions` skill.

## Input Validation

ALWAYS validate user input at trust boundaries. Reject invalid data as early as possible with
clear, specific error messages. For Python, use Pydantic with field validators — see the
`python-conventions` skill for implementation patterns.

## Code Quality Checklist

Before marking work complete:

- [ ] Code is readable and well-named
- [ ] Functions are small (<50 lines)
- [ ] Files are focused (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Proper error handling
- [ ] No `print` statements left in production code (use `logging`)
- [ ] No hardcoded values
- [ ] Immutable patterns used; no in-place mutation
