# Version policy

## Tags

- `@v1` — moving major-version tag. Bug-fixes and backward-compatible
  features advance the tag after the smoke-test workflow passes on `main`.
- `@v2`, `@v3`, … — cut for breaking changes (input rename/removal,
  output rename, behaviour change). `v1` continues to receive bug-fixes
  for **90 days** after `v2` is cut, then enters maintenance.
- Commit SHA — callers may pin to a specific commit for highest
  assurance. Recommended for the highest-risk repo (money-maker);
  optional elsewhere. Dependabot is configured per-caller to auto-PR
  SHA bumps.

## Canary order

When a `v1` change is published (i.e. the `v1` tag is force-moved):

1. **ai-hosting** consumes the new `v1` first (canary).
2. After 24 hours of clean ai-hosting runs (or sooner if the user
   accepts the risk), promote to **vigilos**, **game-fabric**, **atlas**
   in parallel.
3. After the second wave is clean, promote to **thundera** and
   **money-maker** last.

Under "ASAP" mode (audit decision 2026-05-01), the soak windows above
are skipped and all repos consume the new tag immediately. The canary
is still ai-hosting in case of a rapid rollback.

## Rollback

```bash
# Revert v1 to its previous SHA. All callers using @v1 get the
# rollback on their next CI run.
git -C lmcoelho/.github tag -f v1 <previous-sha>
git -C lmcoelho/.github push --force-with-lease origin v1
```

Force-push to a major-version tag is the documented exception to the
"never force-push to main/production" rule. Audit trail is preserved
in `git reflog` and the CHANGELOG.

## Change classification

Every PR to this repo declares a risk class in its title:

- `feat: …` — a new input or workflow (LOW; semver minor; tag advances)
- `fix: …` — bug-fix (LOW; semver patch; tag advances)
- `refactor: …` — internal change with no behaviour difference (LOW)
- `chore: …` — docs / metadata (LOW)
- `BREAKING: …` — input rename, removal, or behaviour change. Cuts a
  new major version. Existing `v1` callers are not auto-updated.

Smoke-test must be green for any PR to merge. CODEOWNERS gates `main`
to require explicit review of every change to a reusable workflow.
