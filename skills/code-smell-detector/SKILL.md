---
name: code-smell-detector
description: 'Detects and fixes common code smells during review or refactoring. Invoke whenever reviewing code for quality issues, before merging a PR, when refactoring legacy code, or when the user asks about code quality, anti-patterns, or technical debt. Detects: over-abstraction, complex inheritance, large functions, tight coupling, hidden dependencies, magic numbers, boolean traps, swallowed exceptions, global state, and duplicate code. Provides specific fixes with before/after examples. Also invoke when someone says "review this code", "is this clean?", "can I improve this?", "this feels messy", or "find problems in my code".'
---

# Code Smell Detector

This skill identifies common anti-patterns and provides constructive, specific fixes. It ensures
code maintains simplicity, clear responsibility, and testability.

> **Relation to existing instructions**: The instruction files in `instructions/core/` define
> *preventative* rules (what to do when writing new code). This skill is *detective* — use it when
> reviewing or refactoring existing code. Where a smell maps to an instruction file rule, a
> cross-reference is noted so you understand the canonical guidance.

## When to Use This Skill

- **Code review**: Identify violations before merging
- **Refactoring**: Find opportunities to simplify
- **New module creation**: Catch issues early
- **Learning/mentoring**: Understand why patterns are problematic

---

## Smells Detected

### 1. Over-Abstraction

**Why it matters**: Unnecessary abstraction adds complexity without benefit. Every layer must
justify its existence — if you can't name a concrete reason to have it, it shouldn't exist.

**Red flags**:
- Abstract base classes with only one concrete implementation
- Generic helper classes that hold no state and do very little
- Deep inheritance hierarchies (3+ levels)
- Over-parameterised functions with feature-flag booleans

**Example**:
```python
# SMELL: Abstraction for one implementation
class DataProcessor(ABC):
    @abstractmethod
    def process(self, data): pass

class SimpleDataProcessor(DataProcessor):
    def process(self, data):
        return data * 2

# FIX: Direct implementation
def process_data(data):
    return data * 2
```

**Detection rules**:
- ABC with ≤ 2 concrete implementations → likely premature abstraction
- Utility class with only static methods and no state → use module-level functions instead
- Mixin that solves a single problem → inline or extract to a helper function

> Cross-reference: `instructions/core/engineering-principles.instructions.md` (YAGNI & KISS)

---

### 2. Complex Inheritance

**Why it matters**: Deep inheritance chains obscure what code does and who owns what. Composition
makes dependencies explicit and code independently testable.

**Red flags**:
- Inheritance depth > 2 levels
- Multiple inheritance from concrete (non-interface) classes
- Methods overridden at multiple levels of the hierarchy
- Inheritance used for code reuse rather than substitutability

**Example**:
```python
# SMELL: Inheritance chain
class Entity:
    def save(self): pass

class TimestampedEntity(Entity):
    def timestamp(self): pass

class AuditableEntity(TimestampedEntity):
    def audit(self): pass

class User(AuditableEntity):
    def authenticate(self): pass

# FIX: Composition
class User:
    def __init__(self, storage, timestamps, audit):
        self.storage = storage
        self.timestamps = timestamps
        self.audit = audit

    def save(self):
        self.storage.save(self)
        self.timestamps.record()
        self.audit.log("saved user")
```

**Detection rules**:
- Follow inheritance chain; flag depth > 2
- Multiple inheritance from concrete classes → flag
- `super()` calls at multiple levels → flag

> Cross-reference: `instructions/core/engineering-principles.instructions.md` (Dependency
> Inversion). For GoF structural patterns (Decorator, Proxy, etc.) as alternatives, invoke
> the `gang-of-four` skill.

---

### 3. Large Functions (> 50 Lines)

**Why it matters**: Large functions do multiple things, making them hard to name, test, and
modify. If a function is hard to summarise in one sentence, it should be split.

**Red flags**:
- Function body > 50 lines (excluding docstring)
- Nesting depth ≥ 3 levels
- 5+ parameters
- Mixed levels of abstraction in one function

**Example**:
```python
# SMELL: One function doing everything
def process_user_data(user_dict, validate=True, save=True, notify=True):
    if validate:
        if not user_dict.get('email'):
            raise ValueError("Email required")
        # ... more validation
    user = User(...)
    if save:
        db.save(user)
    if notify:
        email_service.send(user.email, "Welcome!")
    return user

# FIX: Separate concerns, compose at the top level
def validate_user_data(user_dict): ...
def create_user(user_dict): ...

def process_user_data(user_dict):
    validate_user_data(user_dict)
    user = create_user(user_dict)
    db.save(user)
    email_service.send(user.email, "Welcome!")
    return user
```

> Cross-reference: `instructions/core/engineering-principles.instructions.md` (Function Size)
> and `instructions/core/coding-style.instructions.md` (Code Quality Checklist).

---

### 4. Tight Coupling / Hidden Dependencies

**Why it matters**: When a class instantiates its own dependencies, you cannot test it in
isolation and cannot swap implementations. Coupling two things together means changing one
risks breaking the other.

**Red flags**:
- `ServiceName()` instantiated inside a method
- Deep attribute chaining: `obj.service.repo.data` (3+ dots)
- Circular imports between modules
- Global variable reads inside functions

**Example**:
```python
# SMELL: Hidden dependency
class UserService:
    def create_user(self, name, email):
        db = Database()          # Impossible to swap for tests
        email_svc = EmailService()
        user = db.save(name, email)
        email_svc.send(email, "Welcome!")
        return user

# FIX: Inject dependencies
class UserService:
    def __init__(self, db, email_svc):
        self.db = db
        self.email_svc = email_svc

    def create_user(self, name, email):
        user = self.db.save(name, email)
        self.email_svc.send(email, "Welcome!")
        return user
```

> Cross-reference: `instructions/core/engineering-principles.instructions.md`
> (Dependency Inversion).

---

### 5. Missing `__all__` Exports *(Python-specific)*

**Why it matters**: Without `__all__`, a module's public interface is ambiguous. Consumers
may accidentally import internal helpers, making refactoring harder.

**Red flags**:
- No `__all__` in `__init__.py`
- Internal names (prefixed `_`) re-exported accidentally
- Unclear what callers should use

**Example**:
```python
# SMELL: No public interface defined
from .core import process_data, _internal_helper
from .utils import validate_input, LOG_LEVEL

# FIX: Explicit public API
from .core import process_data
from .utils import validate_input

__all__ = ['process_data', 'validate_input']
```

---

### 6. Magic Numbers and Strings

**Why it matters**: Bare literals reveal nothing about intent. When the same literal appears in
multiple places, a future change silently misses one.

**Red flags**:
- Numeric literals other than 0, 1, or -1 in logic
- Hardcoded strings used as identifiers, status codes, or limits
- Same literal repeated in multiple places

**Example**:
```python
# SMELL
if retries > 3:
    time.sleep(5)

# FIX
MAX_RETRIES = 3       # Based on P99 network reliability measurements
RETRY_DELAY_SEC = 5   # Lambda max 15s; leaves headroom for retries

if retries > MAX_RETRIES:
    time.sleep(RETRY_DELAY_SEC)
```

> Cross-reference: `instructions/core/coding-style.instructions.md` (Code Quality Checklist:
> "No hardcoded values").

---

### 7. Boolean Trap (Flag Parameters)

**Why it matters**: Boolean parameters that change a function's fundamental behaviour are
a sign the function does two things. Call sites become unreadable.

**Red flags**:
- `process(data, True, False, True)` — positional booleans are unreadable
- Single boolean that causes fundamentally different code paths inside the function
- Functions whose name cannot describe both modes

**Example**:
```python
# SMELL: Flag changes fundamental behaviour
def render(data, as_html=False):
    if as_html:
        return f"<div>{data}</div>"
    return str(data)

# FIX: Separate named functions
def render_text(data): return str(data)
def render_html(data): return f"<div>{data}</div>"
```

---

### 8. Swallowed Exceptions

**Why it matters**: Catching and ignoring exceptions hides real failures. Code that silently
returns a default on error is lying to its callers.

**Red flags**:
- `except Exception: pass`
- `except Exception: return {}`
- `except: pass` (bare except)
- Catching broad exceptions and doing nothing with them

**Example**:
```python
# SMELL: Silent failure
def load_config(path):
    try:
        return json.load(open(path))
    except Exception:
        return {}   # Hides missing file, corrupt JSON, permissions errors

# FIX: Explicit handling with context
def load_config(path: str) -> dict:
    try:
        with open(path) as f:
            return json.load(f)
    except FileNotFoundError:
        raise ConfigError(f"Config file not found: {path}")
    except json.JSONDecodeError as exc:
        raise ConfigError(f"Invalid JSON in {path}: {exc}") from exc
```

**Deep analysis — ask these questions for every catch block:**

1. **Specificity**: Is only the expected exception type caught? List every *unrelated* exception
   this block could silently swallow (e.g., a broad `except Exception` around file I/O also
   catches `KeyboardInterrupt`, `MemoryError`, and any bug in the code inside the try block).

2. **Logging quality**: Is the error logged with enough context to debug it six months from now?
   A bare `logger.error("Failed")` is nearly useless — it needs the operation name, relevant IDs,
   and the original exception.

3. **Fallback justification**: If the catch block returns a default value or falls back to
   alternative behaviour, is that fallback explicitly required and documented, or is it masking
   a real problem the caller should know about?

4. **Propagation**: Should this error propagate to a higher-level handler instead of being caught
   here? Catching too low obscures the call chain and prevents proper cleanup.

5. **User impact**: If this error silently succeeds, what will the user experience? Corrupted
   state? Stale data? A success response on an operation that actually failed?

> Cross-reference: `instructions/core/engineering-principles.instructions.md` (Error Handling).

---

### 9. Global State

**Why it matters**: Global variables make execution order unpredictable and tests order-dependent.
Any function that reads or writes global state becomes an implicit dependency of every caller.

**Red flags**:
- Module-level mutable variables that are written to at runtime
- Singleton patterns that store mutable state
- Functions that modify `global x` inside their body
- Class-level mutable attributes used as instance-shared state

**Example**:
```python
# SMELL: Global mutable cache
_cache = {}

def get_user(user_id):
    if user_id not in _cache:
        _cache[user_id] = db.fetch(user_id)
    return _cache[user_id]

# FIX: Pass state explicitly
class UserRepository:
    def __init__(self, db, cache: dict | None = None):
        self.db = db
        self._cache = cache if cache is not None else {}

    def get(self, user_id):
        if user_id not in self._cache:
            self._cache[user_id] = self.db.fetch(user_id)
        return self._cache[user_id]
```

---

### 10. Duplicate Code

**Why it matters**: Every duplicated block is a future maintenance hazard. When the logic needs
to change, you must find and update every copy.

**Red flags**:
- Near-identical code blocks in multiple places
- Copy-pasted validation or transformation logic
- The same multi-step sequence repeated across different functions

**Example**:
```python
# SMELL: Same validation in two places
def process_payment(amount):
    if amount <= 0:
        raise ValueError("Amount must be positive")
    ...

def process_refund(amount):
    if amount <= 0:
        raise ValueError("Amount must be positive")
    ...

# FIX: Extract once
def _require_positive_amount(amount: float) -> None:
    if amount <= 0:
        raise ValueError("Amount must be positive")

def process_payment(amount): _require_positive_amount(amount); ...
def process_refund(amount):  _require_positive_amount(amount); ...
```

> Cross-reference: `instructions/core/engineering-principles.instructions.md` (DRY).

---

## Analysis Process

### Step 1: Scan Code Structure

1. Review file sizes and module boundaries (flag files > 800 lines)
2. Identify inheritance hierarchies — draw the tree
3. Scan function sizes (line count, nesting depth)
4. Check for `__all__` in Python packages
5. Note class instantiation inside methods

### Step 2: Triage Each Smell by Severity

| Severity | Criteria |
|----------|----------|
| **Critical** | Breaks testability, causes data loss, or masks errors |
| **Major** | Significantly hampers readability or makes change risky |
| **Minor** | Style issue; improvement but not urgent |

Swallowed exceptions and global state → Critical  
Tight coupling, large functions → Major  
Over-abstraction, missing `__all__` → Major or Minor depending on scope

### Step 3: Generate Fixes

For each smell found:
1. State which smell it is and why it matters
2. Quote the specific lines (with line numbers if possible)
3. Show BEFORE and AFTER code
4. Give concrete refactoring steps

### Step 4: Report

1. List smells found, grouped by severity
2. Include specific examples from the actual code
3. Provide actionable fix steps
4. Be constructive — the goal is learning and improvement, not criticism

---

## Quick Detection Reference

| Symptom | Smell | Primary Fix |
|---------|-------|-------------|
| ABC with one impl | Over-abstraction | Delete the abstract class |
| Class hierarchy > 2 deep | Complex inheritance | Composition + injection |
| Function > 50 lines | Large function | Extract helpers |
| `Foo()` inside a method | Tight coupling | Constructor injection |
| No `__all__` in `__init__.py` | Missing exports | Add `__all__` |
| Bare numeric/string literals | Magic values | Named constants |
| `func(data, True, False)` | Boolean trap | Split into two functions |
| `except Exception: pass` | Swallowed exception | Raise with context |
| Module-level mutable variable | Global state | Pass as parameter or inject |
| Same block in 2+ places | Duplicate code | Extract shared function |

---

## Common Patterns and Their Fixes

### "Utility Class" Holder
```python
# SMELL
class StringUtils:
    @staticmethod
    def clean(s): return s.strip().lower()

# FIX: Module-level function
def clean_string(s): return s.strip().lower()
```

### "Manager" Class (does everything)
```python
# SMELL
class UserManager:
    def create(self): ...
    def update(self): ...
    def validate(self): ...
    def send_email(self): ...

# FIX: Split by responsibility
class UserService:     # lifecycle operations
class UserValidator:   # validation only
class UserNotifier:    # notifications only
```

### God Function
```python
# SMELL: 200-line function with mixed concerns
def process_order(order_data, validate, save, notify, log): ...

# FIX: Composed workflow of focused functions
def process_order(order_data):
    validate_order(order_data)
    order = build_order(order_data)
    save_order(order)
    notify_customer(order)
```
