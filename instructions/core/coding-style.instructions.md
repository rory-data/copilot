---
description: "Coding style rules covering immutability, file organisation, error handling, and input validation"
applyTo: "**"
---

# Coding Style

> Code examples use Python for illustration. For Python-specific conventions, invoke the python-conventions skill.

## Immutability (CRITICAL)

ALWAYS create new objects, NEVER mutate in place:

```python
# WRONG: Mutation
def update_user(user: dict, name: str) -> dict:
    user["name"] = name  # MUTATION!
    return user

# CORRECT: Immutability
def update_user(user: dict, name: str) -> dict:
    return {**user, "name": name}
```

For dataclasses, use `dataclasses.replace`:

```python
from dataclasses import dataclass, replace

@dataclass(frozen=True)
class User:
    id: int
    name: str

def update_user(user: User, name: str) -> User:
    return replace(user, name=name)
```

## File Organisation

MANY SMALL FILES > FEW LARGE FILES:

- High cohesion, low coupling
- 200–400 lines typical, 800 lines maximum
- Extract utilities from large modules
- Organise by feature/domain, not by type

## Error Handling

ALWAYS handle errors comprehensively:

```python
import logging

logger = logging.getLogger(__name__)

async def risky_operation() -> dict:
    ...

async def process() -> dict:
    try:
        result = await risky_operation()
        return result
    except ValueError as exc:
        logger.error("Operation failed: %s", exc)
        raise RuntimeError("Detailed user-friendly message") from exc
```

## Input Validation

ALWAYS validate user input. Use Pydantic for structured data:

```python
from pydantic import BaseModel, EmailStr, field_validator

class UserInput(BaseModel):
    email: EmailStr
    age: int

    @field_validator("age")
    @classmethod
    def age_must_be_valid(cls, v: int) -> int:
        if not 0 <= v <= 150:
            raise ValueError("Age must be between 0 and 150")
        return v

validated = UserInput.model_validate(raw_input)
```

## Code Quality Checklist

Before marking work complete:

- [ ] Code is readable and well-named
- [ ] Functions are small (<50 lines)
- [ ] Files are focused (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Proper error handling
- [ ] No `print` statements left in production code (use `logging`)
- [ ] No hardcoded values
- [ ] Immutable patterns used; no in-place mutation
