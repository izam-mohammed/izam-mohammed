# the profile repo's hook pins stay current and it has a reporting path

## The ask

The owner asked, in his words, to "have dependabot in all of my repos" and
"have proper security.md file for all of my repos". This repository gets
no licence (the owner listed it as none). It is the GitHub profile README
repository.

Read on `origin/main` (`8e00ea4`) on 22/10/2026:

- `.github/dependabot.yml` covers `github-actions` only.
- `.pre-commit-config.yaml` pins gitleaks and pre-commit-hooks by `rev`,
  and nothing updates those pins.
- There are no package manifests; the content is `README.md` and two SVG
  files.
- There is no `SECURITY.md`. The repository is public.

## Alternatives

### Decision 1: which update bot (two-way door)

Searched 22/10/2026: "dependabot vs renovate", "dependabot pre-commit
ecosystem"; read the GitHub Docs pages "Dependabot options reference" and
"Supported ecosystems and repositories", and the Renovate docs
"Installing and onboarding".

| Must | Baseline: Dependabot | Renovate |
| --- | --- | --- |
| Runs with no app or server to install | yes | no (needs the Renovate GitHub app or a self-hosted runner) |
| Covers actions and pre-commit | yes | yes |
| **Survives** | **yes** | no |

| Criterion | Weight | Baseline: Dependabot |
| --- | ---: | ---: |
| Performance efficiency | 40 | 4 |
| Reliability | 30 | 4 |
| Compatibility | 30 | 5 |
| **Weighted total** | **100** | **4.30** |

- Compatibility 5: already configured here; the shared gate reads its
  alerts.
- Efficiency 4: one grouped PR per ecosystem per week, 7 day cooldown.

Sensitivity: only a manager Dependabot cannot handle would reopen this.
Risk: none beyond a hook bump that a PR review catches.

**Chosen: Dependabot**, extended. Revisit if a manifest Dependabot cannot
read is added.

## The decision

- `.github/dependabot.yml` keeps its `github-actions` block and adds
  `pre-commit` at `/`, weekly, 7 day cooldown, one group, prefix
  `chore(deps)` (decision 1).
- `SECURITY.md` is new and short: latest commit on `main` only, report
  through GitHub private vulnerability reporting, what happens next, and
  scope (`README.md`, the SVG files, the workflows).

New dependencies: none. Permissions: none.

Security: no code, input or secret changes; the change adds update PRs and
a reporting policy.

## Where it lands

| Path | New or edit | Phase | What changes | Firm? |
| --- | --- | --- | --- | --- |
| `.github/dependabot.yml` | edit | 2 | `pre-commit` added | fixed |
| `SECURITY.md` | new | 3 | security policy | fixed |

*Reused, not rewritten:*

- `.github/dependabot.yml`: the existing `github-actions` block stays.
- `/home/izam/fun/decisionsmith/SECURITY.md`: the shape the policy follows.

## Declined

- Renovate: needs an app or a runner installed (decision 1).

## What the tests hold

`.github/dependabot.yml` parses as YAML and each block has a manifest
where it points. `SECURITY.md` names no invented email, response time or
version.

## Phases

### Phase 1: topics

- [x] Repository topics kept (`awesome-readme`, `github`, `profile`,
  `profile-readme`, `readme`) and `github-profile` added with
  `gh repo edit`.
- [x] Check: `gh repo view --json repositoryTopics` lists them.

### Phase 2: Dependabot

- [x] `.github/dependabot.yml` adds `pre-commit` at `/`, weekly, 7 day
  cooldown, grouped.
- [x] Check: the file parses with `python3 -c 'import yaml'`.

### Phase 3: security policy

- [ ] `SECURITY.md` has supported versions, the private vulnerability
  reporting link, what happens next and scope.
- [ ] Check: `grep` finds no em dash and no email address in it.

### Phase 4: open the PR

- [ ] The branch pushed and its PR opened against `main` (or, if one is
  already open for this branch, bring the open PR's body up to date) with
  the title `chore: dependabot and security policy`, a body naming this
  plan, what changed and why, the ecosystems covered and the checks run.
  The PR's URL printed as the evidence. Opened, never merged.
