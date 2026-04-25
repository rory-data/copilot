# Creational Patterns

Five patterns that control how objects are brought into existence.

---

## Factory Method

**Intent:** Define an interface for creating an object, but let subclasses decide which class
to instantiate. Decouples the code that uses an object from the code that creates it.

**Structure:** A creator class declares a factory method returning a product interface.
Concrete creators override the method to return concrete products.

**When to use:**
- You don't know in advance which class to instantiate (depends on configuration or context)
- You want subclasses to control what gets created
- You need to encapsulate object creation so it can be replaced or extended

**When to avoid:**
- The concrete type is always the same — just use a constructor
- You only need one concrete type now and cannot see a second coming

**Python example:**

```python
from abc import ABC, abstractmethod


class Notifier(ABC):
    @abstractmethod
    def send(self, message: str) -> None: ...

    # Factory method
    @classmethod
    @abstractmethod
    def create(cls) -> "Notifier": ...


class EmailNotifier(Notifier):
    def send(self, message: str) -> None:
        print(f"Email: {message}")

    @classmethod
    def create(cls) -> "EmailNotifier":
        return cls()


class SMSNotifier(Notifier):
    def send(self, message: str) -> None:
        print(f"SMS: {message}")

    @classmethod
    def create(cls) -> "SMSNotifier":
        return cls()


def notify_user(notifier_class: type[Notifier], message: str) -> None:
    notifier = notifier_class.create()
    notifier.send(message)


notify_user(EmailNotifier, "Your order has shipped.")
notify_user(SMSNotifier, "Your order has shipped.")
```

**Pythonic note:** A plain function that returns different objects based on a parameter (a
"factory function") is often sufficient and clearer than a factory class hierarchy.

---

## Abstract Factory

**Intent:** Provide an interface for creating families of related objects without specifying
their concrete classes. Ensures that related objects are always used together.

**When to use:**
- You need to create sets of objects that must be compatible (e.g., a UI theme: buttons +
  dialogs + inputs all matching)
- You want to swap entire families of objects at runtime or configuration time

**When to avoid:**
- You only have one product family — Factory Method is simpler
- Adding a new product type requires changing the factory interface and all concrete factories

**Python example:**

```python
from abc import ABC, abstractmethod


class Button(ABC):
    @abstractmethod
    def render(self) -> str: ...


class Dialog(ABC):
    @abstractmethod
    def render(self) -> str: ...


class DarkButton(Button):
    def render(self) -> str:
        return "<button class='dark'>Click</button>"


class DarkDialog(Dialog):
    def render(self) -> str:
        return "<dialog class='dark'>Content</dialog>"


class LightButton(Button):
    def render(self) -> str:
        return "<button class='light'>Click</button>"


class LightDialog(Dialog):
    def render(self) -> str:
        return "<dialog class='light'>Content</dialog>"


class UIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button: ...

    @abstractmethod
    def create_dialog(self) -> Dialog: ...


class DarkThemeFactory(UIFactory):
    def create_button(self) -> DarkButton:
        return DarkButton()

    def create_dialog(self) -> DarkDialog:
        return DarkDialog()


class LightThemeFactory(UIFactory):
    def create_button(self) -> LightButton:
        return LightButton()

    def create_dialog(self) -> LightDialog:
        return LightDialog()


def render_page(factory: UIFactory) -> None:
    print(factory.create_button().render())
    print(factory.create_dialog().render())
```

---

## Builder

**Intent:** Separate the construction of a complex object from its representation, so the
same construction process can produce different representations.

**When to use:**
- An object requires many steps or optional parameters to construct correctly
- You want to construct different representations using the same process
- The telescoping constructor anti-pattern has appeared (many constructor overloads)

**When to avoid:**
- The object is simple — a dataclass or named arguments suffice
- There is only one representation

**Python example:**

```python
from dataclasses import dataclass, field


@dataclass
class QueryBuilder:
    _table: str = ""
    _conditions: list[str] = field(default_factory=list)
    _columns: list[str] = field(default_factory=list)
    _limit: int | None = None

    def from_table(self, table: str) -> "QueryBuilder":
        return QueryBuilder(table, list(self._conditions), list(self._columns), self._limit)

    def select(self, *columns: str) -> "QueryBuilder":
        return QueryBuilder(self._table, list(self._conditions), list(columns), self._limit)

    def where(self, condition: str) -> "QueryBuilder":
        return QueryBuilder(self._table, self._conditions + [condition], list(self._columns), self._limit)

    def limit(self, n: int) -> "QueryBuilder":
        return QueryBuilder(self._table, list(self._conditions), list(self._columns), n)

    def build(self) -> str:
        cols = ", ".join(self._columns) if self._columns else "*"
        sql = f"SELECT {cols} FROM {self._table}"
        if self._conditions:
            sql += " WHERE " + " AND ".join(self._conditions)
        if self._limit is not None:
            sql += f" LIMIT {self._limit}"
        return sql


query = (
    QueryBuilder()
    .from_table("users")
    .select("id", "email")
    .where("active = true")
    .limit(10)
    .build()
)
# SELECT id, email FROM users WHERE active = true LIMIT 10
```

**Pythonic note:** For simple cases, Python's keyword arguments and `dataclasses` with
`field(default=...)` often eliminate the need for a Builder entirely.

---

## Prototype

**Intent:** Create new objects by cloning an existing instance rather than constructing
from scratch. Useful when construction is expensive or when the initial state is complex.

**When to use:**
- Object creation is costly (e.g., involves database lookup or heavy computation)
- You need to create objects with state copied from an existing instance
- The class hierarchy is unknown or deeply nested

**When to avoid:**
- Construction is cheap — just call the constructor
- The object has non-copyable resources (open file handles, sockets)

**Python example:**

```python
import copy
from dataclasses import dataclass, field


@dataclass
class DocumentTemplate:
    title: str
    body: str
    tags: list[str] = field(default_factory=list)
    metadata: dict[str, str] = field(default_factory=dict)

    def clone(self) -> "DocumentTemplate":
        # Deep copy ensures nested mutable objects are independent
        return copy.deepcopy(self)


base_template = DocumentTemplate(
    title="Report Template",
    body="## Summary\n\n## Details\n",
    tags=["report", "draft"],
    metadata={"author": "system"},
)

monthly_report = base_template.clone()
monthly_report.title = "Monthly Report — April"
monthly_report.metadata["author"] = "finance-team"

# base_template is unchanged
```

**Pythonic note:** Python's `copy.deepcopy` is the standard Prototype mechanism. You rarely
need a dedicated `clone()` method unless you want to constrain what is copied.

---

## Singleton

**Intent:** Ensure a class has only one instance and provide a global access point to it.

**When to use:**
- Exactly one shared resource is needed: a connection pool, a configuration store, a logger
- Construction is expensive and results must be shared (not duplicated)

**When to avoid — and this is important:**
- Global state almost always makes code harder to test and reason about
- Consider whether a module-level variable achieves the same goal more simply
- Dependency injection is almost always preferable: pass the shared instance explicitly

**Python example (module-level — usually sufficient):**

```python
# config.py — a module IS a singleton; import it anywhere
import os

DATABASE_URL: str = os.environ["DATABASE_URL"]
DEBUG: bool = os.environ.get("DEBUG", "false").lower() == "true"
```

**Python example (class-based, thread-safe):**

```python
import threading
from typing import ClassVar


class ConnectionPool:
    _instance: ClassVar["ConnectionPool | None"] = None
    _lock: ClassVar[threading.Lock] = threading.Lock()

    def __new__(cls) -> "ConnectionPool":
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._initialise()
        return cls._instance

    def _initialise(self) -> None:
        self._connections: list = []

    def acquire(self) -> object:
        ...
```

**Warning:** Singleton is the most over-applied creational pattern. Before using it, ask:
could I just pass this object as a constructor argument?
