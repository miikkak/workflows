# CLAUDE.md — workflows

- **Primary Goal**: Follow all instructions in this local `CLAUDE.md` as hard constraints.

---

## Environment & Shell

- **Core platform**: GitHub Actions.

---

## File Conventions

- **Workflows**: Yaml workflow definitions inside `.github/workflows/`.
- **Pre-commit configuration**: Managed with `.pre-commit-config.yaml`.

---

## Quality & Tooling

- **Testing / Linting**: `pre-commit run --all-files` (runs actionlint, yamllint, markdownlint, and gitleaks checks)
