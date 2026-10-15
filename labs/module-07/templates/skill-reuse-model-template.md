# Skill Reuse & Governance Model — <skill name>

Copy to `.github/skills/<skill>/REUSE.md`. A reusable engineering skill is a
**cross-project asset with an owner and a version** — not a snippet someone
pasted once.

## Identity

- Skill name:
- Version: (semver — matches `SKILL.md` frontmatter and `VERSION`)
- Owner(s): (team or named people — see `OWNERS`)
- Status: experimental / supported / deprecated

## What it does

One paragraph. What problem it solves, what it takes as input, what it emits.

## Where it's meant to be reused

- This repo: (which specs / features)
- Other projects: (what has to be true for it to apply — same layering? a
  `spec.md`? a constitution?)
- Explicit non-applicability: (where NOT to use it)

## How another project adopts it

1. Copy `.github/skills/<skill>/` into the target repo.
2. Adjust: (list the repo-specific bits — paths, the rubric's project rules).
3. Pin the version in the adopting repo's notes.

## Change management

- Proposing a change: open a PR touching `.github/skills/<skill>/`, tag an owner.
- Review bar: an owner approves; `CHANGELOG.md` updated; `VERSION` bumped
  (patch = wording, minor = new checks, major = output format change).
- Breaking changes: announce to known adopters (list them below).

## Known adopters

| Repo / project | Version in use | Contact |
|----------------|----------------|---------|
| this repo | | |

## Retirement

What replaces it, and the deprecation window, if it's ever removed.
