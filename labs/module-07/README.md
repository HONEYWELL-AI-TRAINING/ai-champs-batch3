# Module 07 — MCP-Enabled Agentic Workflows & Reusable Engineering Skills (Hands-On Lab)

## What this lab is

The features are built (Modules 04–05) and validated (Module 06). Module 07 is
about the **agents that did that work**: connecting them to approved tools via
**MCP** under a real governance layer, composing a multi-step engineering
workflow, and packaging the reusable parts as **versioned, owned skills**. Same
Task Board codebase, same `specs/` and `.github/` layer. No new app.

> **RAG is intentionally excluded** from this programme. The time goes to MCP,
> governance, and reusable skills.

| Lab | Focus | You produce |
|-----|-------|-------------|
| [01](lab-01-mcp-workflow-and-governance.md) | MCP servers + a governed multi-step workflow | expanded `.vscode/mcp.json`, `mcp-workflow.md`, `mcp-governance.md` |
| [02](lab-02-package-a-reusable-skill.md) | Package one step as a versioned, owned skill | `.github/skills/spec-validator/` (SKILL.md, RUBRIC.md, VERSION, OWNERS, CHANGELOG) |
| [03](lab-03-compose-and-measure.md) | Compose workflow + skill, log cost, finalise governance | `workflow-run-log.md`, `skill-reuse-model.md` |

Work through them in order. Total time: about **2 hours**.

## The workflow you will build

The representative MCP-enabled engineering workflow from the course outline:

```
repository analysis → specification validation → test execution → result analysis / PR-evidence collection
```

Run against the **soft-delete** feature (`specs/002-task-soft-delete/`).

## Prerequisites

- **Modules 01–06 completed.** Task Board runs locally; `specs/001-…` and
  `specs/002-…` are on your branch with specs, plans, and test strategies; the
  `.github/skills/port-endpoint/` skill and `.vscode/mcp.json` from Module 02 are
  present; Module 06's Testcontainers tests run.
- [Module 01 prerequisites](../setup/prerequisites.md) + a working backend and DB.
- **Docker** running (for the Postgres MCP server and test execution).
- **GitHub Copilot** in VS Code (Agent mode); `node`/`npx` on PATH; the repo on
  GitHub.
- Awareness of your org's MCP policy — **check which MCP servers you're allowed
  to run** before Lab 01.

## Platform conventions

Shell blocks are given for **macOS / Linux (bash/zsh)** and **Windows
(PowerShell)**. WSL2 users follow the macOS / Linux side. MCP config and Copilot
Chat are identical on every OS.

## Set-up — do this once before Lab 01

### 1. Open the Task Board as the workspace root

**macOS / Linux**

```bash
cd labs/module-01
code .
```

**Windows (PowerShell)**

```powershell
cd labs\module-01
code .
```

### 2. Branch from your Module 06 work

```bash
git checkout module-06-testing        # or wherever Module 06 landed
git checkout -b module-07-mcp
```

### 3. Confirm the starting state

**macOS / Linux**

```bash
docker info > /dev/null && echo "docker ok"
ls .vscode/mcp.json .github/skills/port-endpoint/SKILL.md specs/002-task-soft-delete/spec.md
```

**Windows (PowerShell)**

```powershell
docker info | Out-Null ; "docker ok"
Get-ChildItem .vscode\mcp.json, .github\skills\port-endpoint\SKILL.md, specs\002-task-soft-delete\spec.md
```

Present → [start Lab 01](lab-01-mcp-workflow-and-governance.md).

## How each lab is structured

Objective → Why it matters for the enterprise → Background → Steps (both
platforms) → Files/templates → Verify → Troubleshooting → Recap & carry-forward.

## After this module

- The repo carries a governed MCP workflow and a versioned, owned skill.
- **Module 08** puts the pull request through an automated quality gate and an
  LLM-as-Judge, with human escalation.
