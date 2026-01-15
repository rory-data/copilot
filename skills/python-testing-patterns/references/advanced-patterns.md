# Advanced Testing Patterns

This reference covers Patterns 6-12: specialised testing techniques for logging, async code, file operations, custom fixtures, property-based testing, and data schema validation.

## Pattern 6: Testing with Logging

### Standard Library Logging

```python
# test_logging_standard.py
import logging
import pytest

class UserService:
    """Service with logging using standard library."""

    def __init__(self, logger: logging.Logger):
        self.logger = logger

    def create_user(self, name: str) -> dict:
        """Create user with logging."""
        self.logger.info(f"Creating user: {name}")
        user = {"id": 1, "name": name}
        self.logger.info(f"User created: {user}")
        return user

    def delete_user(self, user_id: int) -> None:
        """Delete user."""
        self.logger.warning(f"Deleting user: {user_id}")


def test_logging_with_caplog(caplog):
    """Test logging with standard logging library using caplog."""
    logger = logging.getLogger(__name__)
    service = UserService(logger)

    with caplog.at_level(logging.INFO):
        user = service.create_user("John Doe")

    assert user["name"] == "John Doe"
    assert "Creating user: John Doe" in caplog.text
    assert "User created" in caplog.text
    assert caplog.records[0].levelname == "INFO"


def test_logging_warning_level(caplog):
    """Test capturing specific log levels."""
    logger = logging.getLogger(__name__)
    service = UserService(logger)

    with caplog.at_level(logging.WARNING):
        service.delete_user(123)

    assert any(record.levelname == "WARNING" for record in caplog.records)
    assert "Deleting user: 123" in caplog.text
```

### loguru Logging

Use this pattern when your project uses loguru for structured logging:

```python
# test_logging_loguru.py
import pytest
from loguru import logger

class UserServiceLoguru:
    """Service with logging using loguru."""

    def create_user(self, name: str) -> dict:
        """Create user with loguru and structured context."""
        logger.info("Creating user: {name}", name=name)
        user = {"id": 1, "name": name}
        # Use bind() for structured logging context
        logger.bind(user=user).info("User created")
        return user

    def delete_user(self, user_id: int) -> None:
        """Delete user."""
        logger.warning("Deleting user: {user_id}", user_id=user_id)


def test_loguru_with_fixture(caplog):
    """Test loguru using a fixture that integrates with pytest caplog."""
    # Add loguru sink to caplog handler
    handler_id = logger.add(caplog.handler, format="{message}", level="INFO")

    try:
        service = UserServiceLoguru()
        service.create_user("Jane Doe")
        assert "Creating user: Jane Doe" in caplog.text
    finally:
        logger.remove(handler_id)


def test_loguru_structured_logging():
    """Test structured logging by capturing records directly."""
    records = []

    # Custom sink to capture full record objects
    handler_id = logger.add(lambda m: records.append(m.record), level="INFO")

    try:
        service = UserServiceLoguru()
        service.create_user("Bob Smith")
        assert len(records) > 1  # Should have two log entries
        # Check structured extra context from bind()
        assert records[1]["extra"]["user"]["name"] == "Bob Smith"
    finally:
        logger.remove(handler_id)
```

## Pattern 7: Testing Async Code

```python
# test_async.py
import pytest
import asyncio

async def fetch_data(url: str) -> dict:
    """Fetch data asynchronously."""
    await asyncio.sleep(0.1)
    return {"url": url, "data": "result"}


@pytest.mark.asyncio
async def test_fetch_data():
    """Test async function."""
    result = await fetch_data("https://api.example.com")
    assert result["url"] == "https://api.example.com"
    assert "data" in result


@pytest.mark.asyncio
async def test_concurrent_fetches():
    """Test concurrent async operations."""
    urls = ["url1", "url2", "url3"]
    tasks = [fetch_data(url) for url in urls]
    results = await asyncio.gather(*tasks)

    assert len(results) == 3
    assert all("data" in r for r in results)


@pytest.fixture
async def async_client():
    """Async fixture."""
    client = {"connected": True}
    yield client
    client["connected"] = False


@pytest.mark.asyncio
async def test_with_async_fixture(async_client):
    """Test using async fixture."""
    assert async_client["connected"] is True
```

## Pattern 8: Monkeypatch for Testing

Use monkeypatch to temporarily modify environment variables, object attributes, and imported modules:

```python
# test_environment.py
import os
import pytest

def get_database_url() -> str:
    """Get database URL from environment."""
    return os.environ.get("DATABASE_URL", "sqlite:///:memory:")


def test_database_url_default():
    """Test default database URL."""
    # Will use actual environment variable if set
    url = get_database_url()
    assert url


def test_database_url_custom(monkeypatch):
    """Test custom database URL with monkeypatch."""
    monkeypatch.setenv("DATABASE_URL", "postgresql://localhost/test")
    assert get_database_url() == "postgresql://localhost/test"


def test_database_url_not_set(monkeypatch):
    """Test when env var is not set."""
    monkeypatch.delenv("DATABASE_URL", raising=False)
    assert get_database_url() == "sqlite:///:memory:"


class Config:
    """Configuration class."""

    def __init__(self):
        self.api_key = "production-key"

    def get_api_key(self):
        return self.api_key


def test_monkeypatch_attribute(monkeypatch):
    """Test monkeypatching object attributes."""
    config = Config()
    monkeypatch.setattr(config, "api_key", "test-key")
    assert config.get_api_key() == "test-key"
```

## Pattern 9: Temporary Files and Directories

```python
# test_file_operations.py
import pytest
from pathlib import Path

def save_data(filepath: Path, data: str):
    """Save data to file."""
    filepath.write_text(data)


def load_data(filepath: Path) -> str:
    """Load data from file."""
    return filepath.read_text()


def test_file_operations(tmp_path):
    """Test file operations with temporary directory."""
    # tmp_path is a pathlib.Path object
    test_file = tmp_path / "test_data.txt"

    # Save data
    save_data(test_file, "Hello, World!")

    # Verify file exists
    assert test_file.exists()

    # Load and verify data
    data = load_data(test_file)
    assert data == "Hello, World!"


def test_multiple_files(tmp_path):
    """Test with multiple temporary files."""
    files = {
        "file1.txt": "Content 1",
        "file2.txt": "Content 2",
        "file3.txt": "Content 3"
    }

    for filename, content in files.items():
        filepath = tmp_path / filename
        save_data(filepath, content)

    # Verify all files created
    assert len(list(tmp_path.iterdir())) == 3

    # Verify contents
    for filename, expected_content in files.items():
        filepath = tmp_path / filename
        assert load_data(filepath) == expected_content
```

## Pattern 10: Custom Fixtures and Conftest

Use `conftest.py` to share fixtures across multiple test modules:

```python
# conftest.py
"""Shared fixtures for all tests."""
import pytest

@pytest.fixture(scope="session")
def database_url():
    """Provide database URL for all tests."""
    return "postgresql://localhost/test_db"


@pytest.fixture(autouse=True)
def reset_database(database_url):
    """Auto-use fixture that runs before each test."""
    # Setup: Clear database
    print(f"Clearing database: {database_url}")
    yield
    # Teardown: Clean up
    print("Test completed")


@pytest.fixture
def sample_user():
    """Provide sample user data."""
    return {
        "id": 1,
        "name": "Test User",
        "email": "test@example.com"
    }


@pytest.fixture
def sample_users():
    """Provide list of sample users."""
    return [
        {"id": 1, "name": "User 1"},
        {"id": 2, "name": "User 2"},
        {"id": 3, "name": "User 3"},
    ]


# Parametrized fixture
@pytest.fixture(params=["sqlite", "postgresql", "mysql"])
def db_backend(request):
    """Fixture that runs tests with different database backends."""
    return request.param


def test_with_db_backend(db_backend):
    """This test will run 3 times with different backends."""
    print(f"Testing with {db_backend}")
    assert db_backend in ["sqlite", "postgresql", "mysql"]
```

## Pattern 11: Property-Based Testing

Use hypothesis to automatically generate test cases and find edge cases:

```python
# test_properties.py
from hypothesis import given, strategies as st
import pytest

def reverse_string(s: str) -> str:
    """Reverse a string."""
    return s[::-1]


@given(st.text())
def test_reverse_twice_is_original(s):
    """Property: reversing twice returns original."""
    assert reverse_string(reverse_string(s)) == s


@given(st.text())
def test_reverse_length(s):
    """Property: reversed string has same length."""
    assert len(reverse_string(s)) == len(s)


@given(st.integers(), st.integers())
def test_addition_commutative(a, b):
    """Property: addition is commutative."""
    assert a + b == b + a


@given(st.lists(st.integers()))
def test_sorted_list_properties(lst):
    """Property: sorted list is ordered."""
    sorted_lst = sorted(lst)

    # Same length
    assert len(sorted_lst) == len(lst)

    # All elements present
    assert set(sorted_lst) == set(lst)

    # Is ordered
    for i in range(len(sorted_lst) - 1):
        assert sorted_lst[i] <= sorted_lst[i + 1]
```

Property-based testing is valuable for finding edge cases automatically. Use hypothesis strategies for common types: `st.text()`, `st.integers()`, `st.lists()`, `st.dictionaries()`, etc.

## Pattern 12: Testing Data Schemas

Modern data engineering relies on robust schema validation. Use tools like `Pydantic` or `Pandera` to ensure data complies with expected structures and types.

```python
# test_data_validation.py
import pytest
from pydantic import BaseModel, EmailStr, Field, ValidationError

class UserSchema(BaseModel):
    """Schema for user data validation."""
    id: int
    name: str = Field(min_length=2)
    email: EmailStr
    age: int = Field(gt=0, lt=150)


def validate_user_data(data: dict) -> UserSchema:
    """Validate raw dictionary against schema."""
    return UserSchema(**data)


def test_valid_user_data():
    """Test validation with correct data."""
    data = {
        "id": 1,
        "name": "Jane",
        "email": "jane@example.com",
        "age": 30
    }
    user = validate_user_data(data)
    assert user.id == 1
    assert user.name == "Jane"


def test_invalid_age_raises_error():
    """Test validation fails for invalid age."""
    data = {
        "id": 1,
        "name": "Jane",
        "email": "jane@example.com",
        "age": -5  # Invalid age
    }
    with pytest.raises(ValidationError) as exc_info:
        validate_user_data(data)

    assert "age" in str(exc_info.value)


@pytest.mark.parametrize("invalid_data", [
    {"id": "not-an-int", "name": "J", "email": "bad-email", "age": 200},
    {"id": 1, "name": "", "email": "valid@email.com", "age": 25},
    {"id": 1, "name": "Jane", "email": "jane@example.com"},  # Missing 'age'
])
def test_schema_constraints(invalid_data):
    """Test various schema constraint violations."""
    with pytest.raises(ValidationError):
        validate_user_data(invalid_data)
```
