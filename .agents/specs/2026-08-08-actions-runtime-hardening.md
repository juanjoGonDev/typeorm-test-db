# Actions runtime hardening

## Request

Correct the shared required-QA Dependabot merge automation and prevent the npm trusted-publisher consumer smoke test from inheriting publisher authentication configuration.

## Evidence

- The shared GraphQL auto-merge path fails when GitHub already reports an approved PR as `clean`.
- Refreshing a behind Dependabot branch that changes `.github/workflows/*` can require the sensitive Workflows permission.
- The npm publisher uses `actions/setup-node` with `registry-url`, which exports a temporary npm user config containing `${NODE_AUTH_TOKEN}`; package-manager consumer smoke tests must not depend on that publisher-only configuration when OIDC intentionally has no long-lived npm token.
- This repository already uses `PAT_FINE` from protected environment `admin` for trusted release writes.

## Decision

- Keep `.github/workflows/auto-release.workflow.yml` at its existing path and retain OIDC with `id-token: write`; add no npm token fallback.
- Set `NPM_CONFIG_USERCONFIG=/dev/null` only for the temporary package-install smoke step so the subsequent publish step retains the setup-node publisher configuration.
- Preserve and revalidate exact-head, non-bot, write-maintainer QA approval.
- Reuse the protected `admin` Actions `PAT_FINE`, validate it as the repository owner, and use it for live branch/merge transitions.
- Squash-merge an exact approved `clean` head; otherwise enable repository auto-merge when available.
- Never grant Workflows permission. Workflow-changing behind PRs require a trusted manual update and fresh approval.

## Acceptance

The publisher identity and OIDC contract remain unchanged, smoke install is isolated from `NODE_AUTH_TOKEN`, clean approved majors do not fail, and workflow-file refresh never escalates permission.

## Checks

Parse changed workflow YAML, syntax-check shell and `github-script` programs, verify immutable Action SHAs, and rely on pull-request CI as authority.

## Rollback

Revert the corrective pull request. No release, npm publish or merge is performed by this branch.

## Delivery status

Implemented on `agent/fix-actions-runtime-20260808`; pending pull-request CI and explicit owner merge approval.
