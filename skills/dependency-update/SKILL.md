---
name: dependency-update
description: Safe dependency update procedure with audit, update, testing, and verification of no breakage
---

## What This Skill Does

Defines a safe process for updating Go module dependencies. Covers auditing what needs updating, performing updates, verifying nothing breaks, and handling breaking changes.

## When To Use

Use this when:
- Routine dependency maintenance is needed
- A security vulnerability (CVE) requires updating a specific dependency
- A new feature requires a newer version of an existing dependency
- `govulncheck` or Dependabot reports vulnerabilities

## Workflow

### Phase 1: Audit
**Agent**: builder

1. Run `go list -m -u all` to see available updates
2. Run `govulncheck ./...` to identify known vulnerabilities
3. Categorize updates:
   - **Security**: CVE fixes — highest priority
   - **Major**: Breaking version changes (v1 → v2) — require careful handling
   - **Minor**: New features, non-breaking — moderate risk
   - **Patch**: Bug fixes — lowest risk
4. Prioritize: security fixes first, then patches, then minors, then majors

### Phase 2: Update

Handle each category differently:

#### Patch Updates (lowest risk)
```bash
go get -u=patch ./...
go mod tidy
```
- Run tests immediately: `go test -race ./...`
- These should be safe — if tests break, investigate before proceeding

#### Minor Updates (moderate risk)
- Update one dependency at a time: `go get <module>@latest`
- Run `go mod tidy`
- Run tests after each update
- Check the dependency's changelog for behavior changes
- **Research**: Fetch the dependency's release notes to understand what changed

#### Major Updates (high risk)
- Read the migration guide for the new major version
- Update the import path (Go modules require `/v2`, `/v3`, etc.)
- Update API usage to match the new version
- Run tests and fix compilation errors
- **Research**: Fetch the dependency's documentation for the new major version. Do not guess at API changes

#### Security Updates (urgent)
- Update the specific vulnerable dependency immediately
- If the fix requires a major version bump, assess whether the vulnerability is exploitable in your usage
- If exploitable, prioritize the major update; if not, document the risk and plan the major update separately

### Phase 3: Verify

1. `go build ./...` — must compile
2. `go vet ./...` — must pass
3. `go test -race ./...` — all tests must pass
4. `govulncheck ./...` — verify the vulnerability is resolved (if security update)
5. If the project has integration tests, run those too
6. Run the linter: `golangci-lint run` — dependency updates can trigger new lint findings

### Phase 4: Review and Commit
**Agents**: code-reviewer, git-workflow

1. Review the `go.mod` and `go.sum` diff
2. Verify no unexpected transitive dependencies were added
3. Check for any `replace` directives that may need updating
4. Commit with descriptive message:
   - `chore(deps): update <dependency> to <version>` for routine updates
   - `security(deps): update <dependency> to <version> (CVE-XXXX-XXXXX)` for security fixes

## Principles

- **One at a time for risky updates**: Update patch versions in bulk, but update minor and major versions individually so you can isolate breakage
- **Read the changelog**: Don't just bump the version — understand what changed
- **Test after every update**: Don't batch updates and test once. Test after each change
- **Research before updating**: For major versions, fetch the migration guide. For new libraries, fetch the docs. Do not guess at API changes
- **Don't mix dependency updates with feature work**: Dependency updates should be their own commits/PRs
