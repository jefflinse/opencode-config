---
name: pr-review
description: Structured PR review workflow with full diff analysis, test verification, specialist agent coordination, and compiled findings
---

## What This Skill Does

Defines a thorough, structured process for reviewing a pull request. Coordinates multiple reviewer agents and compiles findings into a single, actionable review.

## When To Use

Use this when:
- A PR is ready for review
- You want a comprehensive review beyond what a single pass provides
- The PR touches multiple concerns (API, database, concurrency, security)

## Review Workflow

### Phase 1: Understand the Change

Before reviewing any code:
1. Read the PR description — understand *what* changed and *why*
2. Read the linked issue or ticket for context
3. Identify the scope: which packages, files, and systems are affected
4. Determine the type of change: feature, bug fix, refactoring, dependency update, infrastructure

### Phase 2: Full Diff Review
**Agent**: code-reviewer

Review the entire diff, not just individual files:
1. Read through all changed files to understand the full picture
2. Check that the change is complete — no half-implemented features or TODO comments that should have been resolved
3. Verify the change is minimal — no unrelated reformatting, refactoring, or "while I was here" changes mixed in
4. Check for consistency with existing project conventions

Focus areas:
- Correctness: logic errors, nil safety, error handling
- Completeness: are all code paths handled, including errors?
- Naming: clear, consistent, following project conventions
- API surface: are new exports intentional and minimal?
- Resource management: are files, connections, and goroutines properly cleaned up?

### Phase 3: Test Verification

1. Are there tests for the new/changed behavior?
2. Do the tests cover edge cases and error paths, not just the happy path?
3. Run the tests: `go test -race ./...`
4. Check test quality — do they actually assert correct behavior, or are they "coverage-only" tests?
5. For bug fixes: is there a regression test that would have caught the original bug?

### Phase 4: Specialist Reviews

Engage specialist agents based on what the PR touches:

| PR Contains | Agent | What They Check |
|-------------|-------|-----------------|
| Goroutines, channels, mutexes, shared state | concurrency-reviewer | Data races, deadlocks, goroutine leaks |
| User input handling, auth, crypto, HTTP headers | security-auditor | Injection, auth bypass, secret exposure |
| Database queries, schema changes, migrations | db-architect | Query safety, migration compatibility, N+1 queries |
| New/modified API endpoints | api-designer | Contract consistency, backward compatibility |
| Dockerfile, CI config, Makefile changes | ci-ops | Build correctness, image security, pipeline logic |

Only engage specialists relevant to the specific PR. Don't run every agent on every PR.

### Phase 5: AI-Code Awareness

Since the code was likely AI-generated, apply extra scrutiny for:
- **Hallucinated APIs**: Functions or methods that don't exist in the referenced library version
- **Plausible but wrong logic**: Code that reads naturally but has subtle errors
- **Cargo-culted patterns**: Abstractions or patterns that add complexity without value
- **Misleading test names**: Test cases where the name describes one thing but the test verifies another
- **Missing edge cases**: Happy path coverage without error path coverage

### Phase 6: Compile Findings

Consolidate all findings into a structured review:

```markdown
## Review Summary

**Verdict**: [Approve / Request Changes / Needs Discussion]

### Blocking Issues
Issues that must be resolved before merge:
1. [BLOCKING] <file:line> — <description> — <suggested fix>

### Suggestions
Recommended improvements:
1. [SUGGESTION] <file:line> — <description> — <suggested fix>

### Nits
Minor style preferences:
1. [NIT] <file:line> — <description>

### Specialist Findings
- **concurrency-reviewer**: <summary or "not applicable">
- **security-auditor**: <summary or "not applicable">
- **db-architect**: <summary or "not applicable">

### Questions
Things that need clarification from the author:
1. <question>

### Positive Feedback
Things done well (acknowledge good work):
1. <observation>
```

## Principles

- **Review the whole thing**: Don't nitpick the first file and rubber-stamp the rest. Read every changed line
- **Blocking vs. suggestion**: Be clear about what must change vs. what would be nice to change. Don't block a PR over style preferences
- **Explain why**: For every finding, explain *why* it's a problem, not just *what* to change
- **Be constructive**: Provide fixes, not just complaints. Show the better code
- **Acknowledge good work**: If something is well done, say so. Reviews that only contain criticism are demoralizing and incomplete
- **One review, all findings**: Don't trickle findings across multiple review rounds. Compile everything into one comprehensive review
