---
description: "Python testing standards: pytest conventions, what to test, and what not to generate"
applyTo: "**/*.py"
---

# Python Testing Standards

## Test Structure (AAA)

Structure every test with Arrange, Act, Assert. Use inline comments to mark each phase in longer tests:

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

## Coverage Configuration

Configure `pytest` with coverage in `pyproject.toml`:

```toml
[tool.pytest.ini_options]
addopts = "--cov=src --cov-report=term-missing --cov-fail-under=80"
```


- Use `pytest` with fixtures — no `unittest.TestCase` classes
- Mock I/O at the boundary using `pytest-mock` (`mocker` fixture), not `unittest.mock` directly
- Use `@pytest.mark.parametrize` for multiple input variants rather than looping in the test body
- Property-based tests for transform functions: use `hypothesis` with typed strategies derived
  from the function's input schema
- Test file naming: `test_<module_name>.py` mirroring the `src/` structure
- Each test function tests exactly one behaviour — keep assertions focused

```python
import pytest
from hypothesis import given
from hypothesis import strategies as st

@pytest.mark.parametrize("value,expected", [
    (0, True),
    (-1, False),
    (100, True),
])
def test_is_non_negative(value: int, expected: bool) -> None:
    assert is_non_negative(value) == expected


@given(st.lists(st.integers(), min_size=1))
def test_sum_equals_builtin(values: list[int]) -> None:
    assert my_sum(values) == sum(values)
```

## What to Test

For every public function, cover:

- **Happy path** — representative inputs producing expected outputs
- **Null / empty input** — `None`, empty list, empty string, zero
- **Schema contract** — output column names, dtypes, and nullability (critical for DataFrame transforms)
- **Idempotency** — calling the function twice produces the same result
- **Exception paths** — verify the specific exception type and message, not just that an exception was raised

```python
def test_transform_raises_on_empty_dataframe() -> None:
    with pytest.raises(ValueError, match="Input DataFrame must not be empty"):
        transform(pl.DataFrame())
```

## What Not to Generate

- Do not mock the function under test itself
- Do not write assertions that only verify a mock was called — verify the actual output

```python
# WRONG: only verifies the call, not the behaviour
def test_saves_record(mocker) -> None:
    mock_save = mocker.patch("myapp.repo.save")
    create_user("alice")
    mock_save.assert_called_once()  # proves nothing about correctness

# CORRECT: verifies the output
def test_creates_user_with_normalised_name(mocker) -> None:
    mock_save = mocker.patch("myapp.repo.save")
    create_user("  Alice  ")
    saved = mock_save.call_args[0][0]
    assert saved.name == "alice"
```
