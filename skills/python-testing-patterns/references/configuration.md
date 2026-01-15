# Configuration and Dependencies

This reference covers coverage reporting, database testing, CI/CD integration, pytest configuration, and test dependencies.

## Coverage Reporting

### Running Coverage

```bash
# Add coverage dependency
uv add --dev pytest-cov

# Run tests with coverage
uv run pytest --cov=myapp tests/

# Generate HTML report
uv run pytest --cov=myapp --cov-report=html tests/

# Fail if coverage below threshold (recommend >80% for meaningful coverage)
uv run pytest --cov=myapp --cov-fail-under=80 tests/

# Show missing lines
uv run pytest --cov=myapp --cov-report=term-missing tests/
```

### Coverage Quality

Aim for >80% coverage with meaningful tests. Quality matters more than percentages—focus on testing behaviour, edge cases, and error conditions rather than just increasing coverage numbers. One assertion that tests important logic is better than ten trivial assertions.

## Testing Database Code

```python
# test_database_models.py
import pytest
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session

Base = declarative_base()


class User(Base):
    """User model."""
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    email = Column(String(100), unique=True)


@pytest.fixture(scope="function")
def db_session() -> Session:
    """Create in-memory database for testing."""
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)

    SessionLocal = sessionmaker(bind=engine)
    session = SessionLocal()

    yield session

    session.close()


def test_create_user(db_session):
    """Test creating a user."""
    user = User(name="Test User", email="test@example.com")
    db_session.add(user)
    db_session.commit()

    assert user.id is not None
    assert user.name == "Test User"


def test_query_user(db_session):
    """Test querying users."""
    user1 = User(name="User 1", email="user1@example.com")
    user2 = User(name="User 2", email="user2@example.com")

    db_session.add_all([user1, user2])
    db_session.commit()

    users = db_session.query(User).all()
    assert len(users) == 2


def test_unique_email_constraint(db_session):
    """Test unique email constraint."""
    from sqlalchemy.exc import IntegrityError

    user1 = User(name="User 1", email="same@example.com")
    user2 = User(name="User 2", email="same@example.com")

    db_session.add(user1)
    db_session.commit()

    db_session.add(user2)

    with pytest.raises(IntegrityError):
        db_session.commit()
```

## CI/CD Integration

Example GitHub Actions workflow for continuous testing:

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: ["3.9", "3.10", "3.11", "3.12"]

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}

      - name: Set up uv
        uses: astral-sh/setup-uv@v3

      - name: Install dependencies
        run: uv sync --all-extras --dev

      - name: Run tests
        run: uv run pytest --cov=myapp --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
```

## Configuration Files

**Preferred approach**: Use `pyproject.toml` as the single source of truth for project configuration.

### pyproject.toml (Recommended)

```toml
# pyproject.toml - Recommended configuration approach
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = [
    "-v",
    "--strict-markers",
    "--tb=short",
    "--cov=myapp",
    "--cov-report=term-missing",
    "--cov-fail-under=80",
]

markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "integration: marks integration tests",
    "unit: marks unit tests",
    "e2e: marks end-to-end tests",
]

[tool.coverage.run]
source = ["myapp"]
omit = [
    "*/tests/*",
    "*/migrations/*",
    "*/__main__.py",
]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
    "if __name__ == .__main__.:",
]
precision = 2
```

### pytest.ini (Legacy)

If using pytest.ini instead of pyproject.toml:

```ini
# pytest.ini - Legacy approach (use pyproject.toml instead)
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts =
    -v
    --strict-markers
    --tb=short
    --cov=myapp
    --cov-report=term-missing
markers =
    slow: marks tests as slow
    integration: marks integration tests
    unit: marks unit tests
    e2e: marks end-to-end tests
```

## Testing Dependencies

Essential pytest plugins and test dependencies:

### Installation

```bash
# Install with uv (recommended)
uv add --dev pytest pytest-cov pytest-mock pytest-asyncio hypothesis
```

### pyproject.toml Dependencies

```toml
# pyproject.toml - Dependencies section
[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov>=4.0",      # Coverage reporting
    "pytest-mock>=3.10",    # Enhanced mocking with spec support
    "pytest-asyncio>=0.21", # Async test support
    "hypothesis>=6.70",     # Property-based testing
]

test = [
    "pytest>=7.0",
    "pytest-cov>=4.0",
]
```

### Key Plugins

| Plugin             | Purpose                                                 | Version |
| ------------------ | ------------------------------------------------------- | ------- |
| **pytest**         | Testing framework                                       | >=7.0   |
| **pytest-cov**     | Coverage measurement and reporting                      | >=4.0   |
| **pytest-mock**    | Enhanced mocking with `mocker` fixture and spec support | >=3.10  |
| **pytest-asyncio** | Async/await test support with markers                   | >=0.21  |
| **hypothesis**     | Property-based testing for edge case discovery          | >=6.70  |

### Optional Enhancements

For additional functionality:

```toml
[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov>=4.0",
    "pytest-mock>=3.10",
    "pytest-asyncio>=0.21",
    "hypothesis>=6.70",
    "pytest-xdist>=3.0",        # Parallel test execution
    "pytest-timeout>=2.1",      # Timeout handling
    "pytest-randomly>=3.10",    # Randomise test order
    "pytest-benchmark>=4.0",    # Performance benchmarking
]
```

## Test Patterns by Framework

### FastAPI

```python
from fastapi.testclient import TestClient
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}


def test_get_user():
    """Test FastAPI endpoint."""
    client = TestClient(app)
    response = client.get("/users/1")
    assert response.status_code == 200
    assert response.json()["user_id"] == 1
```

### Django

```python
# tests.py
from django.test import TestCase
from django.contrib.auth.models import User

class UserTestCase(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username="testuser",
            password="testpass"
        )

    def test_user_created(self):
        """Test user creation."""
        self.assertEqual(self.user.username, "testuser")
```

### Pydantic

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    email: str


def test_pydantic_validation():
    """Test pydantic validation."""
    user = User(name="Alice", email="alice@example.com")
    assert user.name == "Alice"

    # Validation fails with invalid data
    with pytest.raises(ValueError):
        User(name="Bob")  # Missing email
```

## Best Practices

1. **Minimal Configuration**: Only specify what's necessary
2. **Consistent Naming**: Follow pytest conventions (test*\*.py, test*\*() functions)
3. **Marker Discipline**: Use markers consistently across projects
4. **Coverage Thresholds**: Set meaningful thresholds (80%+ for important code)
5. **CI/CD Integration**: Always run tests on every push
6. **Dependency Pinning**: Pin versions in CI for reproducibility

## Common Configuration Issues

### Tests not discovered

- Check testpaths matches your directory structure
- Verify files start with `test_` prefix
- Ensure test functions start with `test_`
- Run `pytest --collect-only` to see discovered tests

### Coverage not working

- Install pytest-cov plugin
- Use `--cov=module_name` (not full file path)
- Check that `__init__.py` files exist in packages
- Use `--cov-report=term-missing` to see uncovered lines

### Import errors in tests

- Use absolute imports: `from myapp.module import function`
- Add project root to PYTHONPATH
- Ensure package has `__init__.py` files
- Check that dependencies are installed
