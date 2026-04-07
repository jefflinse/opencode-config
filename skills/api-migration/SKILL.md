---
name: api-migration
description: API versioning transition workflow covering parallel deployment, consumer migration, deprecation, and sunset of old API versions
---

## What This Skill Does

Defines the workflow for migrating an API from one version to another — deploying the new version alongside the old, migrating consumers, deprecating the old version, and eventually sunsetting it. Ensures zero downtime and no broken consumers throughout the transition.

## When To Use

Use this when:
- Introducing a breaking change to an existing API (new version needed)
- Migrating consumers from v1 to v2 of an API
- Sunsetting an old API version after a deprecation period
- Changing API contract (request/response shapes, authentication, error formats)

Do NOT use this when:
- Adding new endpoints to an existing API version (use feature-workflow)
- Making backward-compatible changes (just deploy them)
- Building a brand-new API with no existing consumers (use feature-workflow)

## Workflow

### Phase 1: Assess Impact
**Agent**: migration-planner

1. Catalog the current API surface: all endpoints, request/response contracts, auth mechanisms
2. Identify all consumers (internal services, external integrations, mobile apps, SPAs)
3. Map which endpoints have breaking changes and what the changes are
4. Assess consumer migration difficulty for each consumer
5. Propose a versioning strategy (URL path, header, or query parameter)

**Gate**: Impact assessment must be approved before design begins.

### Phase 2: Design New Version
**Agent**: api-designer

1. Design the new API version following the project's conventions
2. Document all breaking changes with migration guidance for each
3. Design backward-compatible fallbacks where possible
4. Define the deprecation timeline: announcement → warning → sunset
5. Produce OpenAPI/protobuf specs for the new version

**Gate**: New API design must be approved before implementation.

### Phase 3: Implement New Version
**Agent**: builder, ts-builder, or java-builder (depending on stack)

1. Implement the new API version alongside the old one (both serve traffic)
2. Share business logic between versions — only the API layer should differ
3. Add deprecation headers to old version responses: `Deprecation: true`, `Sunset: <date>`
4. Add logging to track which consumers use which version
5. Run the full test suite to verify the old version is unaffected

**Acceptance criteria**: Both old and new versions work. Old version has deprecation headers. Usage logging is in place.

### Phase 4: Test Both Versions
**Agent**: test-writer, ts-test-writer, or java-test-writer (depending on stack)

1. Write tests for the new API version covering all endpoints
2. Write migration tests that verify old-to-new request/response mapping
3. Verify the old API version's existing tests still pass
4. Run integration tests with both versions active

### Phase 5: Consumer Migration
**Agent**: migration-planner (tracking and coordination)

1. Notify consumers of the new version and migration timeline
2. Provide migration guides specific to each consumer type
3. Track consumer migration progress (which consumers have migrated)
4. Assist with consumer code changes if they're internal services
5. Monitor error rates on both old and new versions

**Gate**: All consumers migrated (or explicit deadline reached) before sunset.

### Phase 6: Sunset Old Version
**Agent**: builder, ts-builder, or java-builder (depending on stack)

1. Verify no traffic on the old version (or only de minimis traffic from unmigrated consumers)
2. Return `410 Gone` on old version endpoints (instead of immediate removal)
3. After the grace period: remove old version code, routes, and tests
4. Clean up shared code that was only needed for backward compatibility

### Phase 7: Finalize
**Agent**: git-workflow, docs-writer

1. Update API documentation to reflect only the current version
2. Create PR(s) with the full migration changeset
3. Document the migration for future reference (what changed, why, lessons learned)

## Principles

- **Parallel deployment**: Old and new versions must coexist. Never replace in-place.
- **Zero broken consumers**: No consumer should experience unplanned downtime during migration.
- **Deprecation before removal**: Always warn before sunsetting. Use HTTP deprecation headers and documentation.
- **Track everything**: Know exactly which consumers use which version at all times.
- **Share logic, separate contracts**: Business logic should not be duplicated between versions. Only the API contract layer should differ.
