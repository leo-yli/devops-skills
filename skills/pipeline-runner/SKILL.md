---
name: pipeline-runner
description: >
  Trigger and run DevOps platform pipelines. Use this skill whenever the user
  wants to start a build, run a CI/CD pipeline, trigger deployment, or execute
  any pipeline on the DevOps platform. Also use when the user mentions build,
  run pipeline, trigger build, or start deployment even if they don't explicitly
  say "pipeline". This skill handles parameter passing, branch selection,
  environment targeting, and optional wait-for-completion.
---

# Pipeline Runner

Trigger pipeline builds on the DevOps platform.

## When to Use

- User wants to start a new build
- User wants to run/deploy a specific pipeline
- User wants to trigger CI/CD with custom parameters
- User wants to build a specific branch or environment

## Prerequisites

**Before running, check `dops` is installed:**

On Windows, use PowerShell:
```powershell
dops --version
```

On macOS/Linux, use Bash:
```bash
dops --version
```

If not found, install first:

```bash
npm install -g devops-cli
# or: pnpm add -g devops-cli
```

If the user is not authenticated (commands return 401), prompt them to login:

```bash
dops auth login --host <host-url>
```

## Required Information

Before running, collect from the user:
1. **Pipeline Name** (required) — The pipeline name, e.g. `acc-account`
2. **Demand Scheme ID** (optional) — For project-scoped pipelines
3. **Branch** (optional) — Override default branch
4. **Environment** (optional) — `dev`, `test`, `staging`, `prod`
5. **Wait for completion** (optional) — Default: no

## Execution

Always use `--json` flag for machine-parseable output.

```bash
dops --json skill run pipeline-runner --param pipelineName=<name> [options]
```

> **Note:** Use `--param key=value` format. The `--key value` shorthand has a known parsing issue in this version of dops.

### Parameter Mapping

| User Says | CLI Flag |
|-----------|----------|
| pipeline / pipeline name | `--param pipelineName=<name>` |
| demand scheme / project | `--param demandSchemeId=<id>` |
| branch / git branch | `--param branch=<branch>` |
| environment / env / target | `--param environment=<env>` |
| wait / block / sync | `--param wait=true` |
| timeout / max wait | `--param timeout=<minutes>` |
| parameters / vars / extra | `--param parameters='<json>'` |

### Examples

**Basic run:**
```bash
dops --json skill run pipeline-runner --param pipelineName=acc-account
```

**Run with environment and wait:**
```bash
dops --json skill run pipeline-runner \
  --param pipelineName=acc-account \
  --param environment=staging \
  --param wait=true
```

**Run project pipeline with custom params:**
```bash
dops --json skill run pipeline-runner \
  --param pipelineName=acc-account \
  --param demandSchemeId=456 \
  --param branch=feature/api \
  --param parameters='{"version": "1.2.3"}'
```

## Output Handling

Parse the JSON output:
- `success: true` → Build triggered. Report `taskId` and pipeline name.
- `success: false` → Report `error` and any `suggestions`.
- If waiting, report final status (success/failure) and stats.

## Error Handling

| Scenario | Action |
|----------|--------|
| Missing pipelineName | Ask user for the pipeline name |
| Pipeline not found | Suggest checking the name or running `dops pipeline list` |
| Not authenticated | Suggest `dops auth login` |
| User cancels | Report cancellation gracefully |
