# the profile opens with a banner that follows the viewer's github theme

## The ask

Every repository should have a minimal README with either a banner (a
light and a dark SVG, per the `github-banner` skill) or a UI screenshot
near the top. This one is the GitHub profile README: what people see on
github.com/izam-mohammed. Today it is two lines of text and a link to the
personal site, nothing visual.

Built so far: `docs/built/BUILT-2026-10-22-1827.md`.

## Constraints

- Keep it minimal and in the owner's voice: lowercase, casual, dropped
  g's ("tryin' things, breakin' things").
- The banner follows the `github-banner` skill: two files, no animation,
  concept and function only, a concept not in its `concepts-used.md`.

## Alternatives

### Decision 1: what sits at the top of the profile (two-way door)

Searched 2026-10-22: "GitHub profile README banner light dark picture
prefers-color-scheme"; GitHub Docs "Quickstart for writing on GitHub"
(the `<picture>` element with `prefers-color-scheme`, a fallback `<img>`
and alt text).

| Criterion | Weight | Baseline: text only | Light and dark banner | Screenshot of the site | The old `chat.svg` |
| --- | ---: | ---: | ---: | ---: | ---: |
| Performance efficiency | 40 | 5 | 4 | 2 | 3 |
| Fit to the ask | 35 | 1 | 5 | 3 | 2 |
| Maintainability of the result | 25 | 5 | 4 | 2 | 3 |
| **Weighted total** | **100** | **3.60** | **4.35** | **2.35** | **2.65** |

- Efficiency: two small text SVGs through GitHub's image proxy (4); a
  raster screenshot is tens of kilobytes and blurs on zoom (2);
  `chat.svg` animates, so it keeps the page busy (3).
- Fit: the standard asks for a banner or screenshot; text only fails it
  (1). A screenshot of a deliberately plain site says little (3).
  `chat.svg` animates, which the banner skill forbids, and it follows
  the OS theme, not GitHub's (2).
- Maintainability: a screenshot goes stale each time the site changes
  (2); a banner says what the profile is about and changes rarely (4).

**Chosen: a light and dark banner.** It meets the ask, follows the
viewer's GitHub theme through `<picture>`, and stays true as the site
changes. Revisit if the profile grows a section a screenshot would show
better.

## The decision

Two SVGs, `assets/banner-light.svg` and `assets/banner-dark.svg`, built
with the `github-banner` skill's `build.py` from a template and palette
kept outside the repository, shown above the existing two lines through a
`<picture>` element with alt text (decision 1). The existing text stays,
in the owner's voice. The concept is new and goes into the skill's
`concepts-used.md` after it ships; the build picks three concepts about
what this profile is (a person who tries and breaks things, and hides the
embarrassing parts in private repositories) and the strongest one wins,
asking the owner if none is clearly best.

Dependencies and permissions: none in the repository; the skill's
`build.py` and `preview.sh` run on the builder's machine.

Standards: `docs/standards.md` (the pre-commit hooks and the shared
security gate; `sh scripts/check.sh` once `plans/02-quality-gates.md`
lands).

Security: no new input, data, secret, dependency or permission; the SVGs
are self-contained, with no scripts or external URLs (the skill's lint
refuses them).

## Where it lands

| Path | New or edit | Phase | What changes | Firm? |
| --- | --- | --- | --- | --- |
| `assets/banner-light.svg` | new | 1 | light banner | fixed |
| `assets/banner-dark.svg` | new | 1 | dark banner | fixed |
| `README.md` | edit | 1 | `<picture>` above the two lines | reuse |

The two SVG paths are fixed because the skill names them.

*Reused, not rewritten:*

- `github-banner` skill `scripts/build.py` and `scripts/preview.sh`: the
  build, lint and preview, used as they are.
- The scoop bucket's README (PR `#1` there): its `<picture>` markup shape.
- `chat.svg`: considered and not reused (decision 1).

## Declined

- Text only (decision 1): fails the ask.
- A screenshot of the site (decision 1): goes stale, says little.
- The old `chat.svg` (decision 1): animated and theme-blind.

## What the tests hold

- `build.py` exits 0 on both themes: no overlap, nothing off canvas, no
  animation, no external resource, a `<title>` and `<desc>`, contrast
  warnings read.
- Both files are previewed and look right; the README shows the dark one
  in GitHub's dark theme and the light one in its light theme.

## Follow-ups

- `chat.svg` and `github-contribution-grid-snake.svg` are no longer shown
  by the README. Removing them is the owner's call, not done here.

## Phases

### Phase 1: the rough cut (needs you: try it)

- [x] Three concepts for what this profile is about, none repeating the
  skill's `concepts-used.md`, and the strongest picked (or put to you).
- [x] The two banners land at `assets/banner-light.svg` and
  `assets/banner-dark.svg`, built by the skill's `build.py` with no
  `ERROR` and every `WARN` read, and previewed with `preview.sh`.
- [x] `README.md` shows them through `<picture>` with alt text above the
  existing two lines, unchanged.
- [x] The concept is appended to the skill's `concepts-used.md`.
- [ ] Check: `pre-commit run --all-files` passes (or `sh scripts/check.sh`
  if `plans/02-quality-gates.md` landed first), and the branch's README
  on GitHub shows the banner.
- [ ] You try it: open the branch's README on github.com in light and
  dark themes, and say whether the joke lands and reads as you.

### Phase 2: open the PR

- [ ] The branch pushed and its PR opened against the branch it was cut
  from (or, if one is already open for this branch, its body brought up
  to date) with a Conventional Commit title for the whole change, a body
  naming this plan, what changed and why, and the checks run with their
  result. The PR's URL printed as the evidence. Opened, never merged.
