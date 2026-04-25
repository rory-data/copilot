# Go Concurrency Patterns

## Goroutines and Channels

```go
// Use context for cancellation
func processItems(ctx context.Context, items []Item) error {
    results := make(chan Result, len(items))
    errors := make(chan error, 1)

    for _, item := range items {
        go func(item Item) {
            select {
            case <-ctx.Done():
                return
            case results <- processItem(item):
            }
        }(item)
    }

    // Collect results
    for i := 0; i < len(items); i++ {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case err := <-errors:
            return err
        case result := <-results:
            // Handle result
        }
    }

    return nil
}
```

## sync Package Patterns

```go
import "sync"

// Use sync.WaitGroup for coordinating goroutines
var wg sync.WaitGroup

for _, item := range items {
    wg.Add(1)
    go func(item Item) {
        defer wg.Done()
        process(item)
    }(item)
}

wg.Wait()

// Use sync.Once for one-time initialisation
var (
    instance *Service
    once     sync.Once
)

func GetService() *Service {
    once.Do(func() {
        instance = newService()
    })
    return instance
}

// Use sync.Pool for frequently allocated objects
var bufferPool = sync.Pool{
    New: func() any {
        return new(bytes.Buffer)
    },
}

func processData(data []byte) {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer bufferPool.Put(buf)

    buf.Reset()
    buf.Write(data)
    // Process buffer
}
```
