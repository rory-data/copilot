# Go Performance Best Practices

## Memory Optimisation

```go
// Preallocate slices when size is known
items := make([]Item, 0, expectedCount)

// Use pointers for large structs in maps
type Cache map[string]*LargeStruct

// Reuse buffers
var buf bytes.Buffer
buf.Reset() // Clear for reuse

// Use string builder for concatenation
var sb strings.Builder
for _, s := range strings {
    sb.WriteString(s)
}
result := sb.String()
```

## Profiling

```go
import (
    "runtime/pprof"
    "net/http"
    _ "net/http/pprof"
)

// CPU profiling
f, _ := os.Create("cpu.prof")
pprof.StartCPUProfile(f)
defer pprof.StopCPUProfile()

// Memory profiling
f, _ := os.Create("mem.prof")
pprof.WriteHeapProfile(f)

// HTTP profiling endpoint
go func() {
    http.ListenAndServe("localhost:6060", nil)
}()
```
