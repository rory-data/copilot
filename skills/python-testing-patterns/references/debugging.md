# Test Debugging

Pattern 12 demonstrates debugging techniques, output inspection, and pytest debugging commands for troubleshooting failing tests.

## Pattern 12: Test Debugging and Inspection

### Print Debugging with capsys

Capture print output during tests to inspect intermediate values:

```python
# test_debugging.py
import pytest

def complex_calculation(x: int, y: int) -> int:
    """Complex calculation that might fail."""
    if x < 0:
        raise ValueError("x cannot be negative")
    return (x + y) * 2


def test_with_debug_output(capsys):
    """Test using print debugging and capsys."""
    result = complex_calculation(5, 3)

    print(f"Result: {result}")
    print(f"Expected: 16")

    captured = capsys.readouterr()
    assert "Result: 16" in captured.out
    assert result == 16
```

### Using pytest.set_trace() for Debugging

Drop into a debugger when a test runs:

```python
def test_with_pdb_marker():
    """Mark test for debugging with --pdb flag."""
    # Run with: uv run pytest -v test_debugging.py::test_with_pdb_marker --pdb
    # Execution will stop at the pytest.set_trace() call

    result = complex_calculation(10, 5)
    assert result == 30

    # Uncomment to debug:
    # pytest.set_trace()  # Debugger will start here
```

### Fixture Introspection

Inspect fixture values during test execution:

```python
def test_fixture_introspection(capsys):
    """Inspect fixture values during test."""
    test_data = {"key": "value", "number": 42}

    # Use capsys to see fixture content
    print(f"Test data: {test_data}")
    captured = capsys.readouterr()

    assert "key" in captured.out
```

### Parametrised Test Debugging

View all parameter combinations with verbose output:

```python
@pytest.mark.parametrize("value,expected", [
    (2, 4),
    (3, 6),
    (5, 10),
])
def test_with_pytest_verbose(value, expected, capsys):
    """Run with uv run pytest -vv to see all parameter combinations."""
    result = complex_calculation(value, 0)
    assert result == expected
    # Run with: uv run pytest -vv test_debugging.py::test_with_pytest_verbose
```

## Debugging Commands

Common pytest commands for debugging (prepend with `uv run` in uv-managed projects):

### Verbosity and Output

| Command             | Purpose                                            |
| ------------------- | -------------------------------------------------- |
| `uv run pytest -v`  | Verbose output showing test names and results      |
| `uv run pytest -vv` | Very verbose, shows parametrised test combinations |
| `uv run pytest -s`  | Show print statements (default is suppressed)      |
| `uv run pytest -q`  | Quiet mode, minimal output                         |

### Interactive Debugging

| Command                                                              | Purpose                             |
| -------------------------------------------------------------------- | ----------------------------------- |
| `uv run pytest --pdb`                                                | Drop to debugger on test failure    |
| `uv run pytest --pdb --pdbcls=IPython.terminal.debugger:TerminalPdb` | Use IPython debugger                |
| `uv run pytest --trace`                                              | Drop to debugger at start of test   |
| `uv run pytest --lf`                                                 | Run last failed test                |
| `uv run pytest --ff`                                                 | Run failed tests first, then others |

### Test Selection

| Command                                                    | Purpose                           |
| ---------------------------------------------------------- | --------------------------------- |
| `uv run pytest -k "pattern"`                               | Run tests matching pattern        |
| `uv run pytest tests/test_file.py::test_name`              | Run specific test                 |
| `uv run pytest tests/test_file.py::TestClass::test_method` | Run test in class                 |
| `uv run pytest -m slow`                                    | Run tests with specific marker    |
| `uv run pytest -m "not slow"`                              | Run tests without specific marker |

### Output and Reporting

| Command                        | Purpose                |
| ------------------------------ | ---------------------- |
| `uv run pytest --tb=short`     | Short traceback format |
| `uv run pytest --tb=long`      | Long traceback format  |
| `uv run pytest --tb=no`        | No traceback           |
| `uv run pytest -x`             | Stop on first failure  |
| `uv run pytest --maxfail=3`    | Stop after 3 failures  |
| `uv run pytest --durations=10` | Show 10 slowest tests  |

## Debugging Strategy

When a test fails:

1. **Understand the failure**: Read the error message and traceback
2. **Isolate the test**: Run with `uv run pytest -k "test_name"`
3. **Add print statements**: Use `capsys` fixture to inspect values
4. **Use --pdb**: Drop into debugger to step through code
5. **Check assumptions**: Verify fixtures and mocks have expected values
6. **Review test organisation**: Ensure tests are independent and properly isolated

## Common Issues and Solutions

### Test passes locally but fails in CI

- Check environment variables (`uv run pytest --co` shows collected tests)
- Verify dependency versions
- Check for timing-related issues (use fixtures with proper setup/teardown)
- Review file path handling (use `pathlib.Path`)

### Flaky tests

- Isolate test state (use fixtures)
- Add delays/retries for time-dependent code
- Mock external dependencies
- Avoid shared state between tests

### Hard to debug mock behaviour

- Use `mock.call_args` to inspect calls
- Add assertions on `call_count` and `call_args_list`
- Print mock configuration with `print(mock.mock_calls)`
- Use `spec` parameter to catch typos early

### Fixture setup issues

- Use `uv run pytest -vv` to see fixture order
- Add print statements in fixtures
- Check fixture scope (function, module, session)
- Verify autouse fixtures aren't conflicting
