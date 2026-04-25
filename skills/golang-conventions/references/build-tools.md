# Go Build, Tools and Deployment

## Makefile

```makefile
.PHONY: build test lint clean

# Build binary
build:
	go build -o bin/server cmd/server/main.go

# Run tests
test:
	go test -v -race -coverprofile=coverage.out ./...

# Run linter
lint:
	golangci-lint run ./...

# Format code
fmt:
	gofmt -s -w .
	goimports -w .

# Clean build artifacts
clean:
	rm -rf bin/ coverage.out

# Install tools
tools:
	go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
	go install golang.org/x/tools/cmd/goimports@latest
```

## Build Tags

```go
//go:build integration
// +build integration

package user_test

// Integration tests only run with: go test -tags=integration
```

## Essential Tools

```bash
# Format and imports
go install golang.org/x/tools/cmd/goimports@latest

# Linting
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# Testing utilities
go install github.com/stretchr/testify@latest

# Code generation (if needed)
go install github.com/golang/mock/mockgen@latest

# Hot reload for development
go install github.com/cosmtrek/air@latest
```

## IDE Configuration (VS Code)

```json
{
  "go.useLanguageServer": true,
  "go.lintTool": "golangci-lint",
  "go.lintOnSave": "package",
  "go.formatTool": "goimports",
  "editor.formatOnSave": true,
  "go.testFlags": ["-v", "-race"],
  "go.coverOnSave": true
}
```
