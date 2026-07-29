# Fix repository automation runtime

## Request

Audit the merged repository automation after real Dependabot executions, correct common failures and preserve the existing release actor contract.

## Evidence

- The shared Dependabot workflow attempted repository-level label creation without an explicit repository and can fail because it intentionally performs no checkout.
- Eligible Dependabot pull requests were approved by `github-actions[bot]`; the required contract is owner approval followed by GitHub Actions auto-merge.
- Existing release pull requests are already created with `PAT_FINE` as the repository owner and must continue to be approved by GitHub Actions.
- `fastypest` uses `requires-manual-qa` color `E99695` and `auto-release` color `FEF2C0`.

## Decision

- Require `PAT_FINE` as a Dependabot repository secret for owner-authored dependency approvals.
- Preserve the existing Actions `PAT_FINE`, package validation, npm publishing and release workflow without changing its release semantics.
- Validate secrets inside shell steps rather than referencing secrets directly in `if:` expressions.
- Let `github.token` approve trusted owner-authored release pull requests and enable dependency auto-merge.
- Add explicit repository context to label commands, synchronize manual-QA metadata, add fork guards and remove unused cache permissions.

## Acceptance criteria

- Eligible Dependabot updates are approved by `juanjoGonDev` and queued by GitHub Actions.
- Existing owner-authored release pull requests are approved by `github-actions[bot]` for their exact head SHA.
- Production majors are labeled correctly and require a current human maintainer approval.
- Existing package publication behavior remains unchanged.
- Missing Dependabot credentials fail with explicit setup guidance.

## Validation

- Workflow YAML parsed with a non-coercing loader.
- Actions remain pinned by immutable SHA in changed workflows.
- Pull-request CI validates repository checks.
- Full actor-identity validation requires the configured Dependabot secret and subsequent automation events.

## Rollback

Revert the corrective pull request. The branch does not publish, release or deploy.

## Delivery status

Implementation complete on `agent/fix-automation-runtime`; pull request and CI validation pending.
