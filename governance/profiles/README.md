# Profiles

One `<name>.yaml` per profile, same shape as `../defaults.yaml`. A repo gets the
profile mapped from its GitHub primary language (`languages` in `../repos.yaml`)
unless it pins one with `profile:`. A missing file means "no extra config".

No language-specific profile is needed yet: required checks don't differ by
language. The existing exceptions are per repo, as overrides in `../repos.yaml`
(check names for `workflows`, an extra check for `pc-price-tracker`, repos
that skip the ruleset). Add e.g. `go.yaml` when Go repos need a check others
don't.
