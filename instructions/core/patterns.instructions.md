---
description: "Common utility patterns for API responses, repository abstraction, and utilities. For Gang of Four design patterns (Factory, Observer, Strategy, Decorator, etc.), invoke the gang-of-four skill."
applyTo: "**"
---

# Common Patterns

> For GoF design patterns (creational, structural, behavioural), invoke the **gang-of-four** skill.
> Code examples use Python for illustration. For Python-specific conventions, invoke the python-conventions skill.

## API Response Format

Use a consistent envelope for all API responses:

```python
from typing import Generic, TypeVar
from dataclasses import dataclass, field

T = TypeVar("T")

@dataclass
class Meta:
    total: int
    page: int
    limit: int

@dataclass
class ApiResponse(Generic[T]):
    success: bool
    data: T | None = None
    error: str | None = None
    meta: Meta | None = None
```

## Repository Pattern

Abstract data access behind a typed interface:

```python
from abc import ABC, abstractmethod
from typing import Generic, TypeVar

T = TypeVar("T")

class Repository(ABC, Generic[T]):
    @abstractmethod
    async def find_all(self, filters: dict | None = None) -> list[T]: ...

    @abstractmethod
    async def find_by_id(self, id: str) -> T | None: ...

    @abstractmethod
    async def create(self, data: dict) -> T: ...

    @abstractmethod
    async def update(self, id: str, data: dict) -> T: ...

    @abstractmethod
    async def delete(self, id: str) -> None: ...
```

## Debounce / Throttle Utility

```python
import asyncio
from collections.abc import Callable
from typing import Any

def debounce(delay: float) -> Callable:
    """Return a decorator that debounces a coroutine by `delay` seconds."""
    def decorator(fn: Callable) -> Callable:
        task: asyncio.Task | None = None

        async def wrapper(*args: Any, **kwargs: Any) -> None:
            nonlocal task
            if task is not None:
                task.cancel()
            await asyncio.sleep(delay)
            await fn(*args, **kwargs)

        return wrapper
    return decorator
```

## Skeleton Projects

When implementing new functionality:

1. Search for battle-tested starter projects or libraries
2. Evaluate options across multiple dimensions:
   - Security posture
   - Extensibility
   - Relevance to the problem
   - Maintenance activity
3. Clone or install the best match as a foundation
4. Iterate within the proven structure
