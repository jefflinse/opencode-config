---
name: go-project-init
description: Standard Go project scaffolding with directory layout, go.mod, Makefile, Dockerfile, CI pipeline, and linter configuration
---

## What This Skill Does

Guides scaffolding a new Go project with a production-ready structure. Covers directory layout, build tooling, containerization, CI, and development environment setup.

## When To Use

Use this when starting a new Go project or service from scratch. Do not use this to restructure an existing project — use the refactorer agent for that.

## Workflow

### Phase 1: Gather Requirements
**Agent**: planner

Collect the information needed to scaffold correctly:
1. Go module path (e.g., `github.com/org/project`)
2. Project type: CLI tool, HTTP service, gRPC service, or library
3. Minimum Go version
4. Whether the organization has existing projects with conventions to match
5. Required infrastructure: database, cache, message queue, etc.
6. CI platform: GitHub Actions, GitLab CI, or other

**Gate**: Requirements must be confirmed by the user before scaffolding begins. Use the question tool to collect any missing information.

### Phase 2: Scaffold Project Structure
**Agent**: builder

Create the directory layout and core files based on the requirements from Phase 1:

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
├── .gitignore
├── .env.example
└── go.mod
```

Layout decisions:
- Use `internal/` by default — only create `pkg/` if code is explicitly meant for external import
- One `cmd/<name>/main.go` per binary — keep main.go thin (parse config, wire deps, start server)
- Group by domain, not by technical layer (prefer `internal/user/` over `internal/handlers/`, `internal/models/`)

Acceptance criteria: `go build ./...` compiles, `go vet ./...` passes.

### Phase 3: Build Tooling
**Agent**: ci-ops

Create build and development tooling:

1. **Makefile** with standard targets: `build`, `test`, `lint`, `run`, `clean`, `docker-build`, `help`
2. **Dockerfile** using multi-stage build (research current Go and distroless image tags before writing)
3. **docker-compose.yml** for local development dependencies (database, cache, etc.)
4. **.golangci.yml** with a minimal, opinionated linter configuration

**Important**: Research current versions of Go base images, distroless images, and golangci-lint configuration format before writing these files. Do not use versions from memory.

Acceptance criteria: `make help` lists all targets, `make build` succeeds, `docker build .` succeeds.

### Phase 4: CI Pipeline
**Agent**: ci-ops

Create the CI pipeline for the project's CI platform:

1. **GitHub Actions** workflow (`.github/workflows/ci.yml`) with:
   - Test job: `go test -race ./...`
   - Lint job: `golangci-lint run`
   - Pin all action versions to specific SHAs
   - Use `go-version-file: go.mod` for Go version

**Important**: Research current SHA-pinned versions of all GitHub Actions before writing the workflow. Do not use versions from memory.

Acceptance criteria: Workflow YAML is valid syntax, references valid action SHAs.

### Phase 5: Documentation
**Agent**: docs-writer

Create initial project documentation:

1. **README.md** with:
   - One-line project description
   - Installation/setup instructions
   - Quick-start example
   - Development setup (prerequisites, `make` targets)
   - Configuration options

Acceptance criteria: README accurately reflects the scaffolded project structure.

### Phase 6: Review
**Agent**: code-reviewer

Review the complete scaffold for:
1. Consistency between all generated files (Makefile targets match Dockerfile stages, CI matches Makefile)
2. No hardcoded secrets, placeholder values left in committed files
3. All files follow the conventions established in Phase 1
4. `go build ./...`, `go vet ./...`, `go test ./...` all pass

**Gate**: Present the review findings to the user. The scaffold should be clean before the user starts building on it.

## Reference: File Templates

The following sections provide templates for the builder and ci-ops agents. These are reference material — agents should adapt them to the specific project requirements, not copy them verbatim.

### Makefile Template
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

### Dockerfile Template
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

### .gitignore Template
```
bin/
*.exe
.env
coverage.out
vendor/
```
