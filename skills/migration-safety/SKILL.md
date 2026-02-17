---
name: migration-safety
description: Database migration safety checklist covering backward compatibility, rollback planning, data integrity, and zero-downtime deployment
---

## What This Skill Does

Provides a safety framework for writing and deploying database migrations. Ensures migrations are backward compatible, reversible, and safe to run against production data.

## When To Use

Use this when:
- Writing a new database migration
- Reviewing a migration before it's applied to staging or production
- Planning a schema change that affects a production system
- Performing a data migration alongside a schema change

## Safety Principles

1. **Backward compatibility**: The old application code must work with the new schema during deployment
2. **Reversibility**: Every migration must have a working rollback (`down` migration)
3. **Data preservation**: Migrations must never lose data without explicit, documented intent
4. **Zero-downtime**: Migrations must not lock tables for extended periods in production
5. **Testability**: Migrations must be tested against production-like data volumes

## Workflow

### Phase 1: Assess the Change
**Agent**: db-architect

1. Understand the current schema and all code that queries the affected tables
2. Identify all services/applications that access the affected tables
3. Determine if the migration needs to be backward compatible (yes if using rolling deploys)
4. Classify the change type: adding columns, removing columns, renaming columns, modifying types, adding/modifying constraints, dropping tables, or data migration
5. Estimate the migration duration against production data volume
6. Determine the deployment strategy: simple migration (backward compatible) or expand-contract pattern (breaking changes)

**Gate**: Present the assessment to the user. The change classification, backward compatibility requirement, and deployment strategy must be confirmed before writing the migration.

### Phase 2: Write the Migration
**Agent**: db-architect

Write the migration following the safety rules for the specific change type:

**Adding columns**: New column must be nullable OR have a default value. Use `CREATE INDEX CONCURRENTLY` for new indexes.

**Removing columns**: Column must already be unreferenced by application code. Removal must be in a SEPARATE migration from the code change.

**Renaming columns**: Do NOT rename in a single migration. Use the expand-contract pattern: add new column → backfill → switch reads → drop old column.

**Modifying column types**: New type must be compatible with existing data. Verify no existing data will be truncated.

**Adding/modifying constraints**: Existing data must satisfy the new constraint. Backfill NULL values before adding NOT NULL.

**Dropping tables**: No application code or foreign keys reference the table. Data archived if needed.

**Data migrations**: Separate from schema migrations. Idempotent. Batched for large tables.

Every migration must include both `up` and `down`. The `down` must actually reverse the `up`.

Acceptance criteria: Migration SQL is syntactically valid for the target database. Up and down are both present.

### Phase 3: Write Application Code Changes
**Agent**: builder

If the migration requires application code changes:
1. Update repository/store layer to work with the new schema
2. If using expand-contract pattern: implement the dual-write logic for the transition period
3. Handle `sql.ErrNoRows` and other query changes that result from the schema change
4. Ensure the application code works with BOTH the old and new schema during the transition

Acceptance criteria: `go build ./...` compiles, `go test ./...` passes.

### Phase 4: Test the Migration
**Agent**: builder

1. Apply and rollback: Run `up` then `down` then `up` again — must succeed
2. Verify the application works with the new schema: run `go test ./...`
3. If the project has integration tests against a real database, run those
4. Verify query plans haven't degraded: spot-check with `EXPLAIN ANALYZE` on key queries

Acceptance criteria: Migration applies and rolls back cleanly. All tests pass.

### Phase 5: Security Review
**Agent**: security-auditor

Review the migration for:
1. Data exposure risks: does the migration create new columns that could leak sensitive data?
2. Permission changes: does the migration alter access controls?
3. Secrets: are there any hardcoded values, connection strings, or credentials in the migration?

### Phase 6: Review
**Agents**: code-reviewer, db-architect

1. code-reviewer reviews the application code changes
2. db-architect reviews the migration SQL for correctness, safety, and adherence to the checklist

**Gate**: Present all findings. BLOCKING findings must be addressed before the migration is applied to any environment.

### Phase 7: Document the Rollback Plan
**Agent**: db-architect

Produce a rollback plan:
```
- Down migration tested: yes/no
- Rollback procedure: which migrations to reverse, in what order
- Data loss from rollback: describe what would be lost
- Rollback can be performed without downtime: yes/no
- Estimated rollback time: <duration>
```

**Gate**: Present the rollback plan to the user. The user must acknowledge the rollback implications before the migration is committed.

## Reference: Deployment Strategies

### Simple Migration (backward compatible)
The new schema works with both old and new application code. Apply the migration, then deploy the new code. Rollback is straightforward: reverse the migration, redeploy old code.

### Expand-Contract Pattern (breaking changes)
1. **Expand**: Add new columns/tables, deploy code that writes to both old and new
2. **Migrate**: Backfill new columns/tables from old data
3. **Switch**: Deploy code that reads from new, writes to new only
4. **Contract**: Remove old columns/tables

Each step is a separate migration and deployment. This ensures zero-downtime and backward compatibility at every step.
