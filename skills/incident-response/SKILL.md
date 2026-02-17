---
name: incident-response
description: Production incident response workflow covering immediate mitigation, diagnosis, fix, verification, and post-incident review
---

## What This Skill Does

Defines the process for responding to a production incident — something is broken right now and users are affected. This is distinct from bug-triage, which handles known bugs that aren't actively causing harm. Incident response prioritizes mitigation over diagnosis.

## When To Use

Use this when:
- A production service is down, degraded, or returning errors
- Monitoring/alerting has fired for an anomaly
- Users are reporting problems that affect production functionality
- A deployment has gone wrong and needs to be rolled back

Do NOT use this for:
- Bugs found in development or staging → use bug-triage
- Performance issues noticed during testing → use code-health-check
- Planned maintenance → use release-prep or migration-safety

## Key Principle: Mitigate First, Diagnose Second

The priority order during an incident is:
1. **Restore service** — get users back to a working state
2. **Diagnose** — understand what went wrong
3. **Fix** — implement a proper solution
4. **Prevent** — ensure it can't happen again

Do not spend 30 minutes diagnosing a root cause while users are down if you can roll back in 2 minutes.

## Workflow

### Phase 1: Assess
**Agent**: debugger

Time budget: 5 minutes maximum before deciding on mitigation.

1. What is the impact? Who and how many users are affected?
2. When did it start? Correlate with recent deployments, config changes, or dependency updates
3. What are the symptoms? Error rates, latency spikes, specific error messages, affected endpoints
4. Is the issue getting worse, stable, or resolving on its own?

Gather evidence from:
- Application logs (look for error patterns, stack traces, panic output)
- Monitoring dashboards (error rates, latency percentiles, resource usage)
- Recent deployment history (`git log --oneline -10`, CI/CD deployment records)
- Infrastructure status (database connectivity, external service health)

### Phase 2: Mitigate
**Agent**: ci-ops (Options A, B, D), builder (Option C)

Based on the assessment, choose the fastest path to restoration. The orchestrator should select the option and invoke the appropriate agent:

#### Option A: Rollback → invoke ci-ops
Preferred if a recent deployment correlates with the incident.
1. Identify the last known-good deployment
2. Roll back to it: revert the deployment, redeploy the previous container image, or revert the Git commit
3. Verify the service recovers
4. If rollback doesn't resolve the issue, it wasn't the deployment — try another option

#### Option B: Feature flag / circuit breaker → invoke ci-ops
If the failing functionality can be disabled without affecting the rest of the service.
1. Disable the failing feature via config change, feature flag, or environment variable
2. Deploy the config change
3. Verify the service stabilizes

#### Option C: Hotfix → invoke builder
Only if the fix is obvious, small, and can be deployed quickly.
1. The hotfix must be the minimal change that restores service — no cleanup, no refactoring
2. Skip the full review process, but invoke code-reviewer for a quick sanity check
3. Deploy through the normal pipeline if possible, or fast-track if necessary

#### Option D: Infrastructure mitigation → invoke ci-ops
For resource or infrastructure issues.
1. Scale up if the issue is resource exhaustion
2. Restart pods/containers if the issue is a corrupted state
3. Failover to secondary if the issue is infrastructure-specific

**Gate**: Service must be restored (or actively recovering) before moving to Phase 3. If mitigation has not succeeded within 15 minutes, escalate to the user with a status report and ask for direction.

### Phase 3: Diagnose
**Agent**: debugger

Now that the fire is out, find the root cause:

1. Correlate the incident timeline with changes:
   - Code deployments
   - Configuration changes
   - Database migrations
   - Dependency updates
   - Infrastructure changes
   - External service outages
2. Read the logs from the incident window — focus on the first error, not the cascade that followed
3. If the incident was caused by a code change, identify the specific commit and the defect in it
4. If the incident was caused by load, identify the bottleneck (CPU, memory, connections, external dependency)
5. If the incident was caused by data, identify the specific data condition that triggered the failure

Produce a diagnosis report:
```
## Incident Diagnosis

### Timeline
- <HH:MM> — First symptom observed
- <HH:MM> — Impact confirmed, mitigation started
- <HH:MM> — Service restored
- <HH:MM> — Root cause identified

### Root Cause
<what actually broke and why>

### Contributing Factors
<what made this possible — missing tests, missing monitoring, insufficient validation, etc.>

### Impact
<duration, users affected, data implications>
```

**Gate**: Present the diagnosis report to the user before proceeding to the fix phase. The root cause must be confirmed before a fix is implemented.

### Phase 4: Fix
**Agents**: builder, test-writer

If a rollback was the mitigation, a proper fix is still needed. Follow the bug-triage skill's Phase 3-5 process:
1. **builder**: Implement the minimal fix that addresses the root cause
2. **test-writer**: Write a regression test that reproduces the incident condition (must fail without the fix, pass with it)
3. **builder**: Verify the fix against the specific data/load/timing conditions that caused the incident
4. Deploy the fix through the normal pipeline with full review

If a hotfix was the mitigation:
1. **code-reviewer**: Review the hotfix properly (it was fast-tracked during the incident)
2. If the hotfix is adequate as a permanent fix, have **test-writer** add tests and close
3. If the hotfix is a band-aid, have **builder** implement a proper fix and remove the hotfix

### Phase 5: Post-Incident Review
**Agent**: planner (to structure follow-up work)

Produce a post-incident report and action items:

```
## Post-Incident Report

### Summary
<1-2 sentences: what happened and how it was resolved>

### Detection
- How was the incident detected? (Monitoring, user report, manual observation)
- How long between the start of the incident and detection?
- What monitoring would have detected it sooner?

### Response
- How long between detection and mitigation?
- Was the mitigation approach correct, or was time wasted on the wrong path?
- What would have made the response faster?

### Root Cause
<from Phase 3 diagnosis>

### Action Items
1. [IMMEDIATE] <fix the root cause if not already done>
2. [SHORT-TERM] <add monitoring/alerting to detect this class of issue>
3. [SHORT-TERM] <add tests that would have caught this before deployment>
4. [MEDIUM-TERM] <address contributing factors — missing validation, insufficient load testing, etc.>
5. [LONG-TERM] <systemic improvements — better rollback automation, circuit breakers, etc.>
```

## Incident Severity Guide

| Severity | Definition | Response Time |
|----------|------------|---------------|
| **SEV-1** | Service completely down, all users affected | Immediate — drop everything |
| **SEV-2** | Major functionality degraded, many users affected | Within 15 minutes |
| **SEV-3** | Minor functionality affected, some users impacted | Within 1 hour |
| **SEV-4** | Cosmetic or minor issue, workaround available | Next business day |

## Principles

- **Mitigate first**: A 2-minute rollback is better than a 30-minute diagnosis while users are down
- **Don't make it worse**: Before running any mitigation, consider whether it could cause additional damage (e.g., restarting a database mid-migration)
- **Communicate**: Keep stakeholders informed of status, impact, and estimated resolution time
- **Log everything**: Document what you tried, what worked, and what didn't — this is the raw material for the post-incident review
- **No blame**: The post-incident review focuses on systems and processes, not individuals. The goal is to prevent recurrence, not to assign fault
- **Incidents are learning opportunities**: Every incident that results in improved monitoring, tests, or processes makes the system more resilient
