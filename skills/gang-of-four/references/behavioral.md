# Behavioral Patterns

Eleven patterns concerned with algorithms and the assignment of responsibilities between objects.

---

## Chain of Responsibility

**Intent:** Pass a request along a chain of handlers. Each handler decides either to process
the request or pass it to the next handler in the chain.

**When to use:**
- More than one object may handle a request and the handler is unknown a priori
- You want to issue a request to one of several objects without specifying the receiver explicitly
- Middleware pipelines, event processing, approval workflows

**When to avoid:**
- There is always exactly one handler — just call it directly

**Python example:**

```python
from abc import ABC, abstractmethod


class Handler(ABC):
    def __init__(self) -> None:
        self._next: "Handler | None" = None

    def set_next(self, handler: "Handler") -> "Handler":
        self._next = handler
        return handler

    def handle(self, request: int) -> str | None:
        if self._next:
            return self._next.handle(request)
        return None


class LowPriorityHandler(Handler):
    def handle(self, request: int) -> str | None:
        if request < 10:
            return f"LowPriority handled {request}"
        return super().handle(request)


class MediumPriorityHandler(Handler):
    def handle(self, request: int) -> str | None:
        if request < 100:
            return f"MediumPriority handled {request}"
        return super().handle(request)


class HighPriorityHandler(Handler):
    def handle(self, request: int) -> str | None:
        return f"HighPriority handled {request}"


low = LowPriorityHandler()
medium = MediumPriorityHandler()
high = HighPriorityHandler()
low.set_next(medium).set_next(high)

print(low.handle(5))    # LowPriority
print(low.handle(50))   # MediumPriority
print(low.handle(500))  # HighPriority
```

---

## Command

**Intent:** Encapsulate a request as an object, so you can parameterise clients with
different requests, queue or log them, and support undoable operations.

**When to use:**
- You need undo/redo
- You need to queue, schedule, or log operations
- You want to decouple the object that invokes an operation from the one that performs it

**When to avoid:**
- You just want to call a function — a callable is simpler than a Command class

**Python example:**

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass


class Command(ABC):
    @abstractmethod
    def execute(self) -> None: ...

    @abstractmethod
    def undo(self) -> None: ...


@dataclass
class TextEditor:
    text: str = ""


@dataclass
class InsertCommand(Command):
    editor: TextEditor
    position: int
    text: str

    def execute(self) -> None:
        self.editor.text = (
            self.editor.text[: self.position]
            + self.text
            + self.editor.text[self.position :]
        )

    def undo(self) -> None:
        self.editor.text = (
            self.editor.text[: self.position]
            + self.editor.text[self.position + len(self.text) :]
        )


class CommandHistory:
    def __init__(self) -> None:
        self._history: list[Command] = []

    def execute(self, command: Command) -> None:
        command.execute()
        self._history.append(command)

    def undo(self) -> None:
        if self._history:
            self._history.pop().undo()
```

---

## Interpreter

**Intent:** Given a language, define a representation for its grammar along with an
interpreter that uses that representation to interpret sentences in the language.

**When to use:**
- You have a simple grammar that can be represented as an abstract syntax tree
- The grammar is small, stable, and efficiency is not a priority

**When to avoid (strong advice):**
- For any complex language, use a proper parser library (e.g., `lark`, `pyparsing`)
- This pattern is rarely the right answer outside of toy languages or DSLs
- Maintenance becomes painful as the grammar grows

**Pythonic note:** Python's `ast` module, `lark-parser`, or `pyparsing` handle real
grammar requirements far more robustly than hand-rolled Interpreter classes.

---

## Iterator

**Intent:** Provide a way to access the elements of an aggregate object sequentially
without exposing its underlying representation.

**When to use:**
- You need to traverse a collection without coupling to its implementation
- You want uniform traversal across different collection types

**Pythonic reality:** Python's iterator protocol (`__iter__` / `__next__`), generators,
and `for` loops make this pattern built-in. You rarely need an explicit Iterator class.

**Python example:**

```python
class NumberRange:
    """Custom iterator using Python's iterator protocol."""
    def __init__(self, start: int, stop: int, step: int = 1) -> None:
        self._start = start
        self._stop = stop
        self._step = step

    def __iter__(self):
        current = self._start
        while current < self._stop:
            yield current
            current += self._step


for n in NumberRange(0, 10, 2):
    print(n)  # 0, 2, 4, 6, 8
```

---

## Mediator

**Intent:** Define an object that encapsulates how a set of objects interact. Promotes
loose coupling by preventing objects from referring to each other explicitly.

**When to use:**
- Many objects communicate in complex ways, creating a tangled dependency web
- Reusing objects is difficult because they carry references to many others
- Chat rooms, air traffic control, event buses, UI form coordination

**When to avoid:**
- The mediator becomes a god object — that just moves the complexity, not reduces it
- Two objects communicate directly with no wider coordination needed

**Python example:**

```python
from abc import ABC, abstractmethod


class Mediator(ABC):
    @abstractmethod
    def notify(self, sender: object, event: str) -> None: ...


class Component:
    def __init__(self, mediator: Mediator) -> None:
        self._mediator = mediator


class LoginForm(Component):
    def submit(self) -> None:
        self._mediator.notify(self, "login_submitted")


class AuthDialog(Mediator):
    def __init__(self) -> None:
        self.login_form = LoginForm(self)
        self.loading_indicator = LoadingIndicator(self)

    def notify(self, sender: object, event: str) -> None:
        if event == "login_submitted":
            self.loading_indicator.show()
            # ... perform auth
```

---

## Memento

**Intent:** Without violating encapsulation, capture and externalise an object's internal
state so the object can be restored to that state later.

**When to use:**
- You need undo/redo or checkpointing
- Direct access to the object's state would violate encapsulation

**When to avoid:**
- The state is trivially reconstructible from other means
- Capturing state is expensive and you don't actually need history

**Python example:**

```python
from __future__ import annotations
from dataclasses import dataclass, replace
import copy


@dataclass(frozen=True)
class EditorMemento:
    """Immutable snapshot of editor state."""
    content: str
    cursor: int


@dataclass
class Editor:
    content: str = ""
    cursor: int = 0

    def save(self) -> EditorMemento:
        return EditorMemento(self.content, self.cursor)

    def restore(self, memento: EditorMemento) -> None:
        self.content = memento.content
        self.cursor = memento.cursor


class EditorHistory:
    def __init__(self) -> None:
        self._stack: list[EditorMemento] = []

    def push(self, memento: EditorMemento) -> None:
        self._stack.append(memento)

    def pop(self) -> EditorMemento | None:
        return self._stack.pop() if self._stack else None
```

---

## Observer

**Intent:** Define a one-to-many dependency between objects so that when one object
changes state, all its dependents are notified and updated automatically.

**When to use:**
- A change to one object requires updating others and you don't know how many
- Event systems, reactive UIs, data binding, pub/sub

**When to avoid:**
- Observers are hard to debug because the notification order may be non-obvious
- Circular update chains can cause infinite loops
- For simple cases, consider a plain callback list

**Python example:**

```python
from abc import ABC, abstractmethod
from collections import defaultdict
from typing import Callable


class EventBus:
    """Lightweight observer using callbacks."""
    def __init__(self) -> None:
        self._subscribers: dict[str, list[Callable]] = defaultdict(list)

    def subscribe(self, event: str, handler: Callable) -> None:
        self._subscribers[event].append(handler)

    def unsubscribe(self, event: str, handler: Callable) -> None:
        self._subscribers[event].remove(handler)

    def publish(self, event: str, data: object = None) -> None:
        for handler in self._subscribers[event]:
            handler(data)


bus = EventBus()
bus.subscribe("order_placed", lambda data: print(f"Send confirmation for {data}"))
bus.subscribe("order_placed", lambda data: print(f"Update inventory for {data}"))
bus.publish("order_placed", {"order_id": "123"})
```

---

## State

**Intent:** Allow an object to alter its behaviour when its internal state changes.
The object will appear to change its class.

**When to use:**
- An object's behaviour depends heavily on its state and must change at runtime
- You have many conditionals that branch based on the same state variable
- State machines with well-defined transitions

**When to avoid:**
- Only a few states with trivial transitions — a simple conditional is clearer
- The state machine is so simple that a dictionary of callbacks suffices

**Python example:**

```python
from abc import ABC, abstractmethod


class TrafficLightState(ABC):
    @abstractmethod
    def next(self, light: "TrafficLight") -> None: ...

    @abstractmethod
    def colour(self) -> str: ...


class GreenState(TrafficLightState):
    def next(self, light: "TrafficLight") -> None:
        light.state = YellowState()

    def colour(self) -> str:
        return "green"


class YellowState(TrafficLightState):
    def next(self, light: "TrafficLight") -> None:
        light.state = RedState()

    def colour(self) -> str:
        return "yellow"


class RedState(TrafficLightState):
    def next(self, light: "TrafficLight") -> None:
        light.state = GreenState()

    def colour(self) -> str:
        return "red"


class TrafficLight:
    def __init__(self) -> None:
        self.state: TrafficLightState = GreenState()

    def advance(self) -> None:
        self.state.next(self)

    def show(self) -> str:
        return self.state.colour()
```

---

## Strategy

**Intent:** Define a family of algorithms, encapsulate each one, and make them
interchangeable. Strategy lets the algorithm vary independently from the clients that use it.

**When to use:**
- You need to swap algorithms or behaviours at runtime
- You have many similar classes that differ only in behaviour
- You want to eliminate conditionals that select between algorithm variants

**Pythonic note:** In Python, strategies are often just callables (functions or lambdas).
A class hierarchy is only warranted when strategies carry state or share interface contracts.

**Python example (callable strategy — preferred for simple cases):**

```python
from collections.abc import Callable
from dataclasses import dataclass


@dataclass
class Order:
    subtotal: float
    customer_tier: str


def no_discount(order: Order) -> float:
    return 0.0


def ten_percent(order: Order) -> float:
    return order.subtotal * 0.10


def twenty_percent(order: Order) -> float:
    return order.subtotal * 0.20


DISCOUNT_STRATEGIES: dict[str, Callable[[Order], float]] = {
    "standard": no_discount,
    "silver": ten_percent,
    "gold": twenty_percent,
}


def calculate_total(order: Order) -> float:
    strategy = DISCOUNT_STRATEGIES.get(order.customer_tier, no_discount)
    return order.subtotal - strategy(order)
```

---

## Template Method

**Intent:** Define the skeleton of an algorithm in a base class, deferring some steps to
subclasses. Subclasses can redefine certain steps without changing the algorithm's structure.

**When to use:**
- Multiple classes share the same algorithm structure but differ in specific steps
- You want to control which parts subclasses are permitted to override (hook methods)

**When to avoid:**
- Composition is usually more flexible than inheritance — if the steps can be injected
  as collaborators, prefer Strategy or dependency injection over Template Method
- Deep inheritance hierarchies become hard to follow

**Python example:**

```python
from abc import ABC, abstractmethod


class ReportGenerator(ABC):
    def generate(self) -> str:
        """Template method — defines the algorithm skeleton."""
        data = self.fetch_data()
        processed = self.process(data)
        return self.format(processed)

    @abstractmethod
    def fetch_data(self) -> list[dict]: ...

    def process(self, data: list[dict]) -> list[dict]:
        """Hook with a default implementation subclasses may override."""
        return data

    @abstractmethod
    def format(self, data: list[dict]) -> str: ...


class CSVReportGenerator(ReportGenerator):
    def fetch_data(self) -> list[dict]:
        return [{"id": 1, "name": "Alice"}]

    def format(self, data: list[dict]) -> str:
        header = ",".join(data[0].keys())
        rows = "\n".join(",".join(str(v) for v in row.values()) for row in data)
        return f"{header}\n{rows}"
```

---

## Visitor

**Intent:** Represent an operation to be performed on elements of an object structure.
Visitor lets you define a new operation without changing the classes of the elements it acts on.

**When to use:**
- You need to perform many distinct operations on a stable object structure
- You want to add operations to a class hierarchy without modifying those classes
- AST processing, document rendering, tax calculations on order items

**When to avoid:**
- This is the most complex of all 23 patterns — only use it when you genuinely need to
  add many operations to a stable hierarchy
- If the element class hierarchy changes frequently, every visitor must change too
- Python's `singledispatch` often achieves the same goal with less ceremony

**Python example (using `singledispatch`):**

```python
from functools import singledispatch
from dataclasses import dataclass


@dataclass
class Circle:
    radius: float


@dataclass
class Rectangle:
    width: float
    height: float


@singledispatch
def area(shape: object) -> float:
    raise NotImplementedError(f"No area implementation for {type(shape)}")


@area.register
def _(shape: Circle) -> float:
    import math
    return math.pi * shape.radius ** 2


@area.register
def _(shape: Rectangle) -> float:
    return shape.width * shape.height


shapes = [Circle(5), Rectangle(3, 4)]
for s in shapes:
    print(area(s))
```
