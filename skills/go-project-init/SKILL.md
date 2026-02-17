---
name: go-project-init
description: Standard Go project scaffolding with directory layout, go.mod, Makefile, Dockerfile, CI pipeline, and linter configuration
---

## What This Skill Does

Guides scaffolding a new Go project with a production-ready structure. Covers directory layout, build tooling, containerization, CI, and development environment setup.

## When To Use

Use this when starting a new Go project or service from scratch. Do not use this to restructure an existing project — use the refactorer agent for that.

## Pre-Scaffold Checklist

Before creating any files:
1. Confirm the Go module path (e.g., `github.com/org/project`)
2. Confirm the project type: CLI tool, HTTP service, gRPC service, or library
3. Confirm the minimum Go version
4. Check if the organization has existing projects to match conventions from

## Directory Layout

```
.
├── cmd/
│   └── <app>/
│       └── main.go           # Entry point, minimal — just wires things up
├── internal/                  # Private application code
│   ├── config/                # Configuration loading
│   ├── server/                # HTTP/gRPC server setup (if service)
│   └── <domain>/              # Domain-specific packages
├── pkg/                       # Public library code (only if intentionally shared)
├── migrations/                # SQL migration files (if applicable)
├── api/                       # OpenAPI specs, .proto files (if applicable)
├── scripts/                   # Build and dev helper scripts
├── Dockerfile
├── Makefile
├── docker-compose.yml         # Local development dependencies
├── .golangci.yml
├── .gitignore
├── .env.example
├── README.md
└── go.mod
```

### Layout Decisions
- Use `internal/` by default — only move to `pkg/` if code is explicitly meant for external import
- One `cmd/<name>/main.go` per binary — keep main.go thin (parse config, wire deps, start server)
- Group by domain, not by technical layer (prefer `internal/user/` over `internal/handlers/`, `internal/models/`)

## go.mod

```
go mod init <module-path>
```
- Set the Go version to the project's minimum supported version
- Do not add dependencies until they are actually needed

## Makefile

Provide these standard targets:
```makefile
.PHONY: build test lint run clean docker-build help

BINARY := <app>
VERSION := $(shell git describe --tags --always --dirty 2>/dev/null || echo "dev")
LDFLAGS := -ldflags "-X main.version=$(VERSION)"

help: ## Show this help
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  %-15s %s\n", $$1, $$2}'

build: ## Build the binary
	go build $(LDFLAGS) -o bin/$(BINARY) ./cmd/$(BINARY)

test: ## Run tests
	go test -race ./...

lint: ## Run linter
	golangci-lint run

run: build ## Build and run
	./bin/$(BINARY)

clean: ## Remove build artifacts
	rm -rf bin/

docker-build: ## Build Docker image
	docker build -t $(BINARY):$(VERSION) .
```

## Dockerfile

Use multi-stage build:
```dockerfile
FROM golang:<version>-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app/server ./cmd/<app>

FROM gcr.io/distroless/static-debian12
COPY --from=builder /app/server /server
ENTRYPOINT ["/server"]
```

**Important**: Research the current Go version and distroless image tags before writing the Dockerfile. Do not use versions from memory.

## .golangci.yml

Start with a minimal, opinionated configuration:
```yaml
run:
  timeout: 5m

linters:
  enable:
    - errcheck
    - govet
    - staticcheck
    - unused
    - gosimple
    - ineffassign
    - typecheck
    - revive
    - gocritic
    - gofumpt

linters-settings:
  revive:
    rules:
      - name: exported
        arguments:
          - checkPrivateReceivers
  gocritic:
    enabled-tags:
      - diagnostic
      - style
      - performance
```

**Important**: Research the current golangci-lint configuration format before writing this file. Linter names and settings change between versions.

## .gitignore

```
bin/
*.exe
.env
coverage.out
vendor/
```

## CI Pipeline (GitHub Actions)

Create `.github/workflows/ci.yml`:
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<current-sha>
      - uses: actions/setup-go@<current-sha>
        with:
          go-version-file: go.mod
      - run: go test -race ./...

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<current-sha>
      - uses: actions/setup-go@<current-sha>
        with:
          go-version-file: go.mod
      - uses: golangci/golangci-lint-action@<current-sha>
```

**Important**: Research the current SHA-pinned versions of all GitHub Actions before writing the workflow. Do not use versions from memory.

## Post-Scaffold Verification

After scaffolding:
1. `go build ./...` — must compile
2. `go vet ./...` — must pass
3. `go test ./...` — must pass (even if no tests yet)
4. `make help` — must list all targets
5. `docker build .` — must build successfully (if Dockerfile created)

## Agent Coordination

- **builder**: Executes the scaffolding
- **ci-ops**: Should review the Dockerfile and CI pipeline
- **docs-writer**: Should write the initial README
- **code-reviewer**: Should review the final scaffold for consistency
