---
name: python-testing-patterns
description: Implement comprehensive testing strategies with pytest, fixtures, mocking, and test-driven development. Use when writing Python tests, setting up test suites, or implementing testing best practices.
---

# Python Testing Patterns

Master pytest fundamentals through essential patterns and practical examples, from basic unit tests to mocking external dependencies.

## When to Use This Skill

- Writing unit tests for Python code
- Setting up test suites and test infrastructure
- Implementing test-driven development (TDD)
- Creating integration tests for APIs and services
- Mocking external dependencies and services
- Testing async code and concurrent operations
- Setting up continuous testing in CI/CD
- Implementing property-based testing
- Testing database operations
- Debugging failing tests

## Core Concepts

### 1. Test Types

- **Unit Tests**: Test individual functions/classes in isolation
- **Integration Tests**: Test interaction between components
- **Functional Tests**: Test complete features end-to-end
- **Performance Tests**: Measure speed and resource usage

### 2. Test Structure (AAA Pattern)

- **Arrange**: Set up test data and preconditions
- **Act**: Execute the code under test
- **Assert**: Verify the results

### 3. Test Coverage

- Measure what code is exercised by tests
- Identify untested code paths
- Aim for meaningful coverage, not just high percentages

### 4. Test Isolation

- Tests should be independent
- No shared state between tests
- Each test should clean up after itself

## Quick Start

```python
# test_example.py
def add(a, b):
    return a + b

def test_add():
    """Basic test example."""
    result = add(2, 3)
    assert result == 5

def test_add_negative():
    """Test with negative numbers."""
    assert add(-1, 1) == 0

# Run with: uv run pytest test_example.py
```

## Fundamental Patterns

### Pattern 1: Basic pytest Tests

```python
# test_calculator.py
import pytest

class Calculator:
    """Simple calculator for testing."""

    def add(self, a: float, b: float) -> float:
        return a + b

    def divide(self, a: float, b: float) -> float:
        if b == 0:
            raise ValueError("Cannot divide by zero")
        return a / b


def test_addition():
    """Test addition."""
    calc = Calculator()
    assert calc.add(2, 3) == 5
    assert calc.add(-1, 1) == 0


def test_division_by_zero():
    """Test division by zero raises error."""
    calc = Calculator()
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        calc.divide(5, 0)
```

### Pattern 2: Fixtures for Setup and Teardown

```python
# test_database.py
import pytest

@pytest.fixture
def db():
    """Fixture that provides connected database."""
    # Setup
    database = {"connected": True}
    yield database
    # Teardown
    database["connected"] = False


def test_database_query(db):
    """Test using fixture."""
    assert db["connected"] is True
```

### Pattern 3: Parametrised Tests

```python
# test_validation.py
import pytest

@pytest.mark.parametrize("email,expected", [
    ("user@example.com", True),
    ("invalid.email", False),
])
def test_email_validation(email, expected):
    """Test email validation with various inputs."""
    assert is_valid(email) == expected
```

### Pattern 4: Mocking with unittest.mock

```python
# test_api_client.py
import pytest
from unittest.mock import Mock, patch

def test_api_call():
    """Test API call with mock."""
    mock_response = Mock()
    mock_response.json.return_value = {"id": 1}

    with patch("requests.get", return_value=mock_response):
        result = get_user(1)
        assert result["id"] == 1
```

### Pattern 5: Testing Exceptions

```python
# test_exceptions.py
import pytest

def test_exception():
    """Test exception is raised."""
    with pytest.raises(ValueError, match="error message"):
        divide(10, 0)
```

## Advanced & Specialised Patterns

For additional testing patterns and advanced techniques, see the reference files:

- **[Logging, Async, Fixtures & more](references/advanced-patterns.md)**: Patterns 6-12 cover specialised testing scenarios, including logging, async, and data validation
- **[Test Debugging](references/debugging.md)**: Debugging techniques and pytest commands
- **[Type-Safe Testing](references/type-checking.md)**: Protocol-based type checking for mocks
- **[Configuration & Dependencies](references/configuration.md)**: Coverage reporting, database testing, CI/CD, pytest configuration
- **[Testing Anti-Patterns](references/anti-patterns.md)**: Common pitfalls to avoid (production pollution, mock testing, etc.)

## Testing Best Practices

### Test Organisation

```python
# tests/
#   __init__.py
#   conftest.py           # Shared fixtures
#   test_unit/            # Unit tests
#   test_integration/     # Integration tests
#   test_e2e/            # End-to-end tests
```

### Test Naming

```python
# Good test names describe what is tested and expected outcome
def test_user_creation_with_valid_data():
    pass

def test_login_fails_with_invalid_password():
    pass

# Bad test names
def test_1():  # Not descriptive
    pass

def test_user():  # Too vague
    pass
```

### Test Markers

```python
import pytest

@pytest.mark.slow
def test_slow_operation():
    pass

@pytest.mark.integration
def test_database_integration():
    pass

# Run with:
# uv run pytest -m slow          # Run only slow tests
# uv run pytest -m "not slow"    # Skip slow tests
```

## Best Practices Summary

1. **Write tests first** (TDD) or alongside code
2. **One assertion per test** when possible
3. **Use descriptive test names** that explain behaviour
4. **Keep tests independent** and isolated
5. **Use fixtures** for setup and teardown
6. **Mock external dependencies** appropriately
7. **Parametrise tests** to reduce duplication
8. **Test edge cases** and error conditions
9. **Measure coverage** but focus on quality
10. **Run tests in CI/CD** on every commit

## Quick Reference

- **pytest documentation**: https://docs.pytest.org/
- **unittest.mock**: https://docs.python.org/3/library/unittest.mock.html
- **hypothesis**: Property-based testing library
- **pytest-asyncio**: Testing async code
- **pytest-cov**: Coverage measurement and reporting
- **pytest-mock**: Enhanced mocking with spec support
- **pydantic**: Data validation and schema testing
- **pandera**: Schema validation for pandas DataFrames
