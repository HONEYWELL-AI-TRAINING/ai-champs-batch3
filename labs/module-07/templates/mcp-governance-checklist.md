# MCP Governance Checklist — <workflow name>

Copy to `specs/<nnn>-<slug>/mcp-governance.md`. The engineering work in an
MCP-enabled workflow is the governance around the connection — not the
connection itself. Fill one row per MCP server the workflow uses.

## Per-server governance

| Server | Why the workflow needs it | Access level | Approval model | Data boundary | Failure behaviour |
|--------|---------------------------|--------------|----------------|---------------|-------------------|
| filesystem | | read-only / read-write, scoped to `${workspaceFolder}` | auto / per-call prompt | repo only, no `..` escape | |
| postgres | | **read-only** role | per-call prompt | one DB, no writes | degrade: use last-known schema |
| github | | repo scope, no admin | per-call prompt | this repo's issues/PRs only | |

## Tool selection

- Tools **enabled** for this workflow (and why each is needed):
- Tools **disabled / not added** (and why they're out of scope):

## Permissions & approvals

- [ ] No server has broader access than the workflow step requires.
- [ ] Write-capable tools (if any) require an explicit per-call approval.
- [ ] The Postgres role is read-only (verify: a write attempt fails).
- [ ] Tokens are supplied via input prompts / secret storage — **never** in
      `.vscode/mcp.json`. (`git diff` shows no secret.)

## Context / state boundaries

- What context each workflow step is allowed to see:
- What is reset between steps (so step 3 doesn't inherit step 1's full dump):
- Where the workflow's own outputs are written (and not leaked back into a
  prompt verbatim):

## Failure handling & security

| Failure | Detection | Workflow response |
|---------|-----------|-------------------|
| MCP server won't start | | halt, report, don't fake the step |
| Server times out mid-workflow | | retry once, then halt |
| Auth failure | | halt, prompt for re-auth, never proceed with partial access |
| Tool returns unexpected/huge output | | truncate + summarise, don't paste raw into the next step |

- [ ] A server failing does not cause the workflow to silently skip a step or
      fabricate a result.

## Sign-off

- [ ] Every server has a named reason, a least-privilege access level, and a
      failure behaviour.
- [ ] No secret is committed.
- [ ] The workflow halts safely on any governance violation.
