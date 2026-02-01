---
name: python-conventions
description: Python coding standards including PEP 8, type hints, modern syntax (3.11+), ruff formatting, uv package management, and best practices. Use when writing, reviewing, or refactoring Python code, setting up Python projects, or managing dependencies.
license: Proprietary. See parent repository LICENSE
---

# Python Coding Conventions

## Quick Reference

- **Python version**: Use Python 3.11+ for modern features
- **Package manager**: Use Astral's `uv` for dependency and environment management
- **Formatting & Linting**: Use Astral's `ruff` (replaces flake8, isort, pylint, Black)
- **Type hints**: Modern built-in annotations (`list[str]`, `dict[str, int]`)
- **Type checking**: Use Astral's `ty` for type checking
- **Pre-commit**: User `prek` for pre-commit hooks
- **Documentation**: Google-style docstrings following PEP 257

## Core Instructions

- Write clear and concise comments for each function
- Ensure functions have descriptive names and include type hints
- Provide docstrings following PEP 257 conventions using Google format docstrings by default
- Use modern built-in type annotations (e.g., `list[str]`, `dict[str, int]`) for Python 3.9+ collections; only use `typing` module for complex types like `Union`, `Optional`, `Protocol`, `TypedDict`, `Generic`
- Break down complex functions into smaller, more manageable functions
- Use Python 3.11+ features and syntax including exception groups, fine-grained error locations, and TOML support
- Use f-strings for string formatting, but %s formatting for logging
- Prefer `pathlib.Path` over `os.path` for file system operations
- Use `Self` type for method chaining and builder patterns (from `typing_extensions` if needed)
- Leverage match/case statements for pattern matching where appropriate

## General Best Practices

- Always prioritise readability and clarity
- For algorithm-related code, include explanations of the approach used
- Write code with good maintainability practices, including comments on why certain design decisions were made
- Handle edge cases and write clear exception handling
- For libraries or external dependencies, mention their usage and purpose in comments
- Use consistent naming conventions and follow language-specific best practices
- Write concise, efficient, and idiomatic code that is also easily understandable
- Avoid using deprecated libraries, methods, or functions
- Follow the principle of least surprise - write code that behaves as expected
- Prioritise using built-in functions and libraries to simplify code
- Use NZ English spelling and grammar
- Align to best practices for Python development, including SOLID principles, DRY (Don't Repeat Yourself), and YAGNI (You Aren't Gonna Need It)

## Code Style and Formatting

- Follow the **PEP 8** style guide for Python
- Maintain proper indentation (use 4 spaces for each level of indentation)
- Ensure lines do not exceed 88 characters
- Place function and class docstrings immediately after the `def` or `class` keyword
- Use blank lines to separate functions, classes, and code blocks where appropriate
- **Use `ruff` for both formatting and linting** - it replaces flake8, isort, pylint, and Black

## Example of Proper Documentation

```python
from typing_extensions import Self


def calculate_area(radius: float) -> float:
    """Calculate the area of a circle given the radius.

    Args:
        radius: The radius of the circle.

    Returns:
        The area of the circle, calculated as π * radius².

    Raises:
        ValueError: If radius is negative.
    """
    if radius < 0:
        raise ValueError("Radius cannot be negative")

    import math
    return math.pi * radius ** 2


class CircleCalculator:
    """Calculator for circle-related operations."""

    def __init__(self, precision: int = 2) -> None:
        """Initialise the calculator with specified precision.

        Args:
            precision: Number of decimal places for results.
        """
        self.precision = precision

    def area(self, radius: float) -> Self:
        """Calculate area and return self for method chaining."""
        self._last_result = calculate_area(radius)
        return self

    def get_result(self) -> float:
        """Get the last calculated result."""
        return round(self._last_result, self.precision)
```

## Dependencies and Package Management

### Using uv (Recommended)

- Use `uv` for package management and virtual environment management
- Use `pyproject.toml` for project configuration and dependencies
- Pin dependency versions for reproducible builds
- Use dependency groups for development, testing, and optional dependencies

### Environment Detection and Tool Selection

**Always check project setup first**: Look for `.venv/`, `uv.lock`, or `pyproject.toml` with `[tool.uv]` section

**If uv-managed project detected**:

- Use `uv run` for executing Python scripts: `uv run python script.py`
- Use `uv run` for tests: `uv run pytest` or `uv run pytest -v`
- Use `uv add` for adding dependencies: `uv add package-name`
- Use `uv sync` to synchronise environment with lock file

**If standard venv detected** (no uv):

- Activate venv and use standard commands: `source .venv/bin/activate && pytest`

**For new projects**: Prefer `uv init` and `uv venv` for environment setup

### Example pyproject.toml

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "Project description"
requires-python = ">=3.11"
dependencies = [
    "httpx>=0.25.0",
    "pydantic>=2.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "ruff>=0.1.0",
    "mypy>=1.5.0",
]

[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]
```

## Performance Optimisation

- Use built-in data structures (`dict`, `set`, `deque`) for speed
- Profile with `cProfile`, `line_profiler`, or `Py-Spy`
- Use `multiprocessing` or `asyncio` for parallelism if appropriate
- Avoid GIL bottlenecks in CPU-bound code; use C extensions or subprocesses
- Use `lru_cache` for memoisation
- Leverage Python 3.11+ performance improvements like faster startup and optimised frame objects
- Use structural pattern matching (match/case) for complex conditional logic when appropriate

## Modern Python 3.11+ Features

### Exception Groups

```python
# Handle multiple exceptions at once
try:
    raise ExceptionGroup("multiple errors", [
        ValueError("invalid value"),
        TypeError("wrong type"),
    ])
except* ValueError as eg:
    # Handle all ValueErrors in the group
    for exc in eg.exceptions:
        print(f"ValueError: {exc}")
except* TypeError as eg:
    # Handle all TypeErrors in the group
    for exc in eg.exceptions:
        print(f"TypeError: {exc}")
```

### Pattern Matching

```python
def process_response(response: dict) -> str:
    match response:
        case {"status": 200, "data": data}:
            return f"Success: {data}"
        case {"status": 404}:
            return "Not found"
        case {"status": code} if code >= 500:
            return "Server error"
        case _:
            return "Unknown response"
```

### Type Hints Improvements

```python
# Use | for unions instead of Union
def process(value: str | int) -> bool:
    return isinstance(value, str)

# Use Self for return types
class Builder:
    def add_item(self, item: str) -> Self:
        return self
```

## Testing with pytest

Refer to the `python-testing-patterns` skill for comprehensive testing guidance.

## Alignment with Core Principles

- **SOLID Principles**: Single responsibility, dependency injection, interface segregation
- **DRY**: Avoid code duplication through functions, classes, and modules
- **YAGNI**: Build only what's needed now
- **KISS**: Keep solutions simple and straightforward
- **Test Pyramid**: 70% unit tests, 20% integration tests, 10% end-to-end tests

## References

- [PEP 8 - Style Guide for Python Code](https://peps.python.org/pep-0008/)
- [PEP 257 - Docstring Conventions](https://peps.python.org/pep-0257/)
- [Python 3.11 What's New](https://docs.python.org/3/whatsnew/3.11.html)
- [Ruff Documentation](https://docs.astral.sh/ruff/)
- [uv Documentation](https://docs.astral.sh/uv/)
- [ty Documentation](https://docs.astral.sh/ty/)
- [prek Documentation](https://prek.j178.dev/)
