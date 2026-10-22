# every change to the profile is checked by one command, locally and on every PR

## The ask

The repository standard asks for lint and format, tests, a coverage floor
of 90 % (target 95 %), and security checks, run by one local command and
by CI on every PR, reusing the shared gate in `izam-mohammed/.github`.

What it means here: a profile README with no program code. The security
gate already runs on PRs. The gaps: the pre-commit hooks run only on a
machine that installed them, and most commits come from GitHub's web
editor, so they never run; nothing checks the README's links or that an
SVG is well formed; there is no one local command; the gate is pinned at
an older commit (`ce1b7cc`). A coverage floor is not applicable: there is
nothing to cover.

Built so far: `docs/built/BUILT-2026-10-22-1827.md`.

## Alternatives

### Decision 1: how the pre-commit hooks run on every PR (two-way door)

Searched 2026-10-22: "pre-commit.ci vs pre-commit/action GitHub Actions
run hooks on pull request"; pre-commit.ci's site (pushes fixes to the PR,
a hosted app with write access), the R `precommit` CI vignette (the
`pre-commit/action` is in maintenance-only mode).

| Must | Baseline: local only | `pre-commit run` in a workflow | pre-commit.ci app |
| --- | --- | --- | --- |
| Runs for web-editor commits | no | yes | yes |
| **Survives** | no | **yes** | **yes** |

| Criterion | Weight | `pre-commit run` in a workflow | pre-commit.ci app |
| --- | ---: | ---: | ---: |
| Performance efficiency | 40 | 4 | 5 |
| Security | 30 | 5 | 3 |
| Compatibility | 30 | 5 | 3 |
| **Weighted total** | **100** | **4.60** | **3.80** |

- Efficiency: pre-commit.ci caches hook environments on its side (5); a
  workflow installs them per run, cacheable with `actions/cache` (4).
- Security: the app needs write access to push fixes (3); a workflow with
  `contents: read` only reports (5).
- Compatibility: a workflow sits beside `security.yml` with the same
  pinning and permission rules zizmor checks (5).

**Chosen: `pre-commit run --all-files` in a workflow**, through
`scripts/check.sh`, installed with `uvx` (the action is in maintenance
mode). Revisit if hook installs dominate the run time.

### Decision 2: how the README's links are checked (two-way door)

Searched 2026-10-22: "lychee link checker vs markdown-link-check vs
htmltest 2026"; lychee's docs and its (self-published) comparison table,
OpenTelemetry's site link-checking page.

| Criterion | Weight | Baseline: nothing | lychee | markdown-link-check |
| --- | ---: | ---: | ---: | ---: |
| Performance efficiency | 40 | 5 | 5 | 3 |
| Fit to the ask | 35 | 1 | 5 | 4 |
| Reliability | 25 | 1 | 4 | 3 |
| **Weighted total** | **100** | **2.60** | **4.75** | **3.35** |

- Efficiency: lychee is one async Rust binary (5); markdown-link-check
  runs on Node, installed per run (3).
- Fit: both read Markdown; lychee also reads the HTML inside it, where the
  banner's `<picture>` paths live (5).
- Reliability: lychee retries and caches; its comparison table is its
  own, so scored 4, not 5.

**Chosen: lychee**, offline for repository paths on every PR, and with
the network on the weekly schedule so a flaky site never blocks a PR.
Revisit if the README grows past a handful of links.

## The decision

`scripts/check.sh` runs `pre-commit run --all-files` (decision 1) and
lychee on `README.md` (decision 2). The existing `check-xml` hook from
`pre-commit-hooks` is added for `*.svg`, so a malformed banner fails.
`.github/workflows/check.yml` runs the script on every pull request and
push to `main`, read-only, actions pinned by SHA; a weekly job runs
lychee with the network. `.github/workflows/security.yml` moves to the
shared gate's current commit.

Dependencies: lychee (a pinned release binary in CI), `uv` for
`uvx pre-commit`, both new to CI; no secret; no new permission.

Standards: `docs/standards.md` (the pre-commit hooks and the shared gate
today; `sh scripts/check.sh` from Phase 1).

## Where it lands

| Path | New or edit | Phase | What changes | Firm? |
| --- | --- | --- | --- | --- |
| `scripts/check.sh` | new | 1 | runs the hooks and lychee | expected |
| `.pre-commit-config.yaml` | edit | 1 | `check-xml` on `*.svg` | reuse |
| `.github/workflows/check.yml` | new | 1 | runs the script on PR and push; weekly online links | expected |
| `.github/workflows/security.yml` | edit | 1 | gate pinned at the current `.github` commit | reuse |
| `docs/standards.md` | edit | 1 | the one command, each check | reuse |

*Reused, not rewritten:*

- `.pre-commit-config.yaml`: every hook it has, run in CI as is.
- `pre-commit-hooks` `check-xml`: already a dependency, now on SVGs.
- `izam-mohammed/.github` `security-scan.yml`: still called, repinned.

## Security

What is protected: the profile page and the repository's history. Who
reaches it: pull requests and the owner's web edits.

| Threat | Here | Handled by | Proved by |
| --- | --- | --- | --- |
| Information disclosure: a token committed | a web edit pastes a credential | gitleaks in the hooks and in the gate | the gate's stage 1; the hook run in CI |
| Tampering: a dependency swapped | lychee or a hook replaced upstream | lychee by release and checksum, hooks by `rev`, actions by SHA | zizmor on `check.yml` reports no finding |
| Elevation of privilege: a workflow abused | PR content reaching `run:` | `contents: read`, no `${{ }}` in `run:` | zizmor reports no finding |

Data: none personal beyond what the profile already shows publicly.
Secrets: none used. Checklist: CIS GitHub benchmark, edition confirmed
when built.

## Declined

- Local hooks only (decision 1): never run for web edits.
- pre-commit.ci (decision 1): needs write access to the repository.
- markdown-link-check (decision 2): slower, reads Markdown links only.
- A 90 % coverage floor: no program code.
- A `site/` one-pager: the personal site is its own repository.

## What the tests hold

- A trailing space, a malformed SVG or a fake key in a fixture commit
  each fails `sh scripts/check.sh`.
- A broken repository path in `README.md` fails lychee offline.

## Phases

### Phase 1: the rough cut (needs you: try it)

- [ ] `sh scripts/check.sh` runs `uvx pre-commit run --all-files` and
  lychee offline on `README.md`, exiting non-zero on any failure
  (expected in `scripts/check.sh`), with `check-xml` added for `*.svg` in
  `.pre-commit-config.yaml`.
- [ ] `.github/workflows/check.yml` runs the script on every pull request
  and push to `main`, plus lychee with the network weekly, `contents:
  read`, actions pinned by SHA; `.github/workflows/security.yml` pinned at
  the shared gate's current commit.
- [ ] `docs/standards.md` names `sh scripts/check.sh` and each check.
- [ ] Check: `sh scripts/check.sh` passes locally; on a scratch branch a
  trailing space and a broken path each make it fail; the PR shows
  `check` green and the gate's comment.
- [ ] You try it: edit `README.md` in GitHub's web editor on a branch,
  leave a trailing space, open a PR, and say whether the failure is
  clear. What you want changed goes into Phase 2.

### Phase 2: open the PR

- [ ] The branch pushed and its PR opened against the branch it was cut
  from (or, if one is already open for this branch, its body brought up
  to date) with a Conventional Commit title for the whole change, a body
  naming this plan, what changed and why, and the checks run with their
  result. The PR's URL printed as the evidence. Opened, never merged.
