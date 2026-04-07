---
name: onboarding-guide
description: Generates comprehensive codebase onboarding documentation covering architecture, conventions, workflows, and development setup
---

## What This Skill Does

Generates a comprehensive onboarding guide for a codebase — the document you wish existed when you first joined the project. Covers architecture overview, conventions, development setup, common workflows, and "where to find things."

## When To Use

Use this when:
- A new team member needs to ramp up on the codebase
- The project has no onboarding documentation
- Existing documentation is severely outdated
- You want a structured overview of an unfamiliar codebase

Do NOT use this when:
- You need API documentation (use docs-writer directly)
- You need architecture decisions documented (use architect)
- The project is brand new with no code to document (use a project init skill)

## Workflow

### Phase 1: Explore the Codebase
**Agent**: planner (analysis mode)

1. Read the project's existing README, CONTRIBUTING, and any docs/ directory
2. Analyze the project structure:
   - Directory layout and organization
   - Language(s) and framework(s)
   - Build system(s) and tooling
   - Test structure and coverage approach
3. Identify the main entry points (cmd/main, src/index, Application.java)
4. Map the dependency graph (what depends on what)
5. Identify key architectural patterns in use

### Phase 2: Map Architecture
**Agent**: architect (read-only assessment)

1. Identify the system architecture:
   - Monolith, microservices, or monorepo?
   - What services exist and what do they do?
   - How do services communicate?
2. Map the data flow:
   - How do requests enter the system?
   - What databases/caches/queues are used?
   - Where is state stored?
3. Identify key abstractions:
   - Core domain models/entities
   - Interface boundaries between modules
   - Extension points and plugin systems

### Phase 3: Document Conventions
**Agent**: code-reviewer (read-only assessment)

1. Identify coding conventions:
   - Naming patterns
   - Error handling patterns
   - Logging patterns
   - Testing patterns
2. Identify project-specific patterns:
   - How are new endpoints/features added?
   - How is configuration managed?
   - How are database migrations handled?
   - How is authentication/authorization implemented?
3. Identify tooling conventions:
   - Linting and formatting tools
   - CI/CD pipeline
   - Deployment process
   - Branching and PR strategy

### Phase 4: Write the Onboarding Guide
**Agent**: docs-writer

Produce a comprehensive document with these sections:

```markdown
# <Project Name> — Onboarding Guide

## What This Project Does
<1-3 sentences explaining the project's purpose and who uses it>

## Architecture Overview
<High-level architecture diagram or description>
<Service boundaries, communication patterns, data flow>

## Tech Stack
<Languages, frameworks, databases, infrastructure — with versions>

## Project Structure
<Directory tree with annotations explaining each top-level directory>

## Getting Started
### Prerequisites
<What to install before you can work on this project>

### Setup
<Step-by-step: clone, install deps, configure, run locally>

### Running Tests
<How to run unit tests, integration tests, and end-to-end tests>

### Common Development Tasks
<How to: add an endpoint, write a migration, add a new service, etc.>

## Key Conventions
### Code Style
<Naming, formatting, patterns that the team follows>

### Error Handling
<How errors are handled, logged, and returned to clients>

### Testing
<What to test, how to test, where tests go>

### Git Workflow
<Branching strategy, commit message format, PR process>

## Where to Find Things
<Map of "I need to do X, look in Y" for common tasks>

## Key Decisions and Why
<Important architectural or technology decisions and their rationale>

## Known Gotchas
<Things that commonly trip up new developers>
```

### Phase 5: Review
**Agent**: code-reviewer

1. Verify the guide is accurate against the actual codebase
2. Check for outdated information
3. Verify the setup instructions actually work
4. Identify gaps — important aspects of the project not covered

**Gate**: Guide must be accurate and complete before delivery.

## Principles

- **Accuracy over polish**: It's better to be correct and rough than polished and wrong. Verify everything against the actual code.
- **Show, don't tell**: Include actual file paths, command examples, and code snippets — not vague descriptions.
- **Assume intelligence, not context**: The reader is a competent engineer who doesn't know this specific project. Don't explain what a REST API is; do explain why this project uses gRPC instead.
- **Living document**: Note where the guide is likely to become outdated and suggest update triggers.
- **Test the setup instructions**: If the guide says "run make dev", verify that command actually works.
