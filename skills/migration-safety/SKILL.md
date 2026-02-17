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

## Migration Safety Checklist

### Before Writing

```
[ ] Understand the current schema and all code that queries the affected tables
[ ] Identify all services/applications that access the affected tables
[ ] Determine if the migration needs to be backward compatible (yes if using rolling deploys)
[ ] Estimate the migration duration against production data volume
```

### Adding Columns

```
[ ] New column is nullable OR has a default value (never add NOT NULL without default to existing table)
[ ] Default value is appropriate and won't cause issues with existing rows
[ ] If adding an index, use CREATE INDEX CONCURRENTLY (PostgreSQL) to avoid table lock
[ ] Old application code won't break with the new column present
```

### Removing Columns

```
[ ] Column is no longer referenced by any application code (deploy code change FIRST)
[ ] Column removal is in a SEPARATE migration from the code change
[ ] Data in the column has been preserved elsewhere if needed
[ ] Down migration can recreate the column (nullable, as data is lost)
```

### Renaming Columns

**Do not rename columns in a single migration.** Instead:
1. Add the new column
2. Deploy code that writes to both old and new columns
3. Backfill the new column from the old column
4. Deploy code that reads from the new column
5. Remove the old column

### Modifying Column Types

```
[ ] New type is compatible with existing data (e.g., VARCHAR(50) → VARCHAR(100) is safe; the reverse may not be)
[ ] Application code handles both old and new types during transition
[ ] If narrowing a type, verify no existing data will be truncated or rejected
```

### Adding/Modifying Constraints

```
[ ] Existing data satisfies the new constraint (run a validation query first)
[ ] If adding NOT NULL, existing NULL values are handled (backfill first)
[ ] Foreign key constraints won't cause cascading issues
[ ] Check constraints are valid for all existing rows
```

### Dropping Tables

```
[ ] No application code references the table
[ ] No foreign keys reference the table from other tables
[ ] Data has been archived or migrated if needed
[ ] Down migration can recreate the table structure (data loss is expected and documented)
```

### Data Migrations

```
[ ] Schema migration and data migration are SEPARATE files
[ ] Data migration is idempotent (safe to run multiple times)
[ ] Data migration handles NULL values and edge cases
[ ] Data migration performance is tested against production-like data volume
[ ] Data migration runs in batches for large tables (not one giant UPDATE)
```

## Deployment Strategy

### Expand-Contract Pattern (Recommended for breaking changes)

1. **Expand**: Add new columns/tables, deploy code that writes to both old and new
2. **Migrate**: Backfill new columns/tables from old data
3. **Switch**: Deploy code that reads from new, writes to new only
4. **Contract**: Remove old columns/tables

This pattern ensures zero-downtime and backward compatibility at every step.

### Rollback Plan

For every migration, document:
```
[ ] Down migration tested and verified
[ ] Rollback procedure documented (which migrations to reverse, in what order)
[ ] Data loss from rollback is understood and acceptable
[ ] Rollback can be performed without downtime
```

## Testing

1. **Apply and rollback**: Run `up` then `down` then `up` again — must succeed
2. **Test against production-like data**: Use a copy of production data (anonymized) to verify performance
3. **Test with running application**: Apply the migration while the old version of the application is running — it must not break
4. **Verify indexes**: After migration, verify query plans haven't degraded with `EXPLAIN ANALYZE`

## Agent Coordination

- **db-architect**: Writes the migration and reviews schema design
- **builder**: Updates application code to work with the new schema
- **security-auditor**: Reviews for data exposure risks in migration
- **ci-ops**: Ensures migration runs in CI test pipeline
