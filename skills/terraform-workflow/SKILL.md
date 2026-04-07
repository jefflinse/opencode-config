---
name: terraform-workflow
description: Infrastructure-as-code workflow covering Terraform init, plan, review, apply, and verification with safety checks and state management
---

## What This Skill Does

Defines the workflow for safely provisioning and modifying cloud infrastructure using Terraform. Covers planning, review, apply, and verification with appropriate safety checks at each stage.

## When To Use

Use this when:
- Provisioning new cloud infrastructure (VPCs, databases, compute, storage)
- Modifying existing Terraform-managed infrastructure
- Creating or updating Terraform modules
- Setting up a new Terraform project/workspace

Do NOT use this when:
- The change is application code only (use feature-workflow)
- The change is CI/CD pipeline files only (use ci-ops directly)
- The infrastructure isn't managed by Terraform (use devops-engineer directly)

## Workflow

### Phase 1: Plan the Change
**Agent**: devops-engineer

1. Understand the infrastructure requirement
2. Identify which Terraform workspace/state file is affected
3. Check existing modules that could be reused
4. Identify potential impacts on running services
5. Flag any destructive changes (resource destruction, replacement)

**Gate**: Infrastructure plan must be approved before writing Terraform code.

### Phase 2: Write Terraform Code
**Agent**: devops-engineer

1. Write or modify Terraform configuration files
2. Follow existing conventions (naming, module structure, variable patterns)
3. Pin provider versions and module sources
4. Use variables for environment-specific values — never hardcode
5. Add descriptions to all variables and outputs
6. Run `terraform fmt` to format all files
7. Run `terraform validate` to check syntax

**Acceptance criteria**: `terraform validate` passes, `terraform fmt` produces no changes.

### Phase 3: Security Review
**Agent**: security-auditor

Review the Terraform code for:
1. Overly permissive IAM policies or security group rules
2. Missing encryption at rest or in transit
3. Public access to resources that should be private
4. Hardcoded secrets or credentials
5. Missing logging and audit trails
6. Non-compliant resource configurations

**Gate**: No CRITICAL or HIGH security findings before proceeding.

### Phase 4: Plan and Review
**Agent**: devops-engineer

1. Run `terraform plan` and capture the output
2. Review every resource change: create, update, destroy
3. Verify no unexpected destroys or replacements
4. Check for resources that will cause downtime during modification
5. Estimate cost impact of new resources
6. Present the plan summary to the user

**Gate**: Terraform plan must be explicitly approved before apply. Special attention to:
- Any resource destruction
- Any resource replacement (destroy + create)
- Any changes to networking, security groups, or IAM
- Any changes to database instances

### Phase 5: Apply
**Agent**: devops-engineer

1. Run `terraform apply` with the saved plan file
2. Monitor the apply for errors
3. If apply fails partway: assess the partial state, do NOT re-run blindly
4. Verify the apply completed successfully

**Critical rule**: Never run `terraform apply` without an approved plan. Never use `-auto-approve` in production.

### Phase 6: Verify
**Agent**: devops-engineer

1. Verify the infrastructure is working as expected:
   - Health checks pass
   - Connectivity is correct
   - DNS resolves properly
   - Services can reach the new infrastructure
2. Run `terraform plan` again to verify no drift (plan should show "no changes")
3. Verify monitoring and alerting are configured for new resources

### Phase 7: Document
**Agent**: docs-writer

1. Update infrastructure documentation (if maintained separately)
2. Document any manual steps that were required
3. Update runbooks if operational procedures changed

## State Management Safety

- **Always use remote state** (S3+DynamoDB, GCS, Azure Blob) — never local state in shared environments
- **Lock state during operations** — Terraform does this automatically with DynamoDB/GCS/Azure
- **Never manually edit state** — use `terraform state mv`, `terraform state rm`, or `terraform import`
- **Back up state before risky operations** — especially before `terraform state rm` or large refactors

## Rollback Procedure

If the apply causes issues:
1. If the previous state is available: `terraform apply` with the previous configuration
2. If the change was additive: `terraform destroy -target=<new_resource>`
3. If the change was destructive: restore from backup (database, state file)
4. Document what went wrong for post-mortem

## Principles

- **Plan before apply**: Always review the plan. Always.
- **Smallest possible change**: Don't refactor modules and add resources in the same apply.
- **Infrastructure is cattle, state is sacred**: Resources can be recreated. State files cannot.
- **Test in staging first**: Apply to staging before production. Compare the plans.
- **Version everything**: Terraform code, provider versions, module versions — all pinned.
