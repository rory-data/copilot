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
2. Run the test and confirm it fails
3. Write the minimal implementation to pass (GREEN)
4. Run the test and confirm it passes
5. Refactor while keeping tests green (IMPROVE)
6. Verify coverage remains at 80%+

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
