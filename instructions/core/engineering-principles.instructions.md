---
description: "Shared engineering fundamentals, design patterns, and quality principles"
applyTo: "**"
---

# Core Engineering Principles

## Single Responsibility

Each class or function has one reason to change. When a class does two things, split it:

```python
# WRONG: Mixed responsibilities
class User:
    def save_to_db(self) -> None: ...
    def send_welcome_email(self) -> None: ...

# CORRECT: Separate concerns
class UserRepository:
    def save(self, user: "User") -> None: ...

class UserNotifier:
    def send_welcome_email(self, user: "User") -> None: ...
```

## Don't Repeat Yourself (DRY)

Extract shared logic rather than duplicating it. Every piece of knowledge should have a single, authoritative representation:

```python
# WRONG: Duplicated validation
def process_payment(amount: float) -> None:
    if amount <= 0:
        raise ValueError("Amount must be positive")
    ...

def process_refund(amount: float) -> None:
    if amount <= 0:
        raise ValueError("Amount must be positive")
    ...

# CORRECT: Extract shared logic
def _validate_positive_amount(amount: float) -> None:
    if amount <= 0:
        raise ValueError("Amount must be positive")

def process_payment(amount: float) -> None:
    _validate_positive_amount(amount)
    ...
```

## Dependency Inversion

Depend on abstractions, not concretions. Inject dependencies rather than instantiating them internally — this makes code testable and adaptable:

```python
# WRONG: Hard-coded dependency
class OrderService:
    def __init__(self) -> None:
        self.db = PostgresDatabase()  # Impossible to test or swap

# CORRECT: Injected abstraction
class OrderService:
    def __init__(self, db: Database) -> None:
        self.db = db
```

## Error Handling

Never swallow exceptions silently. Handle errors explicitly with context:

```python
# WRONG: Silent failure
def load_config(path: str) -> dict:
    try:
        return json.load(open(path))
    except Exception:
        return {}  # Hides the problem

# CORRECT: Explicit handling with context
def load_config(path: str) -> dict:
    try:
        with open(path) as f:
            return json.load(f)
    except FileNotFoundError:
        raise ConfigError(f"Config file not found: {path}")
    except json.JSONDecodeError as exc:
        raise ConfigError(f"Invalid JSON in config {path}: {exc}") from exc
```

## YAGNI & KISS

Build only what is explicitly needed — do not anticipate requirements. Prefer the simplest correct solution. Every added abstraction must earn its place.

## Function Size

Keep functions small and focused: 50 lines maximum, one level of abstraction per function, no nesting deeper than 3 levels.

## Testing

Test pyramid: **70% unit, 20% integration, 10% end-to-end.** Target 80% coverage with meaningful assertions, not padding.

Structure tests with AAA — Arrange, Act, Assert:

```python
def test_discount_applies_to_premium_orders():
    # Arrange
    order = Order(total=200.0, customer_tier="premium")

    # Act
    discount = calculate_discount(order)

    # Assert
    assert discount == 20.0
```

Test behaviour, not implementation. Test boundary conditions and error paths.

## Code Quality Checklist

Before committing:

- [ ] Functions are ≤50 lines and do one thing
- [ ] Names are descriptive — no abbreviations, no single-letter variables outside loops
- [ ] No commented-out code; no hardcoded values
- [ ] Nesting does not exceed 3 levels
- [ ] All errors are handled explicitly with context
- [ ] New code has tests; existing tests still pass
