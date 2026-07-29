# Fix repository automation runtime

## Request

Audit the merged repository automation after real Dependabot executions, correct common failures, create the required labels and preserve the existing release actor contract.

## Evidence

- Repository-level label creation can fail in no-checkout jobs without explicit repository context.
- Eligible Dependabot pull requests were approved by `github-actions[bot]`; the required contract is owner approval followed by GitHub Actions auto-merge.
- Existing release pull requests are authored with the owner's Actions `PAT_FINE` and must continue to be approved by GitHub Actions for the exact current head.
- Automated review identified missing approval-actor verification, dry-run label mutation and a concurrent label-creation race.
- `fastypest` uses `requires-manual-qa` color `E99695` and `auto-release` color `FEF2C0` with its npm release description.

## Decision

- Require `PAT_FINE` as a Dependabot repository secret with Pull requests read/write.
- Preserve the existing Actions `PAT_FINE`, package validation, npm publishing and release workflow semantics.
- Resolve the Dependabot PAT login with `gh api user` and fail unless it equals `GITHUB_REPOSITORY_OWNER` (`juanjoGonDev`).
- Let `github.token` approve only trusted owner-authored release pull requests for their exact head SHA and enable dependency auto-merge.
- Surface a precise 403 error when **Allow GitHub Actions to create and approve pull requests** is disabled.
- Require repository settings **Allow auto-merge** and **Allow GitHub Actions to create and approve pull requests**.
- Synchronize both automation labels only outside dry-run mode and tolerate only the specific concurrent `422 already_exists` race.
- Add explicit repository context, fork guards and minimum cache permissions.

## Acceptance criteria

- Eligible Dependabot updates are approved by `juanjoGonDev`, never by a bot or another PAT owner.
- Existing owner-authored release PRs are approved by `github-actions[bot]` for the exact current head.
- Production majors remain manual and require current non-bot write maintainer approval.
- Manual dry runs make no repository mutation and concurrent label creation cannot abort processing.
- Existing package publication behavior remains unchanged.
- Missing or incorrectly owned Dependabot credentials fail with precise guidance.

## Validation

- Workflow YAML and repository checks are validated by pull-request CI; changed Actions remain pinned by immutable SHA.
- Full actor-identity validation requires the configured Dependabot secret and subsequent Dependabot/release events after merge.

## Rollback

Revert the corrective pull request. The branch does not publish, release, deploy or merge.

## Delivery status

Implemented on `agent/fix-automation-runtime` and delivered through a normal corrective pull request. Existing release semantics are preserved.
