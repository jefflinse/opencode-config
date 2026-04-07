---
name: refactoring-workflow
description: Structured refactoring workflow with safety net verification, test-first approach, incremental changes, and regression validation
---

## What This Skill Does

Defines the workflow for safely refactoring existing code — improving structure, readability, and maintainability without changing observable behavior. Ensures every refactoring step is protected by tests and verified against regression.

## When To Use

Use this when:
- Code-reviewer identified structural issues (duplication, poor abstractions, inconsistent patterns)
- Tech-debt-analyst flagged code quality debt for remediation
- The team wants to improve a module before adding new features to it
- A pattern migration is needed (e.g., moving from callbacks to async/await, or from one DI pattern to another)

Do NOT use this when:
- Adding new features (use feature-workflow)
- Fixing bugs (use bug-triage)
- The code has no tests and no tests can be quickly added (add tests first, then refactor)

## Workflow

### Phase 1: Assess
**Agent**: code-reviewer (read-only assessment)

1. Identify the specific code quality issues to address
2. Map the affected files, functions, and their callers
3. Assess existing test coverage for the affected code
4. Determine the refactoring strategy: rename, extract, inline, move, restructure
5. Identify risks: what could break if the refactoring goes wrong?

**Gate**: Assessment must be reviewed and approved before proceeding. If test coverage is insufficient, Phase 2 (safety net) is mandatory.

### Phase 2: Safety Net
**Agent**: test-writer, ts-test-writer, or java-test-writer (depending on stack)

1. Write characterization tests for the existing behavior — tests that document what the code currently does
2. Focus on the public API surface of the code being refactored
3. Include edge cases and error paths in the characterization tests
4. Run all tests and confirm they pass against the current code
5. These tests become the safety net that catches behavioral changes during refactoring

**Acceptance criteria**: All new tests pass. The code under refactoring has sufficient coverage to catch regressions.

### Phase 3: Branch
**Agent**: git-workflow

1. Create a refactoring branch: `refactor/<description>`
2. Commit the characterization tests from Phase 2 as a separate commit (so they can be reviewed independently)

### Phase 4: Refactor
**Agent**: builder, ts-builder, java-builder, or refactorer (depending on scope and stack)

1. Apply the refactoring in small, incremental steps
2. After each step: run the full test suite to verify no behavioral change
3. Each step should be a single, well-defined transformation:
   - Rename for clarity
   - Extract function/method
   - Inline unnecessary abstraction
   - Move to correct package/module
   - Replace pattern (with tests confirming identical behavior)
4. If a test fails after a refactoring step: **stop and revert that step**. Understand why before proceeding.
5. Commit after each successful transformation with a descriptive message

**Critical rule**: Refactoring changes structure, not behavior. If you find a bug while refactoring, note it but do NOT fix it in the refactoring branch — file it separately.

### Phase 5: Verify
**Agent**: code-reviewer

1. Review the complete refactoring changeset
2. Verify no behavioral changes were introduced
3. Confirm the refactoring achieved its goals (reduced duplication, improved clarity, etc.)
4. Run the full test suite one final time

**Gate**: No BLOCKING findings. All tests pass.

### Phase 6: Finalize
**Agent**: git-workflow

1. Squash WIP commits into logical refactoring units (keep characterization tests as a separate commit)
2. Create PR with a clear description of what was refactored and why
3. Note any follow-up work identified during refactoring

## Principles

- **Tests before refactoring**: Never refactor code you can't test. If coverage is insufficient, add tests first.
- **One thing at a time**: Each commit should be a single refactoring transformation. Mixing multiple refactorings makes it impossible to bisect failures.
- **Behavior preservation**: The test suite must pass after every step. If it doesn't, revert. No exceptions.
- **Don't fix bugs while refactoring**: Note them, file them, fix them in a separate branch. Mixing refactoring and bug fixes creates unreviable changesets.
- **Know when to stop**: Refactoring is infinite. Define the scope upfront and stick to it.
