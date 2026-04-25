# When to Use Which Pattern

A decision-making reference. Start with the problem, not the pattern.

---

## The First Question

Before reaching for any pattern, ask: **is the problem complex enough to warrant abstraction?**

- One concrete class? Just use it.
- One algorithm? Just write it.
- One configuration? Use a module-level constant.

A pattern is only warranted when the simplest solution genuinely fails to address a real,
current need.

---

## Problem → Pattern Cheat Sheet

### Object Creation Problems

| Problem | Pattern | Notes |
|---------|---------|-------|
| Creating objects without coupling to their concrete class | Factory Method | Use when the exact type varies by context or subclass |
| Creating families of related objects that must be compatible | Abstract Factory | Use when you need consistent product families (e.g., themes, platforms) |
| Constructing a complex object with many optional parts | Builder | Use when telescoping constructors appear; Python kwargs often suffice |
| Copying an existing object is cheaper than constructing | Prototype | Use `copy.deepcopy` in Python |
| One shared instance of a resource | Singleton | Use a module-level variable first; class only if thread-safety matters |

### Structural Problems

| Problem | Pattern | Notes |
|---------|---------|-------|
| Third-party interface doesn't match what your code expects | Adapter | The go-to integration pattern |
| Two dimensions of variation (e.g., shape × renderer) | Bridge | Prevents subclass explosion |
| Tree structure where leaves and branches are treated the same | Composite | File systems, UI hierarchies, expressions |
| Adding behaviour without subclassing | Decorator | Python's `@decorator` syntax often replaces class-based Decorator |
| Simplifying access to a complex subsystem | Facade | Use as a public API layer |
| Very large number of similar lightweight objects | Flyweight | Only when memory is actually a problem |
| Controlling access to an object (lazy init, caching, ACL) | Proxy | Python's `cached_property` handles many caching proxy cases |

### Behavioral Problems

| Problem | Pattern | Notes |
|---------|---------|-------|
| Multiple potential handlers; handler unknown at design time | Chain of Responsibility | Middleware, approval chains |
| Undoable operations, queued or scheduled actions | Command | Classic undo/redo pattern |
| Simple domain-specific language or grammar | Interpreter | Rarely needed; prefer a parser library |
| Traverse a collection without exposing its structure | Iterator | Python's `__iter__`/generators make this built-in |
| Coordinating communication between many objects | Mediator | Avoid if it becomes a god object |
| Capture and restore object state | Memento | Undo history, checkpointing |
| Notify dependents when state changes | Observer | Event systems, pub/sub, reactive UIs |
| Object behaviour changes based on state | State | Prefer over long switch/if-else chains |
| Swappable algorithms or policies | Strategy | Use callables in Python unless strategies carry state |
| Algorithm skeleton with overridable steps | Template Method | Prefer composition (Strategy) when possible |
| Operations on a stable class hierarchy without modifying it | Visitor | Complex; use `singledispatch` in Python first |

---

## Similar Patterns: How to Choose

### Factory Method vs Abstract Factory

- **Factory Method**: one product, one creator. Subclasses decide which product.
- **Abstract Factory**: families of related products. All products in a family must be compatible.
- If you only need one product type, use Factory Method. If you need coordinated families, use Abstract Factory.

### Strategy vs Template Method

- **Strategy**: algorithm is swapped via composition (passed in). More flexible.
- **Template Method**: algorithm is fixed in a base class; subclasses override specific steps.
- Prefer Strategy — it avoids inheritance and is easier to test.

### Strategy vs State

- Both use polymorphism to change behaviour.
- **Strategy**: client chooses the strategy; it doesn't change on its own.
- **State**: the context transitions between states automatically based on events.

### Decorator vs Proxy

- Both wrap another object.
- **Decorator**: adds behaviour (extends the interface).
- **Proxy**: controls access (same interface, different lifecycle/permission/caching logic).

### Facade vs Mediator

- **Facade**: simplifies a subsystem for external clients. One-directional.
- **Mediator**: coordinates communication between peers. Bidirectional.

### Observer vs Mediator

- **Observer**: objects observe a subject; subject doesn't know who is listening.
- **Mediator**: a central object coordinates specific interactions between specific peers.

### Adapter vs Bridge

- **Adapter**: fixes incompatibility after the fact (works with existing interfaces).
- **Bridge**: designed up front to separate abstraction from implementation.

### Command vs Strategy

- **Command**: encapsulates a request as an object (supports undo, queuing, logging).
- **Strategy**: encapsulates an algorithm as an object (supports swapping at runtime).
- Commands represent *what to do* (a verb); strategies represent *how to do it* (an algorithm).

---

## Patterns Grouped by Scenario

### Undo / Redo
→ **Command** (encapsulate operations) + **Memento** (capture state snapshots)

### Plugin / Extension System
→ **Factory Method** or **Abstract Factory** (create plugin instances) + **Strategy** (swap behaviours)

### Event-Driven Architecture
→ **Observer** (pub/sub notifications) + **Command** (encapsulate events as objects)

### Parsing / AST Processing
→ **Composite** (tree structure) + **Visitor** (operations on the tree)

### Layered Architecture
→ **Facade** (public API per layer) + **Adapter** (integrate external dependencies)

### State Machine
→ **State** (per-state behaviour) + optionally **Command** (transitions as commands)

---

## Anti-Patterns: When Patterns Hurt

| Pattern | Common misuse | Better alternative |
|---------|--------------|-------------------|
| Singleton | Used as a lazy global variable | Module-level variable or dependency injection |
| Factory Method | Factory for a single, never-changing type | Direct constructor call |
| Abstract Factory | One product family that never changes | Factory Method or constructor |
| Visitor | Applied to a frequently-changing class hierarchy | Simple method on each class |
| Interpreter | Built for a non-trivial language | `lark`, `pyparsing`, or another parser library |
| Decorator | Chained 4+ deep, making control flow opaque | Refactor to explicit composition |
| Mediator | Becomes a god object knowing everything | Split into smaller coordinators |
| Template Method | Forces inheritance when composition would do | Strategy pattern + dependency injection |

---

## Python-Specific Guidance

Several GoF patterns are largely subsumed by Python idioms:

| GoF Pattern | Python replacement |
|-------------|-------------------|
| Iterator | `__iter__`, generators, `yield` |
| Singleton | Module-level variables |
| Strategy (stateless) | First-class functions and callables |
| Decorator (function) | `@decorator` syntax and `functools.wraps` |
| Prototype | `copy.deepcopy` |
| Visitor | `functools.singledispatch` |
| Template Method | Callable arguments (Strategy-style composition) |
| Observer (simple) | Callback lists; `weakref`-based event systems |

This does not mean these patterns are irrelevant in Python — it means the idiomatic
Python version is often lighter than the classic class-hierarchy formulation.
