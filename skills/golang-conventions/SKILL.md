---
name: golang-conventions
description: Go coding standards for Go 1.21+, including idioms, error handling, testing patterns, concurrency, and golangci-lint configuration. Use when writing, reviewing, or refactoring Go code, working with goroutines, or setting up Go projects.
license: Proprietary. See parent repository LICENSE
---

# Go Coding Conventions

## Quick Reference

- **Go version**: Use Go 1.21+ for modern features (generics, slices package, cmp)
- **Formatting**: Use `gofmt` and `goimports` (automatic via gopls)
- **Linting**: Use `golangci-lint` with comprehensive linters enabled
- **Testing**: Table-driven tests, test fixtures, subtests with `t.Run()`
- **Error handling**: Explicit, descriptive errors with `errors.Is()` and `errors.As()`

## Core Principles

### 1. Simplicity and Clarity

- Write clear, idiomatic Go code
- Prefer simple solutions over clever ones
- Follow the principle of least surprise
- "Clear is better than clever" - Go Proverbs

### 2. Explicit Over Implicit

- No hidden control flow
- Explicit error handling (no exceptions)
- Clear dependency management
- Visible type conversions

### 3. Composition Over Inheritance

- Use interfaces for abstraction
- Prefer struct embedding over inheritance
- Accept interfaces, return structs
- Keep interfaces small and focused

## Code Style and Formatting

### Naming Conventions

```go
// Package names: lowercase, single word, no underscores
package user

// Exported identifiers: PascalCase
type UserService struct {}
func NewUserService() *UserService {}

// Unexported identifiers: camelCase
type userData struct {}
func validateEmail(email string) error {}

// Interface names: single method interfaces end with -er
type Reader interface { Read(p []byte) (n int, err error) }
type UserRepository interface { /* multiple methods */ }

// Acronyms: consistent case (HTTP, ID, URL, not Http, Id, Url)
type HTTPServer struct {}
var userID int64
```

### File Organisation

```go
// 1. Package declaration
package user

// 2. Import statements (grouped: stdlib, external, internal)
import (
    "context"
    "fmt"
    "time"

    "github.com/google/uuid"

    "myapp/internal/database"
)

// 3. Constants
const (
    MaxRetries = 3
    DefaultTimeout = 30 * time.Second
)

// 4. Variables (avoid package-level vars, prefer const or functions)
var (
    ErrUserNotFound = errors.New("user not found")
)

// 5. Types
type User struct {
    ID        uuid.UUID
    Email     string
    CreatedAt time.Time
}

// 6. Functions and methods (exported first, then unexported)
```

### Modern Go Features (1.21+)

```go
// Generics for type-safe collections
func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// min/max built-ins
func clamp(val, minVal, maxVal int) int {
    return max(minVal, min(val, maxVal))
}

// clear() built-in for maps and slices
func resetCache(cache map[string]any) {
    clear(cache)
}

// slices package for common operations
import "slices"
sorted := slices.Clone(original)
slices.Sort(sorted)
```

## Error Handling

### Explicit and Descriptive

```go
// Good: Clear error handling
func GetUser(ctx context.Context, id uuid.UUID) (*User, error) {
    user, err := db.FindUserByID(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("failed to get user %s: %w", id, err)
    }
    return user, nil
}

// Use errors.Is() and errors.As() for sentinel errors
if errors.Is(err, ErrUserNotFound) {
    return handleNotFound()
}

var validationErr *ValidationError
if errors.As(err, &validationErr) {
    return handleValidation(validationErr)
}

// Custom error types for rich error information
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
}
```

### Error Wrapping

```go
// Wrap errors to preserve context
func processUser(ctx context.Context, id uuid.UUID) error {
    user, err := GetUser(ctx, id)
    if err != nil {
        return fmt.Errorf("process user failed: %w", err)
    }

    if err := validateUser(user); err != nil {
        return fmt.Errorf("validation failed for user %s: %w", id, err)
    }

    return nil
}
```

## Testing

### Table-Driven Tests

```go
func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        {
            name:    "valid email",
            email:   "user@example.com",
            wantErr: false,
        },
        {
            name:    "missing at sign",
            email:   "userexample.com",
            wantErr: true,
        },
        {
            name:    "empty email",
            email:   "",
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateEmail(tt.email)
            if (err != nil) != tt.wantErr {
                t.Errorf("ValidateEmail() error = %v, wantErr %v", err, tt.wantErr)
            }
        })
    }
}
```

### Test Fixtures and Helpers

```go
// testdata/ directory for fixtures
// helper_test.go for test utilities

func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()

    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatalf("failed to open test db: %v", err)
    }

    t.Cleanup(func() {
        db.Close()
    })

    return db
}

// Use testify for assertions (optional but popular)
import "github.com/stretchr/testify/assert"

func TestUserCreation(t *testing.T) {
    user := NewUser("test@example.com")

    assert.NotNil(t, user.ID)
    assert.Equal(t, "test@example.com", user.Email)
    assert.False(t, user.CreatedAt.IsZero())
}
```

### Benchmarks and Examples

```go
// Benchmark functions
func BenchmarkValidateEmail(b *testing.B) {
    email := "user@example.com"

    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        ValidateEmail(email)
    }
}

// Example functions (appear in godoc)
func ExampleValidateEmail() {
    err := ValidateEmail("user@example.com")
    fmt.Println(err == nil)
    // Output: true
}
```

## Project Structure

### Standard Layout

```
myproject/
├── cmd/                    # Main applications
│   └── server/
│       └── main.go
├── internal/               # Private application code
│   ├── user/
│   │   ├── user.go
│   │   ├── user_test.go
│   │   ├── repository.go
│   │   └── service.go
│   └── database/
│       └── postgres.go
├── pkg/                    # Public libraries (optional)
│   └── utils/
├── api/                    # API definitions (OpenAPI, protobuf)
├── web/                    # Web assets (if applicable)
├── scripts/                # Build and automation scripts
├── deployments/            # Deployment configs (Docker, k8s)
├── test/                   # Additional test data
├── go.mod
├── go.sum
├── Makefile                # Common tasks
└── README.md
```

### Package Organisation

```go
// internal/user/user.go - Domain types
package user

type User struct {
    ID    uuid.UUID
    Email string
}

// internal/user/repository.go - Data access
package user

type Repository interface {
    Create(ctx context.Context, user *User) error
    FindByID(ctx context.Context, id uuid.UUID) (*User, error)
}

// internal/user/service.go - Business logic
package user

type Service struct {
    repo Repository
}

func NewService(repo Repository) *Service {
    return &Service{repo: repo}
}
```

## Dependency Management

### Go Modules Best Practices

```bash
# Initialise module
go mod init github.com/username/project

# Add dependencies (automatically)
go get github.com/google/uuid@latest

# Tidy up (remove unused, add missing)
go mod tidy

# Verify integrity
go mod verify

# Vendor dependencies (optional)
go mod vendor
```

### Version Pinning

```go
// go.mod with specific versions
module github.com/username/project

go 1.21

require (
    github.com/google/uuid v1.5.0
    github.com/lib/pq v1.10.9
)

require (
    // Indirect dependencies
    golang.org/x/crypto v0.17.0 // indirect
)
```

## Concurrency

Use goroutines with context cancellation, `sync.WaitGroup` for coordination, `sync.Once` for singletons, and `sync.Pool` for buffer reuse. See [`references/concurrency.md`](references/concurrency.md) for full patterns.

## Performance

Preallocate slices when size is known, use `strings.Builder` for concatenation, and profile with `pprof`. See [`references/performance.md`](references/performance.md).

## Linting and Quality

Use `golangci-lint` with the recommended linter set. See [`references/linting.md`](references/linting.md) for the full `.golangci.yml` configuration.

## Common Patterns

Constructor pattern, functional options, interface segregation, and middleware chaining. See [`references/patterns.md`](references/patterns.md).

## Anti-Patterns

Avoid: panicking in library code, ignoring errors, naked returns in long functions, global mutable state. See [`references/patterns.md`](references/patterns.md) for annotated examples.

## Build, Tools and Deployment

Makefile targets, build tags for integration tests, essential tools (`goimports`, `golangci-lint`, `air`), and VS Code settings. See [`references/build-tools.md`](references/build-tools.md).

## Security Best Practices

```go
// Use crypto/rand for random values
import "crypto/rand"

func generateToken() (string, error) {
    b := make([]byte, 32)
    if _, err := rand.Read(b); err != nil {
        return "", err
    }
    return base64.URLEncoding.EncodeToString(b), nil
}

// Validate and sanitise inputs
func sanitiseUserInput(input string) string {
    return html.EscapeString(strings.TrimSpace(input))
}

// Use prepared statements for SQL
stmt, err := db.PrepareContext(ctx, "SELECT * FROM users WHERE id = $1")
defer stmt.Close()

// Set timeouts on contexts
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()
```

## Alignment with Core Principles

- **SOLID Principles**: Single responsibility through package design, dependency inversion via interfaces
- **DRY**: Avoid duplication through proper abstraction and generics
- **YAGNI**: Build what you need now, Go's simplicity encourages this
- **KISS**: Go's design philosophy emphasises simplicity
- **Test Pyramid**: 70% unit (table-driven), 20% integration (with build tags), 10% e2e

## References

### Bundled Reference Files

- [`references/concurrency.md`](references/concurrency.md) — Goroutines, channels, sync primitives
- [`references/performance.md`](references/performance.md) — Memory optimisation, profiling
- [`references/linting.md`](references/linting.md) — Full `.golangci.yml` configuration
- [`references/patterns.md`](references/patterns.md) — Constructor, options, middleware, anti-patterns
- [`references/build-tools.md`](references/build-tools.md) — Makefile, build tags, essential tools, IDE config

### External Links

- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Standard Go Project Layout](https://github.com/golang-standards/project-layout)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
