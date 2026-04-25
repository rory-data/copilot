# Go Linting Configuration

## golangci-lint Configuration

```yaml
# .golangci.yml
run:
  timeout: 5m
  go: "1.21"

linters:
  enable:
    - errcheck # Check error handling
    - gosimple # Simplify code
    - govet # Examine Go source code
    - ineffassign # Detect ineffectual assignments
    - staticcheck # Advanced static analysis
    - unused # Find unused code
    - gofmt # Format checking
    - goimports # Import management
    - misspell # Spell checking
    - revive # Fast, configurable linter
    - gocritic # Comprehensive diagnostics
    - gosec # Security issues
    - errname # Error naming conventions
    - errorlint # Error wrapping
    - exhaustive # Enum switch exhaustiveness

linters-settings:
  errcheck:
    check-blank: true
  govet:
    check-shadowing: true
  gofmt:
    simplify: true
  revive:
    rules:
      - name: unexported-return
        disabled: true
```
