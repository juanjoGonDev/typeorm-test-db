# Repository automation standardization

## Request

Audit the active workflows against `fastypest`, harden cache and Dependabot automation, preserve the package release contract, and open one PR without merging or publishing.

## Evidence

- Default branch: `main`.
- Stack: pnpm, Node.js, TypeScript, and multiple database test services.
- The repository is actively maintained through Dependabot and release PRs.
- Existing cache cleanup was shell-specific and not empty-safe.
- Existing `pull_request_target` automation checked out PR code and used mutable Action tags.
- Dependabot-triggered `pull_request_target` workflows receive a read-only token and no secrets, so privileged Dependabot automation must not depend on repository secrets.
- `auto-release.workflow.yml` owns the existing npm publication contract and already uses its established credentials.

## Decision

- Group weekly npm and GitHub Actions updates after a seven-day cooldown.
- Replace cache deletion with a generic API workflow and manual dry-run by default.
- Use `pull_request` plus the repository-scoped `GITHUB_TOKEN` for Dependabot approval, labels, and auto-merge; no PR code is checked out.
- Preserve the trusted `auto-release` PR approval job using the repository-scoped `GITHUB_TOKEN`.
- Require a current write-permission maintainer approval for production majors, bound to the current head SHA.
- Use the scheduled default-branch workflow and `GITHUB_TOKEN` for required-QA branch updates and auto-merge.
- Leave `auto-release.workflow.yml` unchanged: this PR neither copies nor removes npm publication behavior.

## Acceptance

- [x] Existing release approval behavior remains available.
- [x] No privileged workflow checks out pull-request-controlled code.
- [x] External or stale approvals cannot unlock production majors.
- [x] Cache cleanup is global and empty-safe.
- [x] No new repository secret or variable is required by this PR.
- [x] No release, npm publication, deployment, or merge is performed by this task.

## Validation

The proposed YAML parsed successfully. Dependabot, cache, required-QA, CI, package, and release contracts were inspected. Pull-request CI remains the runtime gate.

## Repository settings

Enable repository auto-merge and `Allow GitHub Actions to create and approve pull requests`. Required status checks must remain enforced on `main`. Existing release credentials remain governed by `auto-release.workflow.yml` and are not changed here.

## Risks and rollback

The new dependency workflows cannot approve or queue pull requests if the repository settings above are disabled. The existing npm release workflow retains separate legacy security debt. Revert this PR to roll back; no package or runtime data requires recovery.

## Delivery

- Branch: `agent/chore-repository-automation`
- Base: `main`
- Merge/release/deploy/publish: not authorized

## Status

Implemented on the task branch; pull-request checks and repository settings remain to be verified.
