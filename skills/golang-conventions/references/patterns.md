# Go Patterns and Anti-Patterns

## Common Patterns

### Constructor Pattern

```go
// NewService creates a Service with required dependencies.
func NewService(repo Repository, logger *slog.Logger) *Service {
    return &Service{
        repo:   repo,
        logger: logger,
    }
}

// With options pattern for complex constructors
type Option func(*Service)

func WithCache(cache Cache) Option {
    return func(s *Service) {
        s.cache = cache
    }
}

func NewServiceWithOptions(repo Repository, opts ...Option) *Service {
    s := &Service{repo: repo}
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

### Interface Segregation

```go
// Good: Small, focused interfaces
type Reader interface {
    Read(ctx context.Context, id string) (*Data, error)
}

type Writer interface {
    Write(ctx context.Context, data *Data) error
}

type ReadWriter interface {
    Reader
    Writer
}

// Accept interfaces, return structs
func ProcessData(r Reader, id string) (*Result, error) {
    data, err := r.Read(context.Background(), id)
    // ...
}
```

### Middleware Pattern

```go
type HandlerFunc func(http.ResponseWriter, *http.Request)

type Middleware func(HandlerFunc) HandlerFunc

func LoggingMiddleware(logger *slog.Logger) Middleware {
    return func(next HandlerFunc) HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            next(w, r)
            logger.Info("request",
                "method", r.Method,
                "path", r.URL.Path,
                "duration", time.Since(start),
            )
        }
    }
}

// Chain middlewares
func Chain(h HandlerFunc, middlewares ...Middleware) HandlerFunc {
    for i := len(middlewares) - 1; i >= 0; i-- {
        h = middlewares[i](h)
    }
    return h
}
```

## Anti-Patterns to Avoid

❌ **Don't do this:**

```go
// Panic in library code (use errors)
func GetUser(id string) *User {
    user := findUser(id)
    if user == nil {
        panic("user not found")
    }
    return user
}

// Ignoring errors
result, _ := doSomething()

// Naked returns in long functions
func calculate(a, b int) (result int) {
    // ... 50 lines of code ...
    return
}

// Global mutable state
var users = make(map[string]*User)
```

✅ **Do this instead:**

```go
// Return errors explicitly
func GetUser(id string) (*User, error) {
    user := findUser(id)
    if user == nil {
        return nil, ErrUserNotFound
    }
    return user, nil
}

// Handle all errors
result, err := doSomething()
if err != nil {
    return fmt.Errorf("operation failed: %w", err)
}

// Explicit returns
func calculate(a, b int) int {
    result := a + b
    return result
}

// Dependency injection
type UserService struct {
    users map[string]*User
}
```
