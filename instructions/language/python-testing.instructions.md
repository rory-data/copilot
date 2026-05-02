---
description: "Python testing standards: pytest conventions, test scope (only project code — not third-party libs or external systems), and what not to generate"
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
- Always pass `spec=` (or `spec_set=`) when creating mocks — this ensures the mock mirrors the
  real object's interface and catches calls to attributes or methods that don't exist on it

```python
# WRONG: unspecced mock silently accepts any attribute access
mock_client = mocker.MagicMock()
mock_client.ftech_user(1)  # typo — passes silently, hides a bug

# CORRECT: spec catches invalid attribute access at test time
mock_client = mocker.MagicMock(spec=HttpClient)
mock_client.ftech_user(1)  # raises AttributeError immediately
```

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

## Test Scope

Tests must only cover the behaviour of the project's own implemented code. Do not write tests
that verify the correctness of external systems or third-party libraries — those are tested by
their authors. Every test must be value-add: it must fail when your code has a bug, and pass
when it does not.

**Do not test external system behaviour.** Integration tests may connect to real external systems
(databases, HTTP APIs, message queues) to verify your code's interaction with them, but the test
must still assert on *your code's output*, not on how the external system works internally. A
test that only confirms Oracle can persist a row is testing Oracle, not your code. Mark
integration tests with `@pytest.mark.integration` and keep them separate from unit tests.

**Do not test third-party library behaviour.** If you call `json.dumps`, `pendulum.now`, or a
Pydantic validator, do not write a test that simply confirms the library works. Write tests that
confirm your code passes the right inputs and handles the outputs correctly.

```python
# WRONG: tests that pandas can read a CSV (not your code)
def test_load_data() -> None:
    df = pd.read_csv("data/sample.csv")
    assert len(df) > 0  # verifies pandas, not your logic

# CORRECT: tests your transformation logic on controlled input
def test_normalise_column_names() -> None:
    df = pd.DataFrame({"First Name": ["Alice"], "Last Name": ["Smith"]})
    result = normalise_column_names(df)
    assert list(result.columns) == ["first_name", "last_name"]
```

```python
# WRONG: tests that requests.get works (not your code)
def test_fetches_user(mocker) -> None:
    mocker.patch("requests.get", return_value=Mock(status_code=200))
    fetch_user(1)
    # no assertion on YOUR code's output

# CORRECT: tests your parsing logic with a controlled response
def test_fetch_user_parses_name(mocker) -> None:
    mocker.patch("myapp.client.get", return_value={"id": 1, "name": "Alice"})
    user = fetch_user(1)
    assert user.name == "Alice"
```

**The value-add test:** Before writing a test, ask: "If I introduced a bug in my code, would
this test catch it?" If the answer is no — because it only exercises library plumbing or an
external service — delete the test or rewrite it to assert on your code's actual output.
