# Profiles

One `<name>.yaml` per profile, same shape as `../defaults.yaml`. A repo gets the
profile mapped from its GitHub primary language (`languages` in `../repos.yaml`)
unless it pins one with `profile:`. A missing file means "no extra config".

Nothing language-specific is needed yet: every repo currently uses the same
required checks. Add e.g. `go.yaml` when Go repos need a check others don't.
