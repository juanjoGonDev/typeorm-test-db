# Repository automation standardization

## Request

Audit the active workflows against `fastypest`, harden cache and Dependabot automation, preserve the package release contract, and open one PR without merging or publishing.

## Evidence

- Default branch: `main`.
- Stack: pnpm, Node.js, TypeScript, and multiple database test services.
- The repository is actively maintained through Dependabot and release PRs.
- Existing cache cleanup is shell-specific and not empty-safe.
- Existing `pull_request_target` automation checks out PR code and uses mutable Action tags.
- `auto-release.workflow.yml` creates an owner-authored PR labeled `auto-release`, waits for a GitHub Actions approval, runs checks, merges, and publishes the npm package.

## Decision

- Group weekly npm and GitHub Actions updates after a seven-day cooldown.
- Replace cache deletion with a generic API workflow and manual dry-run by default.
- Remove checkout and PR-controlled execution from privileged automation; pin introduced Actions by immutable SHA.
- Preserve the trusted `auto-release` approval job so the existing release workflow is not broken.
- Auto-approve patch/minor updates and development-only majors. Production majors require a current approval from a reviewer with repository write permission.
- Leave `auto-release.workflow.yml` unchanged: this PR neither copies nor removes npm publication behavior.

## Acceptance

- [x] Existing release approval behavior remains available.
- [x] No privileged workflow executes pull-request-controlled code.
- [x] External or stale approvals cannot unlock production majors.
- [x] Cache cleanup is global and empty-safe.
- [x] No release, npm publication, deployment, or merge is performed by this task.

## Validation

The proposed YAML parsed successfully. Dependabot, cache, required-QA, CI, package, and release contracts were inspected. Pull-request CI remains the runtime gate.

## Risks and rollback

GitHub Actions approvals, auto-merge, branch protection, and the automation token must remain configured consistently with the release workflow. The existing npm release workflow retains separate legacy security debt. Revert this PR to roll back; no package or runtime data requires recovery.

## Delivery

- Branch: `agent/chore-repository-automation`
- Base: `main`
- Merge/release/deploy/publish: not authorized

## Status

Implemented on the task branch; pull-request checks and repository settings remain to be verified.
