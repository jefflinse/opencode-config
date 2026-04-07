---
name: polyglot-feature-workflow
description: Cross-stack feature implementation workflow for features spanning Go, TypeScript, Java, and infrastructure with coordinated multi-builder execution
---

## What This Skill Does

Defines the workflow for implementing features that span multiple technology stacks — for example, a Go backend API + React frontend + database migration, or a Java microservice + Terraform infrastructure + CI pipeline updates. Coordinates the right builder agent for each stack component.

## When To Use

Use this when a feature requires changes across two or more technology stacks:
- Go backend + TypeScript frontend
- Java service + database migration + infrastructure provisioning
- Multiple microservices in different languages communicating via API
- Application code + infrastructure + CI/CD changes

Do NOT use this when:
- The feature is entirely within one stack (use feature-workflow instead)
- The change is infrastructure-only (use terraform-workflow)
- The change is a single-stack prototype (use rapid-prototype)

## Workflow

### Phase 1: Plan
**Agent**: planner

1. Identify all technology stacks involved in the feature
2. Decompose into per-stack implementation steps
3. Identify dependencies between stacks (e.g., API contract must be defined before frontend can consume it)
4. Route each step to the correct agent:
   - Go code → builder
   - TypeScript/JavaScript code → ts-builder
   - Java/Spring Boot code → java-builder
   - Shell scripts → shell-scripter
   - Infrastructure (Terraform, K8s) → devops-engineer
   - CI/CD pipelines → ci-ops
   - Database schema → db-architect
   - API contracts → api-designer
5. Identify cross-stack integration points and how to test them
6. Flag cross-cutting concerns (auth, security, performance)

**Gate**: Plan must be approved before implementation begins.

### Phase 2: Branch
**Agent**: git-workflow

1. Create a feature branch: `feat/<description>`
2. If the feature is very large, consider one branch per stack with a coordination branch

### Phase 3: Design Cross-Stack Contracts
**Agent**: api-designer

1. Define the API contracts that bridge the stacks:
   - REST/gRPC endpoint definitions
   - Request/response schemas
   - Error format conventions
   - Authentication/authorization requirements
2. Produce OpenAPI specs, protobuf definitions, or TypeScript types as appropriate
3. These contracts become the source of truth that both sides implement against

**Gate**: API contracts must be approved before implementation. Both producers and consumers must agree on the contract.

### Phase 4: Build (Per-Stack, Parallelizable)

Execute the following in parallel where dependencies allow:

#### 4a: Backend Implementation
**Agent**: builder, java-builder, or ts-builder (depending on backend stack)

1. Implement the backend API/service per the agreed contract
2. Follow existing conventions in the backend codebase
3. Run build and tests for the backend stack

#### 4b: Frontend Implementation
**Agent**: ts-builder (or appropriate frontend agent)

1. Implement the frontend changes per the agreed contract
2. Use the API contract for type generation or mock responses during development
3. Run build and tests for the frontend stack

#### 4c: Infrastructure (if needed)
**Agent**: devops-engineer

1. Provision any new infrastructure required by the feature
2. Follow terraform-workflow safety checks
3. Verify infrastructure is ready before integration testing

#### 4d: Database (if needed)
**Agent**: db-architect

1. Design and write the migration
2. Verify backward compatibility
3. Apply the migration to the development environment

### Phase 5: Per-Stack Testing

**Agent**: test-writer, ts-test-writer, java-test-writer (matching each stack)

1. Write unit and integration tests for each stack component
2. Each stack's tests should pass independently (mocking cross-stack dependencies)
3. Run each stack's test suite

### Phase 6: Cross-Stack Integration Testing

**Agent**: builder or ts-builder (whoever owns the integration point)

1. Start all services (use docker-compose or equivalent)
2. Run integration tests that exercise the full cross-stack flow
3. Verify the contract is honored: frontend ↔ backend ↔ database
4. If a web UI is involved and a running instance is available, invoke qa for browser testing

### Phase 7: Review

**Agent**: code-reviewer + specialists

1. Code review for each stack (reviewer should apply language-appropriate standards)
2. Security review if the feature handles user input, auth, or cross-service communication
3. Infrastructure review if Terraform/K8s changes are included
4. Address all BLOCKING findings

**Gate**: No BLOCKING findings remaining.

### Phase 8: Polish and Finalize

**Agent**: builder/ts-builder/java-builder (fixes), docs-writer (documentation), git-workflow (PR)

1. Address review feedback
2. Update documentation for each affected stack
3. Run all test suites one final time
4. Create PR with a structured description covering all stack changes

## Principles

- **Contract first**: Define the API contracts before implementing either side. This enables parallel development.
- **Per-stack testing + cross-stack integration**: Each stack should be testable in isolation AND together.
- **Right agent for each stack**: Never ask the Go builder to write TypeScript or vice versa. Route correctly.
- **Parallel when possible**: If the frontend and backend can be built in parallel (once the contract is defined), do so.
- **Single branch, logical commits**: Group commits by stack and concern. The PR should tell a coherent story of a cross-stack feature.
