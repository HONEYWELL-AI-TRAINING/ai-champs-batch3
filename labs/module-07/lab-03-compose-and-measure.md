# Lab 03 — Compose the Workflow + Skill, and Measure the Cost

**Time:** ~35 min · **Surface:** Copilot Chat (Agent) + terminal ·
**Prereq:** Labs 01–02 complete

## Objective

Run the four-step MCP workflow **with `spec-validator` plugged into step 2**, as
one governed pass — logging the token/cost of each step so the effect of
context/state boundaries is visible. Finalise the governance and reuse model.

## Why it matters for the enterprise

An agent workflow that works but costs 10× what it should is not a win. The
context/state boundaries from Lab 01 aren't just a security measure — they're a
cost control. This lab makes that concrete: the same workflow, measured, so
Module 11's token-economics model has real numbers, and the reuse model is
documented well enough that another team can pick it up.

## Step 1 — Set up the run log

**macOS / Linux**

```bash
cp ../module-07/templates/workflow-run-log-template.md specs/002-task-soft-delete/workflow-run-log.md
```

**Windows (PowerShell)**

```powershell
Copy-Item ..\module-07\templates\workflow-run-log-template.md specs\002-task-soft-delete\workflow-run-log.md
```

## Step 2 — Run the composed workflow, logging each step

Start the MCP servers (Lab 01). Run the four steps again, but this time:

- **Step 2 uses the skill:** `Follow .github/skills/spec-validator/SKILL.md to
  validate specs/002-task-soft-delete/ using repo-analysis.md as input.`
- After **each** step, before moving on, record in `workflow-run-log.md`:
  - which MCP tools it used and how many approvals it prompted;
  - the context you attached (estimate tokens with `tools/estimate-tokens.sh` on
    the attached files, plus the IDE's chat context gauge if shown);
  - roughly how many model requests it took;
  - where the output was written.

Between steps, **start a fresh chat** and attach only the previous step's output
*file* — not the whole conversation. That's the state boundary; the log should
show step 3's context is much smaller than "everything so far".

## Step 3 — Fill the boundary + cost sections

- **Boundary check:** confirm each step's context was scoped (step 3 didn't
  carry step 1's repo dump). If it did, that's a finding — note it and how you'd
  fix it (write step 1's output to a file, reference it).
- **Cost:** total the per-step token estimates. Then estimate the naive version —
  one prompt with the spec + all implementation files + all test files + "check
  everything" — and compare. The boundaried workflow should be materially
  cheaper on every turn, and the difference compounds across a real project.

## Step 4 — Finalise governance + reuse

- **`mcp-governance.md`** (from Lab 01): add a line per workflow step confirming
  the tools it's allowed, and confirm the "halt on failure" behaviour held this
  run.
- **`REUSE.md`** (from Lab 02): fill the "Known adopters" table (this repo,
  v1.0.0) and the adoption steps, tested by imagining a second Task Board-style
  repo picking it up.
- Write a one-paragraph `specs/002-task-soft-delete/workflow-summary.md`: what
  the workflow produces, what it costs, who owns the skill, and when a team
  should run it (before every spec-feature PR? weekly? on request?).

## Step 5 — Commit

```bash
git add specs/002-task-soft-delete/workflow-run-log.md specs/002-task-soft-delete/workflow-summary.md .github/skills/spec-validator/REUSE.md specs/002-task-soft-delete/mcp-governance.md
git commit -m "Module 07 Lab 03: composed MCP workflow run log + cost + reuse model"
git push
```

## Verify

- [ ] `workflow-run-log.md` has all four steps with tools, approvals, token
      estimate, and output location.
- [ ] Step 2's row shows `spec-validator` was used and produced a rubric-format
      report.
- [ ] The boundary check confirms context was scoped per step (or names the
      leak and the fix).
- [ ] The cost comparison shows the boundaried workflow beats the naive one, with
      numbers.
- [ ] `REUSE.md` "Known adopters" and adoption steps are filled;
      `mcp-governance.md` is finalised.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Can't get per-step token numbers | Estimate from attached-file size (`estimate-tokens`) + a fixed ~200 for the prompt. The ratio between steps is what matters, not precision. |
| Step 3 context is as big as step 1 | You continued the same chat. New chat per step; attach the prior step's *file*. |
| Skill output doesn't fit the run log | The log wants a pointer to the report file, not the report itself — keep the log terse. |
| Naive-version estimate feels unfair | Be honest: include every file a "check the whole feature" prompt would need. It's usually 5–15× the boundaried total. |

## Recap — Module 07 complete

The repo now carries:

```
.vscode/mcp.json                              # Lab 01 — filesystem + ro-postgres + github
.github/skills/spec-validator/                # Lab 02 — versioned, owned skill
│   ├── SKILL.md RUBRIC.md VERSION OWNERS CHANGELOG.md REUSE.md
specs/002-task-soft-delete/
├── mcp-workflow.md  mcp-governance.md         # Lab 01 — the workflow + its guardrails
├── repo-analysis.md spec-validation.md test-run.md pr-evidence.md   # Lab 01/03 outputs
├── workflow-run-log.md  workflow-summary.md   # Lab 03 — measured, owned
└── spec-validation-report.md                  # Lab 02 — skill output
```

A governed MCP workflow, a reusable skill with an owner and a version, and a
cost record. **Module 08** takes the pull request this produces evidence for and
runs it through an automated quality gate and an LLM-as-Judge, with human
escalation.
