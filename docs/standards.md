# izam-mohammed: what every change passes

There is no single local command yet. Locally, `pre-commit run
--all-files` runs the hooks below once installed with `pre-commit
install`; in CI only the security gate runs. `plans/02-quality-gates.md`
lands `sh scripts/check.sh` and runs it on every PR.

## Toolchain

| What | Version | Pinned in |
| --- | --- | --- |
| pre-commit hooks | gitleaks v8.30.1, pre-commit-hooks v6.0.0 | `.pre-commit-config.yaml` |
| Shared security gate | `izam-mohammed/.github` at `ce1b7cc` | `.github/workflows/security.yml` |

## Checks

| Check | Tool | Config | Command | Passing means |
| --- | --- | --- | --- | --- |
| Secret scan (local) | gitleaks, `detect-private-key` | `.pre-commit-config.yaml` | `pre-commit run --all-files` | no findings |
| Hygiene (local) | pre-commit-hooks: large files, merge and case conflicts, end of file, trailing whitespace | `.pre-commit-config.yaml` | same | no changes made |
| Security gate (CI) | gitleaks (blocks), osv-scanner, semgrep `p/bash`, zizmor (report) | `.github/workflows/security.yml` | runs on every PR and weekly | stage 1 green |
| Dependency audit | osv-scanner in the gate | | | nothing to scan: no packages |
| Tests | not applicable | | | no program code |

## Constraints

- A profile README rendered by GitHub: no scripts, images served through
  GitHub's proxy, so anything shown must be self-contained.
- Text is in the owner's voice: lowercase, casual, dropped g's.

## Conventions

- Commits: Conventional Commits for recent changes (`ci:`), lowercase,
  no attribution lines.

## CI

`.github/workflows/security.yml` on every pull request and Mondays 03:17
UTC; `permissions: {}` at the top, granted per job. Dependabot updates
actions weekly.

## Decided

- The shared gate over a local one, from `repo-template` (commit
  `e1e9ec7`).
