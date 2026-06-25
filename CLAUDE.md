# CLAUDE.md — workflows

- **Primary Goal**: Follow all instructions in this local `CLAUDE.md` as hard constraints.

---

## Environment & Shell

- **Core platform**: GitHub Actions.
- **Shell**: Bash 5+ (in runner shell scripts).

---

## File Conventions

- **Workflows**: Yaml workflow definitions inside `.github/workflows/`.
- **Pre-commit configuration**: Managed with `.pre-commit-config.yaml`.

---

## Quality & Tooling

- **Testing / Linting**: `pre-commit run --all-files` (runs actionlint, yamllint, markdownlint, and gitleaks checks)

---

## Git & PR Workflow

- **Branching**: Never commit to `main`. Use prefixes: `feat`, `fix`, `docs`, `chore`, `ci`, `build`, `refactor`, `perf`, `test`, `style`, `revert`.
- **Commits**: Conventional format `type: subject` (lowercase type, max 100 chars, no period, blank line before body).
- **PRs**: Create draft PR immediately after pushing branch. Do not mark ready without permission.
- **Merge**: Verify release label (`release:{major,minor,patch}`), merge, delete remote branch, pull main, delete local branch.
- **CI/CD**: Prefer self-hosted/`ubuntu-slim`. Keep workflows harmonized via shared workflows repo.

---

## Review Rules

- **Responses**: Answer every comment.
- **MUST FIX**: Always resolve.
- **CONSIDER**: Evaluate once. Implement if valuable, else reject with short reasoning. Ignore repeats on unchanged code.
- **Completion**: Done when all MUST FIX resolved and comments answered. Do not cycle on repeated CONSIDERs; open issues to defer.
