---
name: code-health-check
description: Comprehensive codebase health audit covering lint, vet, test coverage, dead code, dependency vulnerabilities, and structural issues
---

## What This Skill Does

Performs a comprehensive health check on a Go codebase. Identifies technical debt, security risks, test gaps, and code quality issues. Produces a prioritized report of findings.

## When To Use

Use this when:
- Onboarding to a new codebase and need to understand its health
- Performing periodic maintenance / tech debt assessment
- Preparing for a major feature and want to ensure the foundation is solid
- After a series of rapid changes and need to assess accumulated debt

## Audit Procedure

### 1. Compilation and Static Analysis
**Agent**: builder

```bash
go build ./...          # Must compile cleanly
go vet ./...            # Must pass
golangci-lint run       # Document all findings
```

Record:
- Number of lint findings by category
- Any compilation warnings
- Any `go vet` issues

### 2. Test Coverage
**Agent**: test-writer

```bash
go test -race -coverprofile=coverage.out ./...
go tool cover -func=coverage.out
```

Assess:
- Overall coverage percentage
- Packages with 0% coverage (these are the highest risk)
- Critical packages (auth, data access, business logic) with low coverage
- Whether tests actually assert behavior or are just "coverage padding"

### 3. Dependency Health
**Agent**: builder

```bash
govulncheck ./...                    # Known vulnerabilities
go list -m -u all                    # Available updates
go mod tidy                          # Check for unused deps (look at diff)
```

Assess:
- Number of known vulnerabilities and their severity
- How far behind dependencies are (months/versions)
- Whether `go.sum` has unnecessary entries
- Any `replace` directives that should be temporary

### 4. Code Quality Review
**Agent**: code-reviewer

Scan for:
- **Dead code**: Exported functions/types with no callers
- **God packages**: Packages with too many responsibilities or too many files
- **Circular dependencies**: `go vet` catches some, but review package import graphs manually
- **Inconsistent patterns**: Multiple ways of doing the same thing (error handling, logging, config access)
- **TODO/FIXME/HACK comments**: Count and categorize — are any critical?
- **Deprecated API usage**: stdlib or dependency functions marked deprecated

### 5. Security Scan
**Agent**: security-auditor

- Review for hardcoded secrets
- Check HTTP client timeout configuration
- Verify input validation on all public-facing handlers
- Check for proper TLS configuration
- Review authentication and authorization consistency

### 6. Concurrency Review (if applicable)
**Agent**: concurrency-reviewer

- Identify all goroutine launch sites
- Verify each has a clear shutdown path
- Check for mutex usage patterns
- Recommend `-race` flag if not already in CI

### 7. Structural Quality Review
**Agent**: refactorer

Assess refactoring opportunities:
- **God packages**: Packages with too many files, types, or responsibilities that should be split
- **Dependency direction**: Does domain logic import infrastructure, or is the dependency direction clean?
- **Dead abstractions**: Interfaces with a single implementation that will never have another, unnecessary wrapper types, over-engineered option patterns
- **Inconsistent patterns**: Multiple approaches to the same problem (logging, error handling, config access, HTTP response writing)
- **Deprecated patterns**: Old Go idioms that have cleaner modern equivalents (manual loops vs. `slices` package, `interface{}` vs. `any`, `log` vs. `slog`)
- **Duplication**: Repeated code blocks that should be extracted into shared functions

Produce a prioritized list of refactoring opportunities with effort estimates (small/medium/large) and risk assessments.

### 8. Infrastructure Review
**Agent**: ci-ops

- Is CI running tests on every PR?
- Is the linter running in CI?
- Is the Docker image using multi-stage builds?
- Are GitHub Actions pinned to SHAs?
- Is test coverage tracked over time?

### 9. Compile Report
**Agent**: code-reviewer

Consolidate findings from all previous steps into a single prioritized report:

```markdown
## Codebase Health Report

### Summary
- **Overall health**: [Good / Needs Attention / Critical]
- **Test coverage**: X%
- **Known vulnerabilities**: N (X critical, Y high)
- **Lint findings**: N
- **Outdated dependencies**: N

### Critical Issues (fix immediately)
1. ...

### High Priority (fix before next release)
1. ...

### Medium Priority (address in next sprint)
1. ...

### Low Priority (tech debt backlog)
1. ...

### Positive Observations
- Things the codebase does well (acknowledge good patterns)
```

**Gate**: Present the compiled report to the user. Options: "Accept report", "Investigate specific findings further", "Create action items from findings".

## Principles

- **Be objective**: Report what you find, not what you expect. Some codebases are healthier than they look; some are worse
- **Prioritize by risk**: A security vulnerability matters more than a lint nit. Order findings by actual impact
- **Acknowledge good patterns**: If the codebase does something well, say so. Context about what's working helps as much as what's broken
- **Actionable findings only**: Every finding should have a clear remediation. "This is bad" without "here's how to fix it" is not useful
