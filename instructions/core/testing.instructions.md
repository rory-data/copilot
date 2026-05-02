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

1. Write the test first (RED — it should fail)
2. **Run the test and confirm it fails** — never skip this step. A test that passes immediately proves nothing; it may be testing the wrong thing or testing existing behaviour.
3. Write the **minimal** implementation to pass (GREEN) — do not add features beyond what the test requires
4. Run the test and confirm it passes
5. Refactor while keeping tests green (IMPROVE)
6. Verify coverage remains at 80%+

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

Structure every test with Arrange, Act, Assert:

```python
import pytest
from myapp.discounts import calculate_discount
from myapp.models import Order

def test_discount_applies_to_premium_orders() -> None:
    # Arrange
    order = Order(total=200.0, customer_tier="premium")

    # Act
    discount = calculate_discount(order)

    # Assert
    assert discount == 20.0
```

## Test Isolation

- Each test must be independent — no shared mutable state between tests
- Use fixtures (`@pytest.fixture`) for setup and teardown
- Mock external dependencies (databases, HTTP calls, filesystems)

```python
from unittest.mock import AsyncMock, patch
import pytest

@pytest.fixture
def mock_db():
    with patch("myapp.repository.get_db") as mock:
        mock.return_value = AsyncMock()
        yield mock
```

## Troubleshooting Test Failures

1. Check test isolation — is state leaking between tests?
2. Verify mocks are correctly scoped and reset
3. Fix the implementation, not the tests (unless the tests are wrong)
4. Report actual test output when raising failures

## Coverage Configuration

Configure `pytest` with coverage in `pyproject.toml`:

```toml
[tool.pytest.ini_options]
addopts = "--cov=src --cov-report=term-missing --cov-fail-under=80"
```
