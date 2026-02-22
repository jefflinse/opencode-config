---
name: rapid-prototype
description: Streamlined prototyping workflow that prioritizes speed and iteration over thoroughness, skipping formal review, testing, and polish phases
---

## What This Skill Does

Defines a fast-track workflow for building prototypes, proofs of concept, and exploratory implementations. Trades comprehensive review and test coverage for speed of iteration. Still maintains basic quality standards (code compiles, builds cleanly, runs correctly).

## When To Use

Use this when:
- Proving out an idea or architectural approach before committing to full implementation
- Building a spike to evaluate feasibility
- Creating a throwaway prototype to demonstrate behavior
- Exploring a new library, API, or integration
- The user explicitly wants to move fast and iterate

Do NOT use this when:
- Prototyping a TUI or interactive terminal application (use tui-prototype instead — it routes to the tui-engineer agent and has TUI-specific scaffolding and verification)
- Building production-ready features (use feature-workflow instead)
- Fixing a bug in production code (use bug-triage instead)
- The change will be merged directly to main without further work

## Workflow

### Phase 1: Quick Plan
**Agent**: planner

1. Clarify the goal — what are we trying to prove or demonstrate?
2. Identify the minimal scope needed to answer the question
3. List the steps in implementation order
4. Flag any hard blockers (missing dependencies, unclear requirements)

Keep the plan lightweight. Focus on "what do we build" not "how do we ship it."

**Gate**: Plan must be reviewed and approved before proceeding.

### Phase 2: Branch
**Agent**: git-workflow

1. Create a prototype branch: `proto/<description>`
2. Keep the name descriptive — prototypes often outlive expectations

### Phase 3: Design (if needed)
**Agents**: api-designer, db-architect

Only invoke if the plan explicitly calls for API or schema design. For prototypes, this is often skippable — a rough shape is enough.

If invoked, the designer should favor pragmatic, minimal designs over comprehensive ones. Get the shape right; details can be refined later.

### Phase 4: Build
**Agent**: builder

1. Implement the prototype following the plan
2. Prioritize getting something working over getting it perfect
3. Use existing patterns where convenient, but don't over-invest in consistency for throwaway code
4. Handle the happy path thoroughly; error handling can be basic (log and return error, don't build elaborate error hierarchies)
5. Run `go build ./...` and `go vet ./...` to verify basic correctness
6. If something works but is ugly, move on — this is a prototype

### Phase 5: Smoke Test
**Agent**: builder

1. Run `go test ./...` if tests already exist — don't break existing tests
2. Manually verify the prototype works for the primary use case
3. If the prototype includes a web UI and a running instance is available, invoke the `qa` agent for a quick smoke test (not a comprehensive QA pass)

### Phase 6: Wrap Up
**Agent**: git-workflow

1. Commit with a clear message that this is a prototype: `proto: <description>`
2. Optionally create a lightweight PR if the user wants one
3. Present a brief summary:
   - What was built
   - What works
   - Known gaps and shortcuts taken
   - Recommended next steps if the prototype proves viable

## Principles

- **Speed over polish**: The goal is to learn, not to ship. Get to a working state as fast as possible
- **Working over complete**: A prototype that demonstrates the core idea with rough edges is more valuable than a half-finished "clean" implementation
- **Compile and run**: The bar is not "production-ready" but it IS "compiles, runs, and does what it claims." Broken code teaches nothing
- **Know what you skipped**: Track shortcuts and gaps explicitly so the path to production is clear if the prototype succeeds
- **Don't gold-plate**: Resist the urge to add tests, refactor, or polish. If the prototype validates the idea, those activities belong in a proper feature-workflow pass
