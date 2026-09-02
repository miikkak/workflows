# Contributing to workflows

Thank you for considering contributing to this project! We welcome contributions from the
community.

## Getting Started

1. Fork the repository
2. Clone your fork locally
3. Create a new branch for your changes
4. Make your changes
5. Test your changes
6. Submit a pull request

## Forking this Repository

If you're creating your own fork of this project, you'll need to update repository-specific
references:

### Files to Update

1. **`renovate.json`**
   - Update the `extends` entry pointing at `github>miikkak/renovate-config` to your own Renovate
     config, or drop it if you don't use Renovate

2. **`SECURITY.md`**
   - Update the GitHub Security Advisories URL to point to your fork

3. **`.github/workflows/*.yml`**
   - `sync-to-server-config.yml` and `trigger-package-rebuild.yml` are specific to my own
     infrastructure (a private `server-config` repo, Alpine-based container images) - drop or
     rewrite them if you don't have an equivalent use case
   - `cd.yml`'s "Checkout release tooling" step checks out `miikkak/workflows` by name to fetch
     this repo's own `package.json`/`.releaserc.json` when _this repo itself_ is the one being
     released - update that repository reference if you fork this repo under a different name

This list isn't exhaustive - grep for `miikkak` across the repo to catch anything else hardcoded
to the canonical repository, then update it to point to your fork's location.

## Development Requirements

- Node.js (for `semantic-release` and its plugins - see `package.json`)
- Python 3 (for `dependency-check.yml`'s tooling - see `requirements.txt`)
- Git
- Pre-commit hooks
- [`actionlint`](https://github.com/rhysd/actionlint) and
  [`shellcheck`](https://www.shellcheck.net/) (used by the pre-commit hooks and CI to validate
  workflow YAML and embedded shell)

## Code Quality Standards

This project maintains high code quality standards using automated tooling:

### Pre-commit Hooks

All commits must pass pre-commit hooks. Install the `pre-commit` tool itself first (see
[pre-commit.com's installation guide](https://pre-commit.com/#installation) for other methods,
e.g. your OS package manager):

```bash
pip install pre-commit
```

Then register the hooks for this repo:

```bash
pre-commit install
```

The hooks cover `actionlint`, `shellcheck` (via `.github/shellcheck-wrapper`), `yamllint`,
`markdownlint`, `gitleaks`, and basic file hygiene (trailing whitespace, line endings, valid
YAML/JSON, no accidentally-committed large files).

### Commit Message Format

We use [Conventional Commits](https://www.conventionalcommits.org/) — `semantic-release`
(triggered by the PR's release label, not commit message parsing) relies on the convention for
consistency. Your commit messages should follow this format:

```text
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`

Examples:

- `feat(code-review): add a new fallback model tier`
- `fix(pr-check): correct fork-PR detection for a renamed base repo`
- `docs: document the sync-to-server-config inputs`
- `chore(deps): update actions/checkout digest`

### Code Style

- Standard YAML/Bash conventions, enforced by `actionlint`/`shellcheck`/`yamllint` rather than
  manual review
- Every reusable workflow's `inputs:`/`secrets:` blocks should have a clear `description`
- Comments explain _why_ a decision was made (not what the workflow already says) - especially
  for GitHub Actions permission/token quirks that aren't obvious from the YAML alone (several
  existing comments document real incidents; match that standard for new ones)

## Branch Workflow

- **Never commit directly to `main`**
- Create feature branches from `main`
- Name branches descriptively (e.g., `feat/new-review-tier`, `fix/fork-pr-detection`)
- All work must be submitted via Pull Request
- PRs require passing CI/CD checks

## Testing

There's no traditional test suite - workflow correctness is validated by:

```bash
actionlint -shellcheck .github/shellcheck-wrapper
```

The wrapper works around a version incompatibility between `actionlint`'s shellcheck invocation
syntax and modern `shellcheck` releases (see the comment at the top of the wrapper script) - it's
not something you invoke directly.

Since these are reusable workflows, the most meaningful test is often exercising the change from
an actual consumer repo's PR before merging.

## CI/CD Pipeline

All PRs trigger automated CI/CD that includes:

- Pre-commit checks (`actionlint`, `shellcheck`, `yamllint`, file hygiene)
- `PR Validation (self)` - this repo's own workflows validating themselves
- AI code review

Your PR must pass all checks before it can be merged.

## Pull Request Process

1. Ensure your branch is up-to-date with `main`
2. Push your branch to your fork
3. Open a Pull Request against `main`
4. Fill in the PR template with:
   - Clear description of changes
   - Link to any related issues
   - Test plan
5. Wait for CI/CD checks to complete
6. Address any review feedback
7. Once approved and checks pass, a maintainer will merge your PR

## What to Contribute

We welcome contributions in these areas:

- Bug fixes
- New reusable workflows or inputs that generalize well
- Documentation improvements
- Security improvements

Please open an issue first if you're planning a major change to discuss the approach - especially
for anything touching `pr-check.yml`/`ci-cd.yml`/`code-review.yml`'s fork-PR runner isolation
logic, since that's a real security boundary, not just convenience code.

## Getting Help

- Open an issue for bug reports, feature requests, or questions
- Check existing issues and PRs to avoid duplicates

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
