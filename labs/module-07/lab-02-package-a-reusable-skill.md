# Lab 02 — Package a Reusable Engineering Skill

**Time:** ~40 min · **Surface:** file editing + Copilot Chat / CLI ·
**Prereq:** Lab 01 complete (you ran the spec-validation step by hand)

## Objective

Turn the workflow's spec-validation step into a **reusable engineering skill** —
`spec-validator` — with a rubric, a version, named owners, and a changelog. Test
it against both the comments and soft-delete features and document how another
project adopts it.

## Why it matters for the enterprise

A prompt that lives in one person's chat history helps one person once. A skill
with a version, an owner, and a changelog is a **cross-project asset**: the team
codifies "how we check an implementation against its spec" once, and every repo
gets the same check, improved through PRs, not folklore.

## Background — skill vs the Module 02 skill

Module 02's `port-endpoint` skill was task instructions + scripts. This one adds
the **governance wrapper** a shared asset needs:

```
.github/skills/spec-validator/
├── SKILL.md        # name, description, version (frontmatter) + the procedure
├── RUBRIC.md       # the explicit criteria the validation scores against
├── VERSION         # e.g. 1.0.0  (matches SKILL.md frontmatter)
├── OWNERS          # who approves changes
├── CHANGELOG.md    # what changed in each version
└── REUSE.md        # how another project adopts it (from the template)
```

---

## Step 1 — Scaffold

**macOS / Linux**

```bash
mkdir -p .github/skills/spec-validator
cp ../module-07/templates/skill-reuse-model-template.md .github/skills/spec-validator/REUSE.md
```

**Windows (PowerShell)**

```powershell
New-Item -ItemType Directory -Force -Path .github\skills\spec-validator | Out-Null
Copy-Item ..\module-07\templates\skill-reuse-model-template.md .github\skills\spec-validator\REUSE.md
```

## Step 2 — `SKILL.md`

```md
<!-- .github/skills/spec-validator/SKILL.md -->
---
name: spec-validator
version: 1.0.0
description: >-
  Validate an implemented feature against its spec.md — every acceptance
  criterion checked for coverage and correctness, plus constitution compliance.
  Use before opening a PR, or when reviewing whether a change matches its spec.
---

# Spec Validator

## When to use
- Before opening a PR for a spec-driven feature.
- When reviewing whether an implementation actually matches `spec.md`.

## Inputs
- `specs/<feature>/spec.md` (+ `acceptance-checklist.md` if present)
- The implementation files
- `.specify/memory/constitution.md`

## Procedure
1. Extract every acceptance criterion (`AC-xx`) from `spec.md`, verbatim.
2. For each AC: locate the code that implements it and the test that proves it.
   Classify as **met / partially met / not met / no test**.
3. Check the implementation against every constitution principle. Note
   violations with `file:line`.
4. Find behaviour in the code that no acceptance criterion covers (unspecified
   work → flag for the spec or removal).
5. Score against `RUBRIC.md`. Produce the report in the RUBRIC's output format.

## Output
A Markdown report: per-AC table, constitution findings, unspecified-behaviour
list, rubric scores, and a single verdict (ready / not ready) with the blockers.
Do not modify code — this is validation only.
```

## Step 3 — `RUBRIC.md`

```md
<!-- .github/skills/spec-validator/RUBRIC.md -->
# Spec-Validator Rubric  (v1.0.0)

Score each 0–2. The feature is **ready** only if every criterion is 2 (or a
documented, accepted exception).

| # | Criterion | 0 | 1 | 2 |
|---|-----------|---|---|---|
| R1 | AC coverage | criteria with no implementation | all implemented, some untested | every AC implemented AND has a passing test |
| R2 | AC correctness | an AC is implemented wrongly | minor deviation, documented | matches the spec exactly |
| R3 | Constitution compliance | a principle is violated | a borderline case, noted | fully compliant |
| R4 | No unspecified behaviour | significant behaviour not in the spec | trivial extra, flagged | nothing shipped that isn't specified |
| R5 | Traceability | can't map code → AC | partial map | every change traces to an AC |

## Output format
```
## Spec Validation — <feature>  (spec-validator vX.Y.Z)
### Acceptance criteria
| AC | Criterion | Code (file:line) | Test | Status |
### Constitution findings
### Unspecified behaviour
### Rubric
| R1 | R2 | R3 | R4 | R5 | 
### Verdict: READY / NOT READY
Blockers:
```
```

## Step 4 — Governance files

```bash
# .github/skills/spec-validator/VERSION
1.0.0
```

```md
<!-- .github/skills/spec-validator/OWNERS -->
# Owners — spec-validator skill
# Changes require approval from one of:
- @your-github-handle           (primary)
- <a teammate or the AI Champions guild>   (backup)
```

```md
<!-- .github/skills/spec-validator/CHANGELOG.md -->
# Changelog — spec-validator

## 1.0.0 — 2026-09-02
- Initial version. Rubric R1–R5. Built in Module 07 from the MCP workflow's
  spec-validation step.
```

Fill in `REUSE.md` from the template — the "how another project adopts it"
section is the point.

## Step 5 — Test the skill

Run it against **both** features. In Copilot Chat / CLI:

```
Follow .github/skills/spec-validator/SKILL.md to validate the soft-delete
feature (specs/002-task-soft-delete/). Then do the same for the comments feature
(specs/001-task-comments/). Save each report next to its spec as
spec-validation-report.md.
```

Sanity-check the reports: do the rubric scores match what you know about each
feature? If `spec-validator` misses something obvious, that's a v1.0.1 — fix the
rubric or the procedure, bump `VERSION`, add a `CHANGELOG` line.

## Step 6 — Commit

```bash
git add .github/skills/spec-validator specs/00*/spec-validation-report.md
git commit -m "Module 07 Lab 02: spec-validator reusable skill (v1.0.0)"
```

## Verify

- [ ] `.github/skills/spec-validator/` has SKILL.md (with `version`), RUBRIC.md,
      VERSION, OWNERS, CHANGELOG.md, REUSE.md.
- [ ] `VERSION` matches the `version:` in `SKILL.md` frontmatter.
- [ ] Running the skill produces a report in the RUBRIC's output format for both
      features.
- [ ] `REUSE.md` explains concretely how another repo adopts and adjusts it.
- [ ] Any fix you made bumped the version and added a changelog line.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Skill report is freeform, ignores the rubric format | Reference `RUBRIC.md` explicitly in `SKILL.md` step 5, and paste the output template. |
| Agent doesn't discover the skill | Invoke by path (`Follow .github/skills/spec-validator/SKILL.md`). Auto-discovery varies by Copilot version. |
| Rubric scores feel arbitrary | Anchor each level with a concrete description (done above); re-run. |
| Version drift | Make `VERSION`, the frontmatter, and the newest `CHANGELOG` heading agree — check all three on every change. |

## Recap & carry-forward

You have a versioned, owned skill that any project can adopt. **Lab 03**
composes the MCP workflow and this skill into one run, logs the token cost per
step, and finalises the reuse/governance model.
