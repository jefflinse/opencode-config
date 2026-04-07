---
name: tech-debt-sprint
description: Focused technical debt reduction workflow with assessment, prioritization, incremental remediation, and progress tracking
---

## What This Skill Does

Defines the workflow for a focused technical debt reduction effort — assessing debt, prioritizing by business impact, remediating incrementally, and tracking progress. Designed for time-boxed debt sprints where the goal is measurable improvement.

## When To Use

Use this when:
- The team allocates time specifically for debt reduction
- Tech-debt-analyst has produced an assessment and it's time to act on it
- Code health metrics are trending in the wrong direction
- A module needs cleanup before a major feature addition

Do NOT use this when:
- You're in the middle of feature work (finish the feature first)
- The debt is a single specific issue (just fix it, don't run a workflow)
- The codebase is fundamentally unsound (consider a migration-planner assessment first)

## Workflow

### Phase 1: Assess
**Agent**: tech-debt-analyst

1. Analyze the target scope (full codebase, specific module, or specific debt category)
2. Produce a prioritized debt inventory with P0/P1/P2/P3 classifications
3. For each item: estimate interest (ongoing cost), principal (fix effort), and business risk
4. Recommend which items to tackle in this sprint based on available time

**Gate**: Debt inventory and sprint scope must be approved before remediation begins.

### Phase 2: Plan
**Agent**: planner

1. Take the approved debt items and decompose them into ordered tasks
2. Group related items that can be fixed together
3. Assign to appropriate agents (builder, ts-builder, java-builder, refactorer, shell-scripter, etc.)
4. Identify dependencies between items
5. Flag items that need safety nets (tests before refactoring)

**Gate**: Plan approval before implementation.

### Phase 3: Safety Nets
**Agent**: test-writer, ts-test-writer, or java-test-writer (depending on stack)

For each debt item that involves refactoring existing code:
1. Write characterization tests for the current behavior
2. Verify tests pass against the current code
3. Commit tests separately

Skip this phase for items that don't change behavior (dependency updates, documentation, adding missing tests).

### Phase 4: Remediate
**Agent**: builder, ts-builder, java-builder, refactorer, shell-scripter, ci-ops (depending on item)

For each item in the plan:
1. Apply the fix incrementally
2. Run the full test suite after each change
3. Commit with a clear message referencing the debt item
4. Track progress — update the todo list after each item

**Critical rule**: If a fix introduces test failures, revert and reassess. Debt reduction must not create new bugs.

### Phase 5: Verify
**Agent**: code-reviewer

1. Review the complete changeset for each debt item
2. Verify no behavioral regressions
3. Confirm the debt was actually addressed (not just moved around)
4. Run the full test suite, linter, and build

**Gate**: No BLOCKING findings.

### Phase 6: Measure Progress
**Agent**: tech-debt-analyst

1. Re-run the same analysis from Phase 1 on the updated code
2. Compare metrics: test coverage, complexity, dependency vulnerabilities, code smells
3. Document what was fixed, what remains, and recommended next steps
4. Update the debt roadmap

### Phase 7: Ship
**Agent**: git-workflow

1. Create PR(s) with clear descriptions of what debt was addressed
2. Include before/after metrics
3. Note remaining debt items for future sprints

## Principles

- **Time-box ruthlessly**: Debt sprints have a fixed duration. If an item takes longer than estimated, move on to the next one and reschedule.
- **Measure before and after**: Debt reduction without metrics is just refactoring. Track coverage, complexity, vulnerability count, and build times.
- **Ship incrementally**: Don't accumulate a massive changeset. Ship each item as it's completed and verified.
- **Prioritize by business impact**: A clean utility module is nice; a tested payment flow is necessary. Fix what matters most.
- **Don't create new debt while fixing old debt**: If your fix is a hack, it's not a fix.
