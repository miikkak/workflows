# Security Policy

## Supported Versions

Only the latest state of `main` is supported. There are no version tags; consumers pin to a
specific commit SHA and update on their own schedule (typically via Renovate).

| Version               | Supported          |
| --------------------- | ------------------ |
| latest commit on main | :white_check_mark: |
| older pinned commits  | :x:                |

## Reporting a Vulnerability

Please follow these steps if you discover a security vulnerability in this project:

### Do Not

- **Do not** open a public GitHub issue for security vulnerabilities
- **Do not** disclose the vulnerability publicly until it has been addressed

### Do

1. **Report privately** via [GitHub Security Advisories](https://github.com/miikkak/workflows/security/advisories/new) <!-- markdownlint-disable-line MD013 -->
2. **Include in your report:**
   - Description of the vulnerability
   - Steps to reproduce the issue
   - Potential impact
   - Suggested fix (if you have one)

3. **Response timeline:**
   - You should receive an acknowledgment within 48 hours
   - We'll provide a detailed response within 7 days
   - We'll work with you to understand and fix the issue
   - We'll release a fix as soon as possible

## Scope

These are reusable GitHub Actions workflows, called via `uses:` from other repositories'
workflow files. Their blast radius is meaningfully larger than a typical single-purpose repo: a
compromised workflow here runs with whatever permissions and secrets each _calling_ repo grants
it, across every repo that references this one. The most security-sensitive logic is:

- The fork-PR runner isolation in `pr-check.yml`/`ci-cd.yml`/`code-review.yml` (the
  `detect-runner` jobs), which route untrusted fork PRs off self-hosted runners - a bug here
  could let an untrusted fork PR execute on infrastructure it shouldn't reach
- Any `actions/github-script` block that accepts PR/issue-derived text (titles, bodies, branch
  names) and uses it in a shell command or file path without sanitization - this is the classic
  GitHub Actions script-injection pattern
- Anywhere a workflow uses `secrets.SEMANTIC_RELEASE_TOKEN`/`secrets.PULL_REQUEST_TOKEN` or
  similar cross-repo PATs, since those carry more privilege than the default `GITHUB_TOKEN`

If you find a way a PR (especially a fork PR) could get a workflow here to execute unintended
commands, leak a secret, or run on infrastructure it shouldn't have access to, that's exactly the
kind of thing to report.

## Security Best Practices

When consuming these workflows:

- Pin `uses: miikkak/workflows/.github/workflows/<name>.yml@<sha>` to a specific commit SHA, not
  a floating branch - Renovate can keep the pin current for you
- Grant only the `secrets`/`permissions` a given workflow's own `inputs:`/`secrets:` block
  actually declares needing - don't blanket-grant `secrets: inherit` if a narrower pass-through
  works for your use case
- Review a workflow's diff before bumping its pinned SHA, the same way you would for any other
  third-party Action

## Security Scanning

This project uses automated security scanning:

- **`gitleaks`** (via pre-commit) to catch accidentally-committed secrets before they're pushed
- **`actionlint`**/**`shellcheck`** on every PR to catch unsafe shell patterns in embedded `run:`
  blocks (e.g. unquoted expansions, injectable `${{ }}` interpolation directly into a shell
  command instead of via an `env:` variable)
- **Trivy** (`security-scan.yml`, run on this repo via `self-ci.yml`) for dependency
  vulnerability scanning of `package-lock.json`/`requirements.txt`
- **Renovate** for automated dependency updates

## Other Automated Review

Every pull request also gets an AI code review. This is a general correctness/quality review,
not a vulnerability scanner - don't rely on it as a substitute for the security scanning above.

## Disclosure Policy

- Security issues are fixed in private before public disclosure
- After a fix is released, we publish a security advisory
- We credit reporters in the advisory (unless they prefer anonymity)

## Past Security Advisories

No security advisories have been published yet.

## Contact

For security-related questions or concerns, please use the reporting method above rather than
public channels.
