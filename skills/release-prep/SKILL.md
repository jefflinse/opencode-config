---
name: release-prep
description: Release preparation workflow including changelog generation, version bump, tagging, release notes, and deployment checklist
---

## What This Skill Does

Guides the process of preparing a release — from deciding the version number through changelog, tagging, and creating release artifacts. Ensures nothing is missed before shipping.

## When To Use

Use this when preparing a tagged release of the project. This covers both regular releases and hotfix releases.

## Workflow

### Phase 1: Determine Version
**Agent**: git-workflow

1. Review all changes since the last release: `git log <last-tag>..HEAD --oneline`
2. Determine version bump using semver:
   - **Major** (v2.0.0): Breaking changes to public API
   - **Minor** (v1.1.0): New features, no breaking changes
   - **Patch** (v1.0.1): Bug fixes only
3. If using conventional commits, this can be automated by scanning commit types:
   - `BREAKING CHANGE:` in footer → major
   - `feat:` → minor
   - `fix:`, `perf:` → patch

### Phase 2: Changelog
**Agent**: git-workflow

1. Generate changelog entries from commits since last release
2. Follow Keep a Changelog format:
   ```markdown
   ## [v1.2.0] - YYYY-MM-DD
   ### Added
   - Feature description (#PR)
   ### Changed
   - Change description (#PR)
   ### Fixed
   - Bug fix description (#PR)
   ### Removed
   - Removed feature description (#PR)
   ### Security
   - Security fix description (#PR)
   ```
3. Write entries from the user's perspective, not the developer's
4. Link to PRs and issues for each entry
5. Include migration instructions for any breaking changes

**Gate**: Present the changelog to the user for review before proceeding. The changelog will be included in the release, so it must be accurate.

### Phase 3: Pre-Release Checks
**Agent**: builder

Run the full verification suite:

1. **Tests**: `go test -race ./...` — all must pass
2. **Lint**: `golangci-lint run` — no new issues
3. **Vet**: `go vet ./...` — clean
4. **Vulnerabilities**: `govulncheck ./...` — no known vulnerabilities
5. **Build**: `go build ./...` — compiles cleanly
6. **Docker**: `docker build .` — image builds successfully (if applicable)
7. **Integration tests**: Run if available

Acceptance criteria: All checks pass. Do not proceed with any failures.

**Gate**: If any check fails, present the failures to the user. Options: "Fix and re-run checks", "Abort release". Do not proceed with known failures.

### Phase 4: Version Bump
**Agent**: builder

Update version references in:
- Source code version constants (if any)
- README badges (if version is shown)
- Makefile version defaults (if hardcoded)
- Any version-stamped configuration

Acceptance criteria: `go build ./...` still compiles after version changes.

### Phase 5: Tag and Release
**Agent**: git-workflow

1. Commit changelog and version bump: `chore(release): prepare v1.2.0`
2. Create annotated tag: `git tag -a v1.2.0 -m "Release v1.2.0"`
3. Push: `git push origin main --tags`
4. Create GitHub release using `gh release create`:
   - Use the changelog section as the release body
   - Attach any binary artifacts
   - Mark as pre-release if applicable

### Phase 6: Post-Release
**Agent**: builder

1. Verify CI/CD pipeline completed successfully for the tag
2. Verify published artifacts are accessible (container images, binaries)
3. If the project is a library, verify it's fetchable: `go get <module>@v1.2.0`

**Gate**: Present the release verification results to the user. Include any manual steps the user needs to take (announcing the release, notifying stakeholders). These are human actions that cannot be automated.

## Release Checklist (Quick Reference)

```
[ ] Version number determined (semver)
[ ] Changelog written and reviewed
[ ] All tests pass
[ ] Lint clean
[ ] No known vulnerabilities
[ ] Build succeeds
[ ] Docker image builds (if applicable)
[ ] Version references updated
[ ] Committed and tagged
[ ] GitHub release created
[ ] CI/CD completed for tag
[ ] Artifacts verified
```

## Hotfix Releases

For urgent fixes to production:
1. Branch from the release tag: `git checkout -b fix/<description> v1.0.0`
2. Apply the minimal fix
3. Run full verification suite
4. Tag as patch release: `v1.0.1`
5. Cherry-pick to main if applicable
