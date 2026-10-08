# workflows

Reusable GitHub Actions workflows shared across this account's repositories: CI/CD, release
automation, linting, security scanning, and dependency-freshness checks. Consumer repos call
these with `uses:` instead of duplicating the same job logic in each one.

## About this project

This was built with heavy Claude Code assistance — most of the implementation is AI-generated,
with the design and review driven by me. It's dogfooded across roughly 25 of my own repositories
in daily use (Java/Velocity plugins, Go CLIs, Bash tooling, Ansible), so it sees real production
traffic, not just its own test suite. Read the source and file issues if something looks off.

## Requirements to consume these workflows

- A `self-hosted` runner reachable by your repo, with labels matching whatever a given workflow's
  `runner`/`test-runner` input expects by default (see each workflow's `inputs` block) - or pass
  your own `runner: '["ubuntu-latest"]'`-style input to use GitHub-hosted runners instead
- `secrets: inherit` (or explicitly passed secrets) on the calling job, for any workflow that
  declares a `secrets:` block

Every workflow below is called like:

```yaml
jobs:
  ci:
    uses: miikkak/workflows/.github/workflows/ci-cd.yml@<pinned-sha>
    with:
      # inputs specific to that workflow - see its own `inputs:` block
```

Pin to a commit SHA (not a floating tag) in your own repo's workflow files - Renovate can keep
that pin current for you (see [`renovate-config`](https://github.com/miikkak/renovate-config)).
`@main` also works, but only pin to it for local testing of an unreleased change; never leave a
consumer repo tracking it.

Only add a `secrets:` block if the workflow you're calling actually declares one in its own
`workflow_call.secrets` (most don't) - grant just the named secret it asks for, e.g.
`secrets: { PULL_REQUEST_TOKEN: ${{ secrets.PULL_REQUEST_TOKEN }} }`, rather than a blanket
`secrets: inherit` that hands it every secret your repo has.

## Reusable workflows

| Workflow                      | Purpose                                                                             |
| ----------------------------- | ----------------------------------------------------------------------------------- |
| `pr-check.yml`                | Pre-commit hooks on every PR, with fork-PR runner isolation                         |
| `ci-cd.yml`                   | PR validation: changed-files detection, test suite, optional Docker Buildx          |
| `cd.yml`                      | Post-merge pipeline: `semantic-release` versioning/tagging driven by release labels |
| `code-review.yml`             | AI code review with a configurable model fallback chain                             |
| `security-scan.yml`           | Scheduled Trivy/govulncheck-based dependency vulnerability scanning                 |
| `secret-scan.yml`             | gitleaks secret scanning: PR range (merge gate), push range, scheduled full history |
| `super-linter.yml`            | `super-linter`-based multi-language linting                                         |
| `release-label.yml`           | Enforces a `release:major`/`release:minor`/`release:patch`/`release:none` label     |
| `check-release-label.yml`     | Lighter-weight release-label presence check for other workflows to depend on        |
| `release-deploy.yml`          | Deploys a release to configured target servers over SSH                             |
| `release-tarball.yml`         | Builds and attaches a release tarball to a GitHub Release                           |
| `release-go.yml`              | Tests, cross-builds (`-trimpath`), archives and releases a Go binary                |
| `dependabot-automerge.yml`    | Auto-approves and merges eligible Dependabot PRs                                    |
| `gradle-lockfile-refresh.yml` | Regenerates a stale `gradle.lockfile` on PRs that need it                           |
| `cache-cleanup.yml`           | Prunes stale GitHub Actions caches on a schedule                                    |
| `precommit-updates.yml`       | Opens a PR when pinned pre-commit hook revisions have updates available             |
| `dependency-check.yml`        | Polls for upstream releases/security fixes not covered by Renovate/Dependabot       |
| `trigger-package-rebuild.yml` | Opens a rebuild PR to pick up Alpine `apk` security updates in a container image    |
| `validate-tag.yml`            | Validates a manually-pushed tag matches the expected format before release          |
| `sync-to-server-config.yml`   | Mirrors specific files from a source repo into my private `server-config` repo      |

`self-ci.yml` and `self-verify-toolchains.yml` are this repo's _own_ CI. They run on PRs against
`main` (`self-verify-toolchains.yml` only when `security-scan.yml`/`release-tarball.yml` change)
and aren't meant to be `uses:`-called from elsewhere - included for transparency, not for
external consumption.

This repo deliberately doesn't tag or publish releases of itself - consumers pin to a commit SHA
(see above), not a version tag, so a self-referential release/tarball would have no consumer and
nothing to attach beyond the repo's own config files (`.github/` itself, the actually useful
part, would have to be excluded from any such tarball to avoid duplicating what git already
gives you).

## Secret scanning

Layered so no single bypass (`git commit --no-verify`, a skipped hook, a direct push) leaves a
secret unnoticed:

1. **Local** - the `gitleaks` pre-commit hook (scans staged changes; skippable with `--no-verify`).
2. **PR gate** - `secret-scan.yml` on `pull_request` scans the PR's commit range; mark the check
   required in branch protection/rulesets so it blocks merge.
3. **Push** - the same workflow on `push` scans each push's new commits within minutes, even
   before a PR exists, and opens a `security` issue on a hit.
4. **Scheduled** - the same workflow on `schedule` scans full history of every ref daily and
   opens/updates the issue. `security-scan.yml` (Trivy, `vuln,secret`) is a second, tree-only
   opinion.

Consumers add one thin caller (pin the SHA like every other reusable workflow here):

```yaml
name: Secret Scan

on:
  pull_request:
    branches: [main]
  push:
    branches: ["**"]
  schedule:
    - cron: "30 3 * * *"
  workflow_dispatch:

permissions:
  contents: read
  issues: write

jobs:
  secret-scan:
    uses: miikkak/workflows/.github/workflows/secret-scan.yml@<sha> # main
```

Findings are redacted; any hit means the credential must be rotated. False positives go in a
repo-level `.gitleaksignore` (fingerprints) or `.gitleaks.toml` (`[extend] useDefault = true`
plus an allowlist) - gitleaks picks both up from the repo root, so there's no shared config to
drift.

## Design notes

- Every reusable workflow accepts a `runner`/`test-runner` input (a JSON array string, e.g.
  `'["self-hosted","linux","generic"]'`) so a consumer without self-hosted runners can override
  it to GitHub-hosted instead.
- `pr-check.yml`, `ci-cd.yml`, and `code-review.yml` each run a `detect-runner` job that routes
  untrusted fork PRs (`github.event.pull_request.head.repo.full_name != github.repository`) to
  ephemeral GitHub-hosted `ubuntu-slim` runners instead of self-hosted infrastructure, regardless
  of what the consumer repo's own `runner` input says - this can't be overridden by a fork PR,
  by design.
- Workflow-to-workflow pins use a commit SHA with a trailing `# main`/`# vX.Y.Z` comment
  (`uses: actions/checkout@<sha> # v7`) so the human-readable ref stays visible next to the pin
  Renovate is actually tracking.

## License

[MIT License](LICENSE) - Copyright (c) 2026 Miikka Karhuluoma
