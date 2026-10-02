# Repo governance config

Desired state for the GitHub settings of every `miikkak/*` repo: merge-button
settings, security toggles, Actions workflow permissions, labels and the
ruleset on `main`. It is applied by `repo-governance-sync.sh` (in the
`scripts` repo, deployed to `/usr/local/sbin/`), so a change here can be
rolled out to all repos in one run, and drift is detectable at any time.

## Layers

For each repo the effective config is merged from:

```text
defaults.yaml  ->  visibility/<public|private>.yaml  ->  profiles/<profile>.yaml  ->  repos.yaml: repos.<name>.config
```

- **Visibility** is read live from GitHub, so a repo that goes public picks up
  the public layer (secret scanning, push protection, CodeRabbit) by itself.
  Private repos don't manage `security_and_analysis` (not available on this
  account tier).
- **Profile** comes from the repo's primary language (`languages:` in
  `repos.yaml`) unless the repo pins `profile:`. See `profiles/README.md`.
- **Repo override** is `config:` under the repo in `repos.yaml`.

Merge rules: objects deep-merge, arrays are replaced, a key set to `null`
removes it (this is how an override drops a default required check). Keys no
layer mentions are never touched on the target repo.

Labels and required checks are **maps** (keyed by label name / check name) so
layers can add or remove single entries. Labels are only created or
corrected; deleting one requires listing it in `labels_prune`.

A required check can carry `requires_file: <path>`: it is only enforced once
that file exists on `main`, otherwise every PR would wait for a check that
never reports.

## Repos not listed in `repos.yaml`

They are governed anyway (defaults + visibility + language profile), so a new
repo needs no config entry. `repos.yaml` is only for exceptions: `skip:`
sections, a pinned profile, or overrides.

## Usage

```bash
repo-governance-sync.sh audit                 # drift report for all repos, exit 1 on drift
repo-governance-sync.sh audit <repo>          # one repo
repo-governance-sync.sh apply --dry-run <repo>
repo-governance-sync.sh apply <repo>          # a new repo: first run aligns everything
repo-governance-sync.sh apply --all
repo-governance-sync.sh show <repo>           # effective config
repo-governance-sync.sh export <repo>         # current state as config YAML
```

The config directory defaults to `~/src/workflows/governance` (override with
`--config-dir` or `$GOVERNANCE_DIR`).

## Changing policy

1. Edit the config in a PR here.
2. After merge: `repo-governance-sync.sh audit` shows which repos differ,
   `apply --all` rolls it out.
