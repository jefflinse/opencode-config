---
name: feature-workflow
description: End-to-end feature implementation workflow from planning through branch, build, test, review, and PR creation
---

## What This Skill Does

Defines the complete workflow for implementing a feature, from understanding the requirement through to a merge-ready pull request. Coordinates multiple agents through the process.

## When To Use

Use this when implementing a non-trivial feature that involves multiple files, new packages, or changes to public APIs. For single-file bug fixes, use the bug-triage skill instead.

## Workflow

### Phase 1: Plan
**Agent**: planner

1. Clarify the requirement — what exactly are we building and why?
2. Identify scope boundaries — what is in and what is out
3. Analyze the existing codebase to understand where changes belong
4. Decompose into ordered subtasks with acceptance criteria
5. Identify which agents need to be involved and when
6. Flag any cross-cutting concerns (auth, concurrency, schema changes, API changes)

**Gate**: Plan must be reviewed and approved before proceeding.

### Phase 2: Branch
**Agent**: git-workflow

1. Create a feature branch from main: `feat/<description>`
2. Use kebab-case, descriptive but concise

### Phase 3: Design (if applicable)
**Agents**: api-designer, db-architect

The planner's output from Phase 1 determines whether this phase is needed. Check the plan for:
- If the plan includes a step assigned to **api-designer** → invoke api-designer to produce the API specification
- If the plan includes a step assigned to **db-architect** → invoke db-architect to produce the migration
- If the plan does not reference either agent → skip this phase entirely

**Gate**: If this phase was executed, the API spec and/or migration must be reviewed and approved before proceeding to implementation.

### Phase 4: Build
**Agent**: builder

1. Implement the feature following the plan's subtask ordering
2. Follow existing project conventions — do not introduce new patterns
3. Research any unfamiliar libraries or APIs before using them
4. Handle all error paths
5. Run `go build ./...` and `go vet ./...` after each significant change

### Phase 5: Test
**Agent**: test-writer

1. Write tests for all new exported functions and methods
2. Cover happy path, error cases, and edge cases
3. Add integration tests if the feature touches external dependencies
4. Run `go test -race ./...` and verify all tests pass

### Phase 6: Review
**Agents**: code-reviewer + specialists as needed

1. code-reviewer reviews the full changeset
2. If the feature touches concurrency → concurrency-reviewer
3. If the feature handles user input or auth → security-auditor
4. If the feature includes database changes → db-architect reviews the migration
5. Address all BLOCKING findings before proceeding

**Gate**: No BLOCKING findings remaining.

### Phase 6b: QA (if web UI)
**Agent**: qa

This phase applies only if the feature includes browser-facing UI changes and a running instance is available. If the feature is purely backend, CLI, or library work, skip this phase.

1. Invoke the qa agent with the application URL, a description of what was built, and the key user flows to test
2. QA agent performs smoke testing and regression checks through the browser
3. QA agent writes Playwright end-to-end tests (`e2e/*.spec.ts`) for the new feature's key user flows
4. Address any BLOCKING findings before proceeding (same as code review findings)

**Gate**: No BLOCKING QA findings remaining.

### Phase 7: Polish
**Agents**: builder (fixes), refactorer (cleanup), docs-writer (documentation)

1. Address review feedback
2. If the code-reviewer flagged structural issues (duplication, poor abstraction, inconsistent patterns), involve the refactorer to clean up — but only after the feature is functionally correct and tests pass
3. Update documentation if the feature changes public behavior
4. Update README if the feature adds new configuration or usage patterns
5. Run the full test suite one final time

**Note**: Refactoring during polish is optional and should be scoped tightly. The goal is to clean up the feature's code, not to refactor the entire codebase. If broader refactoring is warranted, create a follow-up issue.

### Phase 8: PR
**Agent**: git-workflow

1. Squash WIP commits into logical units
2. Write commit messages following conventional commit format
3. Create PR with structured description:
   - Summary of what and why
   - List of key changes
   - Testing approach
   - Breaking changes (if any)
   - Link to relevant issue/ticket

## Principles

- **Plan before building**: The cheapest time to find a design flaw is before any code is written
- **Research before implementing**: Verify libraries, APIs, and patterns are current before using them
- **Review before merging**: Every feature gets at least a code-reviewer pass; specialist reviews for relevant concerns
- **Test as you go**: Don't leave all testing to the end — the builder should verify compilation and basic correctness continuously
- **Small, reviewable chunks**: If the feature is too large for one PR, break it into multiple PRs that each leave the codebase in a working state
