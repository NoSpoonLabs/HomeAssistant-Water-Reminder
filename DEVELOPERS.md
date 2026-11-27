# Developer guide

## Versioning

Versions follow `<year>.<month>.<increment>` (e.g. `2026.10.2`), without zero-padding the month.

- The increment starts at `1` each month: `2026.11.1`, `2026.11.2`, `2026.12.1`.
- Pre-releases for testing append `b<n>`: `2026.11.1b1`, `2026.11.1b2`.
- `version` in `custom_components/water_reminder/manifest.json` must equal the release tag.

## Local setup

```bash
python3.13 -m venv .venv && . .venv/bin/activate
pip install -r requirements_test.txt ruff
ruff check . && ruff format --check .
pytest
```

## Testing a development version in Home Assistant

The Marketplace only installs releases (or the default branch while no release exists),
not arbitrary branches. To test unreleased code:

1. Set the manifest `version` on the branch to a pre-release, e.g. `2026.11.1b1`, and push.
2. GitHub → **Releases → Draft a new release**: tag `2026.11.1b1`, target = the branch,
   tick **Set as a pre-release**, publish.
3. Home Assistant → **Settings → Marketplace** → *Water Reminder* → turn on **Pre-release**,
   update, restart Home Assistant.
4. After testing, turn **Pre-release** off again.

## Release

Only the *release maintainer* can release: the user named in the `if:` of the job in
`.github/workflows/release.yaml`, who is also the code owner of `.github/` (`.github/CODEOWNERS`).

1. Merge the PR(s) into `main`.
2. GitHub → **Actions → Release → Run workflow** on `main`, as the release maintainer.
   - *Version*: leave empty for the next version (`2026.11.1` → `2026.11.2`, new month →
     `2026.12.1`, `2026.11.2b1` → `2026.11.2`), or enter one; a `b<n>` suffix makes it a pre-release.
   - *Notes*: optional text placed above the generated notes.
3. Approve the waiting job (environment `release`).
4. The workflow commits `Release <version>` to `main`, pushes the tag and creates a **draft**
   release with notes generated from the merged pull requests (grouped by label, see
   `.github/release.yml`; label `skip-changelog` leaves a PR out).
5. GitHub → **Releases**: review the draft, edit if needed, **Publish**.

One-time setup (repository settings, not in the code, so a pull request cannot change them):

1. **Settings → Environments → New environment** `release`: *Required reviewers* = the release
   maintainer, *Deployment branches* = `main` only.
2. As the release maintainer, create a fine-grained personal access token: resource owner = the
   organization, only this repository (the organization must allow fine-grained tokens),
   permission *Contents: Read and write*. Save it as secret `RELEASE_TOKEN` of the environment
   `release`, not as a repository secret.
3. Ruleset for `main`: pull request required, squash merge only, *Require review from Code Owners*,
   *Repository admin* in the bypass list (the token pushes the release commit; the maintainer
   merges their own pull requests this way).

## CI

Three workflows in `.github/workflows/`, each on every push to a pull request and on
pushes to `main`:

- `secret-scan.yaml`: gitleaks and absolute local paths on the new commits.
- `validate.yaml`: Marketplace validation + hassfest; also daily on `main` (04:23 UTC).
- `tests.yaml`: ruff + pytest; also daily on `main` (04:37 UTC).

`release.yaml` runs only by hand, see [Release](#release).
