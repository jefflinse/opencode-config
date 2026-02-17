---
name: bug-triage
description: Structured bug investigation workflow from reproduction through diagnosis, fix, verification, and regression testing
---

## What This Skill Does

Defines a disciplined process for investigating and fixing bugs. Ensures bugs are properly reproduced, root-caused, fixed, and verified — not just patched over.

## When To Use

Use this when:
- A bug report comes in and the cause is not immediately obvious
- A test is failing and the reason is unclear
- Production behavior differs from expected behavior
- A fix was attempted but didn't resolve the issue

## Workflow

### Phase 1: Reproduce
**Agent**: debugger

1. Understand the reported symptom — what was expected vs. what happened
2. Identify the minimal reproduction steps
3. Reproduce the bug locally — if it can't be reproduced, gather more information before proceeding
4. Capture the exact error message, stack trace, or incorrect output
5. Identify the environment: Go version, OS, dependencies, configuration

**Gate**: Bug must be reproducible before proceeding. If it can't be reproduced, document what was tried and ask for more information.

### Phase 2: Diagnose
**Agent**: debugger

1. Read the code at and around the failure point
2. Form hypotheses ranked by likelihood
3. Trace the data flow backward from the failure point
4. Check recent changes (git log, git diff) for likely introduction points
5. Determine: is this a logic error, data issue, race condition, environment problem, or dependency bug?
6. Identify the root cause, not just the proximate trigger

**Deliverable**: A diagnosis report with:
- Symptom description
- Root cause identification (file, function, line)
- Causal chain from root cause to symptom
- Recommended fix approach

**Gate**: Root cause must be identified and documented before attempting a fix.

### Phase 3: Fix
**Agent**: builder

1. Create a fix branch: `fix/<description>`
2. Implement the minimal fix that addresses the root cause
3. Do not refactor, clean up, or "improve" surrounding code in the same change — fix only the bug
4. Run `go build ./...` and `go vet ./...`

### Phase 4: Verify
**Agent**: test-writer

1. Write a test that reproduces the bug — this test must FAIL on the unfixed code
2. Verify the test PASSES with the fix applied
3. Run the full test suite: `go test -race ./...`
4. If the bug was a race condition, run with `-count=100` to verify stability

### Phase 5: Check for Related Issues
**Agent**: debugger

1. Search the codebase for similar patterns that might have the same bug
2. Document any related risks in the PR description
3. If widespread, create follow-up issues rather than expanding the current fix

### Phase 6: Review and PR
**Agents**: code-reviewer, git-workflow

1. code-reviewer reviews the fix for correctness and completeness
2. If the bug was security-related → security-auditor reviews
3. If the bug was concurrency-related → concurrency-reviewer reviews
4. git-workflow creates commit message and PR description

## Principles

- **Reproduce first**: Never attempt a fix without reproducing the bug. "It works on my machine" is not a diagnosis
- **Root cause, not symptoms**: A fix that addresses only the symptom will break again. Find the underlying cause
- **Minimal fix**: Fix the bug. Don't refactor. Don't "improve" adjacent code. Keep the diff reviewable
- **Test the failure**: The regression test must fail without the fix — this proves it actually tests the bug
- **Check for siblings**: If a bug exists in one place, the same pattern error likely exists in others
