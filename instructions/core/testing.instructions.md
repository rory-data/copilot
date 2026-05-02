---
description: "Testing requirements and test-driven development workflow"
applyTo: "**"
---

# Testing Requirements

## Minimum Test Coverage: 80%

All three test types are required:

1. **Unit tests** — individual functions, utilities, pure logic
2. **Integration tests** — API endpoints, database operations, external service calls
3. **End-to-end tests** — critical user flows

## Test-Driven Development

MANDATORY workflow for new features and bug fixes:

1. Confirm the behavioural specification before writing any code
2. Write the test first (RED — it should fail) — commit as `test(scope): add tests for <behaviour>`
3. **Run the test and confirm it fails** — never skip this step. A test that passes immediately proves nothing; it may be testing the wrong thing or testing existing behaviour.
4. Write the **minimal** implementation to pass (GREEN) — do not add features beyond what the test requires — commit as `feat(scope): implement <behaviour>`
5. Run the test and confirm it passes
6. Refactor while keeping tests green (IMPROVE) — commit as `refactor(scope): <what changed and why>`
7. Verify coverage remains at 80%+

Tests and implementation go in **separate commits**, in that order. Do not collapse them into one commit — the test commit is the audit trail of intent.

**If code was written before the test:** delete it. Do not keep it as "reference" or "adapt" it while writing tests — that is still writing tests after the fact. Start over from a failing test.

## TDD Rationalisations to Reject

| Excuse | Why it is wrong |
|--------|----------------|
| "Too simple to need a test" | Simple code breaks. A test takes 30 seconds. |
| "I'll write the test after" | Tests written after pass immediately and prove nothing — they test what you built, not what is required. |
| "Already manually tested all edge cases" | Manual testing is ad-hoc, leaves no record, and cannot be re-run when the code changes. |
| "Deleting X hours of work is wasteful" | Sunk cost. The choice is: X more hours with high confidence, or 30 minutes of low-confidence tests on code you cannot trust. |
| "TDD is dogmatic; I'm being pragmatic" | TDD is pragmatic — it finds bugs before commit, prevents regressions, and enables safe refactoring. "Pragmatic shortcuts" mean debugging in production. |

## Test Structure (AAA)

Structure every test with Arrange, Act, Assert. Each section should be visually distinct —
separate them with a blank line or inline comment in longer tests:

- **Arrange**: Set up test data and preconditions
- **Act**: Execute the code under test
- **Assert**: Verify the results

## Test Quality

**Test behaviour, not implementation.** A test that breaks when you rename an internal variable
but keep the external behaviour the same is testing the wrong thing. Ask: "Would this test still
fail if a regression was introduced?" If no — it provides no safety.

**DAMP over DRY for tests**: Tests should be Descriptive and Meaningful Phrases. Prefer slightly
repetitive, self-contained test bodies over heavy fixture abstraction that forces readers to jump
around to understand a test. Test clarity matters more than deduplication.

**Check for brittleness**: Tests tightly coupled to implementation details (mocking private
methods, asserting on exact internal call counts for non-contract behaviour) will break on every
refactor. They create churn, not safety.

## Test Isolation

- Each test must be independent — no shared mutable state between tests
- Use fixtures for setup and teardown
- Mock external dependencies (databases, HTTP calls, filesystems)

## Troubleshooting Test Failures

1. Check test isolation — is state leaking between tests?
2. Verify mocks are correctly scoped and reset
3. Fix the implementation, not the tests (unless the tests are wrong)
4. Report actual test output when raising failures

## Coverage Configuration

Configure coverage to fail below 80% and report missing lines. For pytest configuration see the
`python-testing-patterns` skill or `language/python-testing.instructions.md`.

