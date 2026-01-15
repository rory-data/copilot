# Type-Safe Testing

Pattern 13 demonstrates using Python Protocols to ensure mocks are type-safe and implement required interfaces correctly.

## Pattern 13: Type Checking Tests

### Why Type Safety in Tests?

Type-safe mocks prevent issues like:

- Typos in mock attribute names (`mock_processor.non_existent_method()`)
- Missing required methods in mock implementation
- Mismatched method signatures

Using `Protocol` and `spec` parameter ensures mocks conform to expected interfaces at definition time.

### Using Protocol for Type-Safe Mocking

Define a Protocol that mocks must implement:

```python
# test_type_checking.py
import pytest
from typing import Protocol

class DataProcessor(Protocol):
    """Protocol for type-safe mocking."""

    def process(self, data: dict) -> dict:
        """Process data."""
        ...


class ServiceWithTyping:
    """Service that uses typed dependencies."""

    def __init__(self, processor: DataProcessor):
        self.processor = processor

    def handle_request(self, data: dict) -> dict:
        """Handle request with type checking."""
        return self.processor.process(data)


def test_with_protocol_mock():
    """Use Protocol for type-safe mocking."""
    from unittest.mock import Mock

    # Mock must implement Protocol
    mock_processor: DataProcessor = Mock(spec=DataProcessor)
    mock_processor.process.return_value = {"status": "ok"}

    service = ServiceWithTyping(mock_processor)
    result = service.handle_request({"input": "data"})

    assert result["status"] == "ok"
    mock_processor.process.assert_called_once()
```

### Protocol Ensures Implementation

When using `spec=DataProcessor`, the mock enforces that only protocol methods are available:

```python
def test_protocol_ensures_implementation():
    """Protocol ensures mock has required methods."""
    from unittest.mock import Mock

    # Using spec=DataProcessor ensures mock has correct interface
    mock: DataProcessor = Mock(spec=DataProcessor)
    mock.process.return_value = {"result": "data"}

    # This works - process() is in the Protocol
    result = mock.process({"input": "test"})
    assert result["result"] == "data"

    # This would raise AttributeError - non_existent_method() is not in Protocol
    # mock.non_existent_method()  # Would fail!

    # Calling non-existent methods raises error early during test setup
    # instead of failing silently in production code
```

### Complex Protocol with Multiple Methods

```python
from typing import Protocol, Optional

class DatabaseClient(Protocol):
    """Protocol for database operations."""

    def connect(self, url: str) -> bool:
        """Connect to database."""
        ...

    def query(self, sql: str) -> list:
        """Execute query."""
        ...

    def insert(self, table: str, data: dict) -> int:
        """Insert record."""
        ...

    def disconnect(self) -> None:
        """Close connection."""
        ...


class DataService:
    """Service that depends on database."""

    def __init__(self, db: DatabaseClient):
        self.db = db

    def get_user(self, user_id: int) -> Optional[dict]:
        """Get user from database."""
        results = self.db.query(f"SELECT * FROM users WHERE id = {user_id}")
        return results[0] if results else None


def test_database_service_with_protocol():
    """Test service with protocol-based mock."""
    from unittest.mock import Mock

    mock_db: DatabaseClient = Mock(spec=DatabaseClient)
    mock_db.query.return_value = [{"id": 1, "name": "Alice"}]

    service = DataService(mock_db)
    user = service.get_user(1)

    assert user["name"] == "Alice"
    mock_db.query.assert_called_once()
```

### Generic Protocol

For reusable mock patterns with type parameters:

```python
from typing import Protocol, TypeVar, Generic

T = TypeVar("T")


class Repository(Protocol, Generic[T]):
    """Protocol for data repository operations."""

    def get(self, id: int) -> T:
        """Get item by ID."""
        ...

    def save(self, item: T) -> int:
        """Save item."""
        ...

    def delete(self, id: int) -> bool:
        """Delete item by ID."""
        ...


class User:
    """User model."""

    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email


class UserService:
    """Service using repository."""

    def __init__(self, repo: Repository[User]):
        self.repo = repo

    def get_user(self, user_id: int) -> User:
        """Get user from repository."""
        return self.repo.get(user_id)


def test_generic_repository():
    """Test with generic repository mock."""
    from unittest.mock import Mock

    # Mock implements Repository[User]
    mock_repo: Repository[User] = Mock(spec=Repository)
    test_user = User("Alice", "alice@example.com")
    mock_repo.get.return_value = test_user

    service = UserService(mock_repo)
    user = service.get_user(1)

    assert user.name == "Alice"
    mock_repo.get.assert_called_once_with(1)
```

## Benefits of Protocol-Based Testing

1. **Catches Typos Early**: Trying to access non-existent methods raises `AttributeError` immediately
2. **Self-Documenting**: Protocol clearly shows what methods are required
3. **Refactoring Safe**: Changes to Protocol signature are caught in all test mocks
4. **IDE Support**: Better autocomplete and type hints in IDEs
5. **Consistent Contracts**: Ensures all implementations follow the same interface

## When to Use Protocols vs Traditional Mocks

| Use Protocol When                          | Use Traditional Mock When         |
| ------------------------------------------ | --------------------------------- |
| Interface is stable and won't change often | Rapid prototyping and exploration |
| Mocking complex objects with many methods  | Simple, one-off mocks             |
| Testing against external dependencies      | Testing internal implementation   |
| Enforcing contract compliance              | Testing specific call patterns    |

Protocols are best for production-critical code where type safety prevents bugs.
