# Lab 01 — An MCP-Enabled Workflow, Under Governance

**Time:** ~45 min · **Surface:** VS Code Agent mode + `.vscode/mcp.json` ·
**Prereq:** Module 07 [set-up](README.md#set-up--do-this-once-before-lab-01) done

## Objective

Add the MCP servers a real engineering workflow needs (filesystem, read-only
Postgres, GitHub), run the four-step workflow **repository analysis → spec
validation → test execution → result analysis** against the soft-delete feature,
and document the governance layer that makes it safe.

## Why it matters for the enterprise

MCP is not a novelty connector — it is the standard way to let an agent reach
your repositories, databases, APIs, and test infrastructure. The engineering
work is the **governance between the agent and the tool**: which servers, what
access level, whose approval, what boundary, what happens on failure. An
ungoverned MCP setup is how an agent ends up with write access to production it
never needed.

## Background — the governance layer

```
  Agent ──▶ [ tool selection · permissions · approvals ]
            [ context / state boundaries ]              ──▶  Approved MCP servers
            [ failure handling · security ]                   (repo, DB, GitHub, docs, test infra)
```

You already have `.vscode/mcp.json` from Module 02 (filesystem, optionally
GitHub). This lab extends it and wraps a workflow and a governance record
around it.

---

## Step 1 — Extend `.vscode/mcp.json`

Open `.vscode/mcp.json`. Add a **read-only** Postgres server alongside the
existing ones. (Check your org allows it first.)

```jsonc
// .vscode/mcp.json
{
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${workspaceFolder}"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "${input:pg_url}"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${input:gh_token}" }
    }
  },
  "inputs": [
    {
      "id": "pg_url",
      "type": "promptString",
      "description": "READ-ONLY Postgres URL for the Task Board DB (use a read-only role)",
      "password": true
    },
    {
      "id": "gh_token",
      "type": "promptString",
      "description": "GitHub PAT, repo scope only",
      "password": true
    }
  ]
}
```

### Create a read-only DB role first

**macOS / Linux**

```bash
docker exec -i taskboard-db psql -U postgres -d taskboard <<'SQL'
CREATE ROLE mcp_ro LOGIN PASSWORD 'mcp_ro';
GRANT CONNECT ON DATABASE taskboard TO mcp_ro;
GRANT USAGE ON SCHEMA public TO mcp_ro;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_ro;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO mcp_ro;
SQL
```

**Windows (PowerShell)**

```powershell
@'
CREATE ROLE mcp_ro LOGIN PASSWORD 'mcp_ro';
GRANT CONNECT ON DATABASE taskboard TO mcp_ro;
GRANT USAGE ON SCHEMA public TO mcp_ro;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_ro;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO mcp_ro;
'@ | docker exec -i taskboard-db psql -U postgres -d taskboard
```

Use `postgresql://mcp_ro:mcp_ro@localhost:5432/taskboard` as the `pg_url` input.
**Prove it's read-only:** later, ask the agent to run an `INSERT` via the
Postgres tool — it must fail.

## Step 2 — Start the servers

Open `.vscode/mcp.json`, click **Start** on each server's code-lens. Check
**MCP: List Servers** (Command Palette) shows all three *Running*. In Agent mode,
click the **Tools** icon and enable the MCP tools for this chat.

## Step 3 — Run the four-step workflow

Do each step as its **own** chat turn, writing the output to a file before the
next step. Attach only what that step needs.

### Step 3.1 — Repository analysis

```
Using the filesystem MCP tools, analyse the soft-delete feature area:
- the layers and files involved (repository, service, router, frontend)
- every read path that must exclude soft-deleted tasks
- the tests that currently cover it
Write a concise summary to specs/002-task-soft-delete/repo-analysis.md.
Paths + one-line notes. Do not propose changes.
```

### Step 3.2 — Specification validation

```
Read specs/002-task-soft-delete/spec.md and repo-analysis.md. For each
acceptance criterion, state whether the implementation satisfies it, citing the
file. List any criterion not met and any behaviour in the code that isn't in the
spec. Write to specs/002-task-soft-delete/spec-validation.md.
```

### Step 3.3 — Test execution

```
Run the soft-delete tests: `pytest -q -m integration` and the contract tests in
backend-python. Using the read-only Postgres MCP tools, verify the DB state
matches what the tests assert (e.g. a soft-deleted row still exists with
deleted_at set). Summarise pass/fail + the DB check to
specs/002-task-soft-delete/test-run.md.
```

(Approve each terminal command and each Postgres tool call.)

### Step 3.4 — Result analysis / PR-evidence collection

```
From repo-analysis.md, spec-validation.md, and test-run.md, produce a PR-ready
evidence summary: what's implemented, which acceptance criteria are proven by
which test, test results, and any gap. Write to
specs/002-task-soft-delete/pr-evidence.md. Using the GitHub MCP tools, check
whether an open PR or issue for this feature exists and link it.
```

## Step 4 — Document the governance layer

**macOS / Linux**

```bash
cp ../module-07/templates/mcp-governance-checklist.md specs/002-task-soft-delete/mcp-governance.md
```

**Windows (PowerShell)**

```powershell
Copy-Item ..\module-07\templates\mcp-governance-checklist.md specs\002-task-soft-delete\mcp-governance.md
```

Fill every section. Run the two proofs:

- **Read-only proof:** ask the agent to `INSERT` a row via the Postgres tool →
  it must fail with a permissions error. Record it.
- **Secret proof:** `git diff .vscode/mcp.json` shows no URL or token — only
  `${input:...}` placeholders.

Also write `specs/002-task-soft-delete/mcp-workflow.md`: the four steps, what
each consumes and produces, and the tools each is allowed.

## Step 5 — Commit

```bash
git add .vscode/mcp.json specs/002-task-soft-delete/mcp-*.md specs/002-task-soft-delete/*-analysis.md specs/002-task-soft-delete/spec-validation.md specs/002-task-soft-delete/test-run.md specs/002-task-soft-delete/pr-evidence.md
git commit -m "Module 07 Lab 01: MCP workflow + governance for soft-delete"
```

## Verify

- [ ] `.vscode/mcp.json` has filesystem, read-only postgres, and github servers;
      no secrets committed.
- [ ] All three show *Running* in **MCP: List Servers**.
- [ ] The four workflow steps each produced their output file.
- [ ] An `INSERT` via the Postgres tool **fails** (read-only proven).
- [ ] `mcp-governance.md` is complete: every server has a reason, an access
      level, an approval model, and a failure behaviour.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Postgres MCP server won't start | Check the URL; try `npx -y @modelcontextprotocol/server-postgres "<url>"` in a terminal for the real error. Some builds renamed the package — check `modelcontextprotocol/servers`. |
| `INSERT` via the tool succeeds | You're connected as `postgres`, not `mcp_ro`. Fix the `pg_url` input. |
| Agent skips a step when a tool fails | That's the governance failure. Add "if a tool fails, halt and report — do not fabricate the step" to the prompt and to `mcp-governance.md`. |
| GitHub tool 401 | PAT lacks `repo` scope or expired. Re-run the input; **MCP: Reset Cached Tokens**. |
| Steps bleed context into each other | Start a fresh chat per step; attach only that step's inputs. |

## Recap & carry-forward

You have a governed, multi-step MCP workflow. **Lab 02** takes its most valuable
step — spec validation — and packages it as a versioned, owned skill any project
can adopt.
