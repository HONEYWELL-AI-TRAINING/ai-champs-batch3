# Workflow Run Log — <workflow name>

Copy to `specs/<nnn>-<slug>/workflow-run-log.md`. One row per step of the
MCP-enabled workflow, each run recorded with its governance and cost.

## Run

- Date / feature under analysis:
- Workflow: repository analysis → spec validation → test execution → result analysis
- Model(s) used:

## Per-step log

| Step | MCP tools used | Approvals prompted | Context sent (est. tokens) | Model requests | Output (where written) | Governance ✅ |
|------|----------------|--------------------|-----------------------------|----------------|------------------------|:------------:|
| 1 · Repository analysis | filesystem | | | | `…/repo-analysis.md` | |
| 2 · Spec validation | filesystem (+ spec-validator skill) | | | | `…/spec-validation.md` | |
| 3 · Test execution | filesystem, postgres (ro) | | | | `…/test-run.md` | |
| 4 · Result analysis / PR evidence | filesystem, github | | | | PR comment / `…/pr-evidence.md` | |
| **Total** | | | | | | |

## Context/state boundary check

- [ ] Step 3 did **not** carry step 1's full repo dump — only what it needed.
- [ ] Each step's output was written to a file and *referenced* by the next
      step, not pasted verbatim.
- [ ] Context was reset between unrelated steps.

## Cost (feeds Module 11)

- Total estimated tokens for the run:
- vs a naive "one giant prompt with everything" version (estimate):
- Where the boundaries saved the most:

## Outcome

- Did the workflow reach a correct, useful result end to end? (Y/N + notes)
- Any step that had to halt on a governance rule (which, why):
