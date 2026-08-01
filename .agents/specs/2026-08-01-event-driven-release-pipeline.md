# Event-driven release pipeline

## Request

Apply the release separation proven in `fastypest` to `typeorm-test-db`:

1. Evaluate the commit threshold after each default-branch push and prepare an owner-authored version pull request.
2. Validate the generated release pull request and enable repository-controlled squash auto-merge only for its exact trusted head.
3. Create an immutable GitHub Release after the package version changes on the default branch.
4. Publish to npm from the GitHub `release.published` event so manual and automated GitHub Releases use the same trusted-publisher path.
5. Preserve `.github/workflows/auto-release.workflow.yml` as the npm trusted-publisher workflow identity.

No merge, GitHub Release, tag, npm publication, or deployment is part of this implementation delivery.

## Evidence

- The current preparation workflow is schedule-driven and creates `release/<run-number>` branches.
- The current npm workflow is coupled to release pull-request closure.
- `package.json` is `typeorm-test-db@1.0.57`.
- Pull request `#244` is an existing release pull request created by the previous pipeline and remains outside this implementation branch.
- The repository supports native auto-merge.
- The `admin` environment already owns the release automation credential and npm OIDC boundary.

## Scope

### In scope

- `.github/workflows/prepare-release.workflow.yml`
- `.github/workflows/release-auto-merge.workflow.yml`
- `.github/workflows/create-github-release.workflow.yml`
- `.github/workflows/auto-release.workflow.yml`
- `.github/workflows/dependabot-auto-merge.workflow.yml`
- this task specification

### Out of scope

- merging implementation or release pull requests
- modifying package source or package version
- publishing npm packages
- creating or changing repository secrets, environments, branch rules, tags, or releases
- applying the change to any other repository

## Decision

### Preparation

`prepare-release.workflow.yml` runs for every push to the default branch and through an explicit manual dispatch. It:

- binds work to the exact event commit
- requires the current `package.json` version to have a matching published GitHub Release
- rejects release drift, missing tags, wrong tag targets, or non-ancestor baselines
- leaves an existing trusted release pull request untouched
- counts commits from the current release tag
- preserves the four-commit threshold and explicit force/dry-run controls
- builds and package-smoke-tests before versioning
- creates `release/v<version>` without pushing the local `standard-version` tag
- creates an owner-authored, labeled pull request but does not merge or publish

### Release pull-request trust

`release-auto-merge.workflow.yml` is the sole owner of release pull-request validation and approval. It runs from the trusted default-branch workflow definition and never checks out or executes pull-request code. It validates:

- same repository, owner author, default base, `auto-release` label, and exact `release/v<version>` branch
- current event and live head SHA equality
- exact title contract
- strict and monotonic SemVer
- package identity and version
- exact changed files: `CHANGELOG.md` and `package.json`
- changelog release entry
- absence of a destination tag or GitHub Release
- approval bound to the exact validated head

The owner-scoped `PAT_FINE` enables native squash auto-merge with `--match-head-commit`, allowing repository requirements to remain the merge authority and ensuring the resulting push can trigger downstream workflows.

### GitHub Release

`create-github-release.workflow.yml` runs after each default-branch push and supports owner recovery through an exact version input. It:

- no-ops when `package.json` did not change
- resolves the first-parent commit that introduced the current version, including multi-commit pushes and later same-version metadata changes
- rejects non-SemVer versions, ancestry violations, tag collisions, release target collisions, and metadata drift
- validates the `PAT_FINE` actor as the repository owner
- creates an immutable lightweight tag and generated GitHub Release notes
- verifies tag target and release metadata
- emits a user-authored `release.published` event for the npm workflow

### npm publication

`.github/workflows/auto-release.workflow.yml` retains its filename and `name: 🔄 Auto release`. It now runs only for `release.published` and:

- accepts only non-draft owner-authored releases in the source repository
- validates package name, tag/version equality, strict SemVer, prerelease metadata, and default-branch ancestry
- performs frozen installation, build, archive inspection, and consumer installation smoke testing
- checks npm state idempotently
- publishes through npm OIDC only, using `latest` for stable versions and `next` for prereleases
- verifies package visibility and the destination dist-tag
- has no long-lived npm token fallback

### Dependabot

`dependabot-auto-merge.workflow.yml` returns to one responsibility: Dependabot policy. Patch and minor updates remain eligible; majors remain manual. Release validation is not duplicated there.

## Risks and controls

- **Privileged event handling:** release validation uses a trusted workflow definition and never executes PR content.
- **Secret exposure:** release jobs are restricted to the `admin` environment and validate the authenticated token owner.
- **Stale events:** every approval and merge action rechecks the current head SHA.
- **Event suppression:** event-emitting transitions use the owner token rather than `GITHUB_TOKEN`.
- **Tag replacement:** no force push or tag mutation is permitted.
- **Duplicate publication:** GitHub Release and npm operations are idempotent and fail closed on mismatched existing state.
- **Manual releases:** they must use an owner-authored GitHub Release whose tag points to a default-branch package version.
- **Existing PR #244:** it was created under the previous branch contract. This implementation does not mutate or merge it.

## Acceptance criteria

- A default-branch push evaluates the release threshold.
- Below threshold, preparation exits successfully without a branch or PR.
- At or above threshold, preparation creates only a version/changelog PR.
- The generated PR cannot be approved or queued unless all trust checks pass for the exact current head.
- Merging a version PR causes GitHub Release creation from the exact version-introducing commit.
- Publishing a valid owner GitHub Release invokes the unchanged npm trusted-publisher workflow file.
- Manual and automated GitHub Releases share the same npm path.
- No workflow polls check runs or directly decides which required tests constitute merge eligibility.
- No npm token fallback, force tag update, destructive cleanup, or PR-code execution in privileged jobs exists.

## Checks

- Parse all changed workflow YAML.
- Run `bash -n` over every changed shell block after neutralizing GitHub expressions.
- Run JavaScript syntax checks over `actions/github-script` programs.
- Verify every third-party Action reference is pinned to a full commit SHA.
- Assert the npm workflow filename and display name remain unchanged.
- Assert preparation contains no npm publication, tag push, release creation, direct merge, or check polling.
- Assert GitHub Release creation contains no npm publication.
- Assert npm publication is triggered only by `release.published`.
- Assert no force push, tag mutation, npm token fallback, or destructive failure cleanup exists.
- Run repository CI on the final pull-request head.

## Rollback

Revert the implementation pull request. Existing tags, GitHub Releases, npm versions, and the open release pull request remain immutable external evidence and are not deleted.

## Delivery

- Branch: `agent/refactor-event-driven-release`
- Pull request: pending
- Merge: requires explicit owner approval
- Publication: not performed

## Status

Implementation in progress.
