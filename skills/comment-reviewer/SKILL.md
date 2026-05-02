---
name: comment-reviewer
description: Review code comments for factual accuracy, comment rot, misleading content, and value. Use when reviewing a PR that adds or modifies comments, after generating documentation, or when auditing an existing codebase for comment quality. Invoke when the user asks to "check comments", "verify documentation", "find outdated comments", or "review the docs in this PR".
---

# Comment Reviewer

This skill analyses code comments through the lens of a future maintainer encountering the code
months later without the original context. It finds inaccurate, outdated, and low-value comments
before they become technical debt.

## Core Questions for Every Comment

For each comment, ask:

**Factual accuracy**
- Do the documented parameters and return types match the actual function signature?
- Does the described behaviour match what the code actually does?
- Are referenced types, functions, and variables real and used correctly?
- Are claimed edge cases actually handled?
- Are performance or complexity claims correct?

**Long-term value (comment rot risk)**
- Does this comment restate obvious code? → Flag for removal
- Is it explaining *what* rather than *why*? → Flag for removal or rewrite
- Does it reference a temporary state, transitional implementation, or a name that may change? → Flag as rot risk
- Would a future maintainer need this context, or would the code speak for itself?

**Accuracy of completeness claims**
- Are critical preconditions or assumptions documented?
- Are non-obvious side effects mentioned?
- Is complex algorithm rationale captured?
- Is the business logic *reason* recorded for anything non-self-evident?

**Misleading content**
- Is there ambiguous language with multiple valid interpretations?
- Do examples in comments match the current implementation?
- Do TODOs and FIXMEs still apply, or have they been resolved?
- Has code been refactored in a way that makes the comment stale?

## Review Process

### 1. Scope the review
- For a PR: review comments added or modified in the diff
- For a file audit: review all non-trivial comments in the file
- For generated docs: review all authored docstrings

### 2. Categorise findings

**Critical**: Factually wrong or actively misleading
- Documented parameter names that don't exist
- Described behaviour that is the opposite of actual behaviour
- Security-relevant claims that are incorrect

**Improvement**: Incomplete or unclear but not wrong
- Missing context for non-obvious logic
- Ambiguous language
- Docstring that omits a critical parameter

**Remove**: No value or creates noise
- Restates what the code already says clearly
- Changelog-style comments (`# Fixed by Alice, 2023-01-15`)
- Commented-out code

**Rot risk**: Likely to become inaccurate with future changes
- References to internal names likely to be renamed
- Comments tied to temporary implementation decisions
- Assertions about external system behaviour

### 3. Output

Report findings with:
- File and line reference
- Category (Critical / Improvement / Remove / Rot risk)
- Specific problem
- Suggested fix or replacement text (where applicable)

End with a brief summary: total comments reviewed, issues found by category, and any standout
positive examples worth preserving as a pattern.

## Decision Framework

Before flagging a comment, ask: *Would removing this comment make the code harder to understand?*
- If no → flag for removal
- If yes, but the comment is wrong → flag as Critical and rewrite
- If yes, and the comment is correct → keep as-is or improve for clarity

The best comments explain *why* something was done a certain way, or document constraints that
cannot be expressed in code (external API quirks, business rule rationale, performance trade-offs).
The worst comments restate code that already reads clearly.
