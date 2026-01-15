---
applyTo: "Dockerfile"
description: "Essential Docker best practices for optimised, secure, and efficient container images"
---

# Docker Best Practices

## Quick Reference

- **Multi-stage builds**: Separate build and runtime dependencies
- **Alpine/slim images**: Minimal base images for smaller attack surface
- **Layer caching**: Order instructions from least to most frequently changing
- **Security first**: Non-root user, pinned versions, minimal packages
- **Single process**: One primary process per container

## Core Principles

### 1. Immutability

- Never modify running containers - create new images instead
- Use semantic versioning for image tags (`v1.2.3`)
- Treat images as versioned artifacts

### 2. Efficiency & Security

- Prefer Alpine variants for smaller images (`node:18-alpine`)
- Use official images from trusted sources
- Update base images regularly for security patches
- Avoid `latest` tag in production

### 3. Layer Optimisation

```dockerfile
# GOOD: Optimise for caching
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./        # Cache-friendly: deps change less
RUN npm ci --only=production
COPY . .                     # App code changes most
```

## Multi-Stage Builds (Essential)

```dockerfile
# Build stage
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/package*.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

## Security Essentials

### Non-Root User

```dockerfile
# Create and use non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs
```

### Minimal Dependencies

```dockerfile
# Combine commands to reduce layers and clean up
RUN apk add --no-cache \
    python3 \
    py3-pip \
    && pip install --no-cache-dir flask \
    && apk del build-dependencies
```

### Version Pinning

```dockerfile
# Pin specific versions for reproducibility
FROM python:3.11.5-slim
RUN pip install flask==2.3.3
```

## Common Patterns

### Python Application

```dockerfile
FROM python:3.11-slim AS build
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM python:3.11-slim AS production
WORKDIR /app
COPY --from=build /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY . .
USER nobody
EXPOSE 8000
CMD ["python", "app.py"]
```

### Node.js Application

```dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS production
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
USER node
EXPOSE 3000
CMD ["node", "server.js"]
```

### Go Application

```dockerfile
FROM golang:1.21-alpine AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o main .

FROM scratch AS production
COPY --from=build /app/main /main
EXPOSE 8080
CMD ["/main"]
```

## Performance Optimisation

### .dockerignore

```dockerignore
node_modules
.git
.gitignore
README.md
.env
.nyc_output
coverage
.vscode
```

### Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

### Resource Limits (docker-compose)

```yaml
services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "0.5"
```

## Security Checklist

- [ ] Use multi-stage builds
- [ ] Run as non-root user
- [ ] Use minimal base images (Alpine/slim)
- [ ] Pin dependency versions
- [ ] Remove unnecessary packages
- [ ] Scan images for vulnerabilities
- [ ] Set resource limits
- [ ] Use secrets management for sensitive data

## Common Anti-Patterns to Avoid

❌ **Don't do this:**

```dockerfile
FROM ubuntu:latest                    # Use specific versions
RUN apt-get update                   # Combine with install
COPY . .                            # Do this after deps
RUN apt-get install -y curl         # Separate command
ADD https://example.com/file.tar.gz  # Use COPY + RUN
```

✅ **Do this instead:**

```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

## Development vs Production

### Development

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]
```

### Production (Multi-stage)

```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app
COPY --from=build /app/dist ./dist
RUN addgroup -g 1001 nodejs && adduser -S -u 1001 nextjs
USER nextjs
EXPOSE 3000
CMD ["node", "dist/server.js"]
```
