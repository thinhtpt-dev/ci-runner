# ci-runner

Public repo that runs CI/build jobs for private repos on free public-repo minutes.

Flow: push/PR in a private repo -> `dispatch-ci.yml` (ubuntu, ~10s) fires `repository_dispatch`
-> this repo checks out the exact SHA with a PAT, runs the jobs, posts commit statuses
(`ci-runner/<job>`) back, and uploads APK/IPA to a release on the **private** repo.

## Layout
- `.github/workflows/<project>.yml` - per-project entry (plan -> jobs). Add one per project.
- `.github/workflows/{node-checks,android-*,ios-*}.yml` - reusable job workflows.
- `.github/actions/` - `commit-status`, `publish-private-release`.
- `templates/*-dispatch-ci.yml` - copy into the private repo as `.github/workflows/dispatch-ci.yml`.

## Setup
1. Two fine-grained PATs (least privilege):
   - `CI_RUNNER_PAT` (stored only on `ci-runner`): Contents rw, Commit statuses rw, Metadata r on each private repo.
   - `CI_DISPATCH_PAT` (stored on each private repo): Contents rw on `ci-runner` only (needed for `repository_dispatch`).
2. Repo settings on `ci-runner`: Actions -> General -> require approval for all outside collaborators; disable fork PR workflows.
3. Add `EXPO_PUBLIC_REVENUECAT_TEST_KEY` to `ci-runner` (i-am-not-busy e2e).

## Manual run
Actions -> pick the project workflow -> Run workflow (`ref`, `jobs`).

## Security
Logs and run pages here are public. Private code can appear in failure output; never echo
secrets or source. Build artifacts are never uploaded here, only to private releases.
