---
name: log-analysis
description: Production log and trace analysis workflow for diagnosing incidents, identifying patterns, and producing operational insights
---

## What This Skill Does

Defines the workflow for analyzing production logs, distributed traces, and metrics to diagnose issues, identify patterns, and produce actionable operational insights. Structures the investigation from symptom through evidence to root cause.

## When To Use

Use this when:
- Investigating elevated error rates or latency in production
- Post-incident analysis to understand what happened
- Identifying patterns in recurring issues
- Validating that a fix resolved the production issue

Do NOT use this when:
- The issue is clearly a code bug with a stack trace (use debugger directly)
- The system is currently down (use incident-response — triage first, analyze later)
- You need to profile for optimization (use performance-audit)

## Workflow

### Phase 1: Define Investigation Scope
**Agent**: log-analyst

1. What is the symptom? (errors, latency, data inconsistency)
2. When did it start? (timestamp or triggering event)
3. What is the blast radius? (all users, specific endpoints, specific regions)
4. What observability data is available? (log aggregator, tracing system, metrics dashboards)
5. What has already been checked?

**Gate**: Investigation scope must be agreed upon before deep analysis.

### Phase 2: Gather Evidence
**Agent**: log-analyst

1. **Logs**: Pull logs for the affected time window from relevant services
   - Filter by service, log level, endpoint
   - Extract error messages and stack traces
   - Identify unique error types and their frequency
2. **Traces**: Find traces for affected requests
   - Identify slow spans and error spans
   - Map the request flow across services
   - Compare healthy vs. unhealthy traces
3. **Metrics**: Check dashboards for the affected period
   - Request rate, error rate, latency percentiles
   - Resource utilization (CPU, memory, connections, disk)
   - Dependency health (database, cache, external APIs)
4. **Deployment history**: Check for recent deploys, config changes, or infrastructure changes

### Phase 3: Analyze and Correlate
**Agent**: log-analyst

1. Build a timeline: what happened, in what order?
2. Correlate events across data sources:
   - Do error spikes align with deploy times?
   - Do latency increases correlate with traffic increases?
   - Do specific error types cluster around specific services or endpoints?
3. Identify the upstream cause:
   - Where in the call chain does the problem originate?
   - Is it propagating to downstream services?
4. Categorize the issue:
   - Deploy regression
   - Resource exhaustion
   - Upstream dependency failure
   - Traffic spike / scaling issue
   - Data-dependent / edge case bug
   - Configuration issue

### Phase 4: Diagnose
**Agent**: log-analyst

1. State the root cause with supporting evidence
2. Explain the causal chain from root cause to observed symptom
3. Quantify the impact (affected requests, duration, data impact)
4. Identify any secondary issues discovered during analysis

### Phase 5: Recommend
**Agent**: log-analyst

1. **Immediate**: What should be done right now (if the issue is ongoing)?
2. **Short-term**: What code/config/infrastructure changes prevent recurrence?
3. **Observability improvements**: What alerting or monitoring gaps did this reveal?

Route recommendations to appropriate agents:
- Code fixes → builder, ts-builder, java-builder
- Infrastructure fixes → devops-engineer
- Monitoring/alerting → devops-engineer
- Security concerns → security-auditor

### Phase 6: Document (if post-incident)
**Agent**: docs-writer

1. Write a post-incident report with:
   - Timeline of events
   - Root cause analysis
   - Impact assessment
   - Actions taken
   - Preventive measures
   - Lessons learned

## Principles

- **Evidence before conclusions**: Every diagnosis must be supported by specific log entries, traces, or metrics.
- **Timeline is king**: Build the timeline first. Most production issues become obvious once you know the exact sequence of events.
- **Correlation ≠ causation**: Verify the causal chain, don't just note coincidences.
- **Check the simple things first**: Recent deploy? Upstream outage? Traffic spike? Don't hunt for exotic causes before ruling out the obvious.
- **Quantify everything**: "Some errors" → "2,847 500-errors on /api/checkout between 14:30-15:15 UTC affecting ~12% of requests."
