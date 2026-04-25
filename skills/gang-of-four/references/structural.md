# Structural Patterns

Seven patterns that describe how classes and objects are composed into larger structures.

---

## Adapter

**Intent:** Convert the interface of a class into another interface that clients expect.
Lets classes work together that otherwise couldn't because of incompatible interfaces.

**When to use:**
- Integrating a third-party library whose interface doesn't match yours
- Reusing existing code that has a different, incompatible interface
- You want to wrap a legacy component without modifying it

**When to avoid:**
- You control both sides — just align the interfaces directly

**Python example:**

```python
from abc import ABC, abstractmethod


class PaymentGateway(ABC):
    """Interface our application expects."""
    @abstractmethod
    def charge(self, amount_cents: int, currency: str) -> str: ...


class StripeClient:
    """Third-party library with a different interface."""
    def create_charge(self, amount: float, currency_code: str) -> dict:
        return {"id": "ch_123", "status": "succeeded"}


class StripeAdapter(PaymentGateway):
    """Adapts StripeClient to our PaymentGateway interface."""
    def __init__(self, client: StripeClient) -> None:
        self._client = client

    def charge(self, amount_cents: int, currency: str) -> str:
        result = self._client.create_charge(amount_cents / 100, currency.upper())
        return result["id"]
```

---

## Bridge

**Intent:** Decouple an abstraction from its implementation so that the two can vary
independently. Avoids an explosion of subclasses when you need to combine two dimensions
of variation.

**When to use:**
- You have two orthogonal dimensions (e.g., shape × colour, platform × feature)
- You want to switch implementations at runtime
- Inheritance would create an unwieldy class hierarchy

**When to avoid:**
- Only one dimension of variation exists — straightforward inheritance is clearer

**Python example:**

```python
from abc import ABC, abstractmethod


class Renderer(ABC):
    @abstractmethod
    def render_circle(self, radius: float) -> str: ...


class SVGRenderer(Renderer):
    def render_circle(self, radius: float) -> str:
        return f'<circle r="{radius}"/>'


class CanvasRenderer(Renderer):
    def render_circle(self, radius: float) -> str:
        return f"ctx.arc(0, 0, {radius}, 0, 2*Math.PI)"


class Shape(ABC):
    def __init__(self, renderer: Renderer) -> None:
        self._renderer = renderer

    @abstractmethod
    def draw(self) -> str: ...


class Circle(Shape):
    def __init__(self, renderer: Renderer, radius: float) -> None:
        super().__init__(renderer)
        self._radius = radius

    def draw(self) -> str:
        return self._renderer.render_circle(self._radius)


# SVG circle or Canvas circle — renderer swapped independently of shape
svg_circle = Circle(SVGRenderer(), 10)
canvas_circle = Circle(CanvasRenderer(), 10)
```

---

## Composite

**Intent:** Compose objects into tree structures to represent part-whole hierarchies.
Lets clients treat individual objects and compositions uniformly.

**When to use:**
- You have a tree structure (file system, UI widget hierarchy, org chart, arithmetic expressions)
- Clients should be able to ignore the difference between leaf and composite nodes

**When to avoid:**
- Your structure is not hierarchical — don't force a tree where a flat list would do

**Python example:**

```python
from abc import ABC, abstractmethod


class FileSystemItem(ABC):
    def __init__(self, name: str) -> None:
        self.name = name

    @abstractmethod
    def size(self) -> int: ...

    @abstractmethod
    def display(self, indent: int = 0) -> None: ...


class File(FileSystemItem):
    def __init__(self, name: str, size_bytes: int) -> None:
        super().__init__(name)
        self._size = size_bytes

    def size(self) -> int:
        return self._size

    def display(self, indent: int = 0) -> None:
        print(" " * indent + f"{self.name} ({self._size} bytes)")


class Directory(FileSystemItem):
    def __init__(self, name: str) -> None:
        super().__init__(name)
        self._children: list[FileSystemItem] = []

    def add(self, item: FileSystemItem) -> None:
        self._children.append(item)

    def size(self) -> int:
        return sum(child.size() for child in self._children)

    def display(self, indent: int = 0) -> None:
        print(" " * indent + f"{self.name}/")
        for child in self._children:
            child.display(indent + 2)
```

---

## Decorator

**Intent:** Attach additional responsibilities to an object dynamically. Decorators provide
a flexible alternative to subclassing for extending functionality.

**When to use:**
- You need to add behaviour to individual objects without affecting others of the same class
- Behaviour combinations would create too many subclasses
- You want to compose behaviours at runtime

**When to avoid:**
- A simple subclass would be clearer and there's no need for runtime composition
- Python's built-in `functools.wraps` and `@` decorator syntax already solve the function
  case elegantly — no class hierarchy needed

**Python example (class-based):**

```python
from abc import ABC, abstractmethod


class DataSource(ABC):
    @abstractmethod
    def write(self, data: bytes) -> None: ...

    @abstractmethod
    def read(self) -> bytes: ...


class FileDataSource(DataSource):
    def __init__(self, path: str) -> None:
        self._path = path

    def write(self, data: bytes) -> None:
        with open(self._path, "wb") as f:
            f.write(data)

    def read(self) -> bytes:
        with open(self._path, "rb") as f:
            return f.read()


class DataSourceDecorator(DataSource):
    def __init__(self, wrapped: DataSource) -> None:
        self._wrapped = wrapped

    def write(self, data: bytes) -> None:
        self._wrapped.write(data)

    def read(self) -> bytes:
        return self._wrapped.read()


class CompressionDecorator(DataSourceDecorator):
    def write(self, data: bytes) -> None:
        import zlib
        self._wrapped.write(zlib.compress(data))

    def read(self) -> bytes:
        import zlib
        return zlib.decompress(self._wrapped.read())


class EncryptionDecorator(DataSourceDecorator):
    def write(self, data: bytes) -> None:
        # Simplified — use a real cipher in production
        self._wrapped.write(bytes(b ^ 0xFF for b in data))

    def read(self) -> bytes:
        return bytes(b ^ 0xFF for b in self._wrapped.read())


# Compose: file → compress → encrypt
source = EncryptionDecorator(CompressionDecorator(FileDataSource("/tmp/data.bin")))
source.write(b"hello world")
```

**Pythonic note:** For functions and methods, Python's `@decorator` syntax is idiomatic and
avoids the class hierarchy entirely.

---

## Facade

**Intent:** Provide a simplified interface to a complex subsystem. Hides internal complexity
behind a convenient, high-level API.

**When to use:**
- A subsystem is complex and most clients need only a simple subset of its capabilities
- You want to layer a system: high-level API on top of lower-level components
- Reducing coupling between clients and a complex subsystem

**When to avoid:**
- The facade becomes a "god object" that does everything — split it instead

**Python example:**

```python
class VideoConverter:
    """Facade over a complex video processing subsystem."""

    def convert(self, filename: str, target_format: str) -> str:
        file = VideoFile(filename)
        source_codec = CodecFactory.extract(file)
        destination_codec = CodecFactory.find(target_format)
        buffer = BitrateReader.read(filename, source_codec)
        result = BitrateReader.convert(buffer, destination_codec)
        output = AudioMixer.fix(result)
        output_name = filename.replace(file.extension, target_format)
        output.save(output_name)
        return output_name
```

The client calls `VideoConverter().convert("video.avi", "mp4")` without knowing anything
about codecs, bitrate readers, or audio mixers.

---

## Flyweight

**Intent:** Use sharing to support large numbers of fine-grained objects efficiently.
Separate intrinsic state (shared) from extrinsic state (context-specific).

**When to use:**
- You need a very large number of similar objects and memory is a constraint
- Most object state can be externalised (passed in rather than stored)

**When to avoid:**
- You don't have a memory problem — premature optimisation
- The objects are not sufficiently similar to share meaningful state

**Python example:**

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class GlyphStyle:
    """Intrinsic (shared) state — font, size, colour."""
    font: str
    size: int
    colour: str


class GlyphStyleCache:
    _cache: dict[tuple, GlyphStyle] = {}

    @classmethod
    def get(cls, font: str, size: int, colour: str) -> GlyphStyle:
        key = (font, size, colour)
        if key not in cls._cache:
            cls._cache[key] = GlyphStyle(font, size, colour)
        return cls._cache[key]


@dataclass
class Glyph:
    """Extrinsic (context-specific) state — position."""
    char: str
    x: int
    y: int
    style: GlyphStyle  # shared flyweight


# 10,000 glyphs — but only a handful of GlyphStyle objects
glyphs = [
    Glyph(char="A", x=i, y=0, style=GlyphStyleCache.get("Arial", 12, "black"))
    for i in range(10_000)
]
```

---

## Proxy

**Intent:** Provide a surrogate or placeholder for another object to control access to it.

**Proxy types:**
- **Virtual proxy** — defers expensive creation until needed (lazy initialisation)
- **Protection proxy** — controls access based on permissions
- **Remote proxy** — represents an object in a different process or network location
- **Caching proxy** — caches results of expensive operations

**When to use:**
- You need lazy initialisation of a heavyweight object
- You need access control without modifying the real subject
- You want transparent caching or logging around an object

**When to avoid:**
- A simple function wrapper would be clearer (especially for caching — use `functools.lru_cache`)

**Python example (virtual + caching proxy):**

```python
from abc import ABC, abstractmethod
from functools import lru_cache


class ImageLoader(ABC):
    @abstractmethod
    def display(self) -> str: ...


class RealImageLoader(ImageLoader):
    def __init__(self, path: str) -> None:
        self._path = path
        self._data = self._load()

    def _load(self) -> bytes:
        print(f"Loading image from disk: {self._path}")
        return b"<image data>"

    def display(self) -> str:
        return f"Displaying {self._path}"


class LazyImageProxy(ImageLoader):
    """Defers loading until display() is first called."""
    def __init__(self, path: str) -> None:
        self._path = path
        self._real: RealImageLoader | None = None

    def display(self) -> str:
        if self._real is None:
            self._real = RealImageLoader(self._path)
        return self._real.display()


# Image is not loaded until display() is called
image = LazyImageProxy("/var/assets/hero.png")
print("Proxy created — no disk I/O yet")
print(image.display())  # I/O happens here
```

**Pythonic note:** `functools.lru_cache` and `cached_property` handle caching proxies
elegantly without a class hierarchy.
