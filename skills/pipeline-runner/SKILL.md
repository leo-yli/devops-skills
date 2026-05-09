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
1. **Pipeline Name** (optional) — The pipeline name, e.g. `acc-account`. If not provided, automatically detected from `package.json` name field in current directory
2. **Demand Scheme ID** (optional) — For project-scoped pipelines
3. **Branch** (optional) — Override default branch
4. **Environment** (optional) — `dev`, `test`, `staging`, `prod`
5. **Wait for completion** (optional) — Default: no

> **Auto-Resolution:** If the current Git branch matches `feature/<number>` (e.g. `feature/1081265`), the demand scheme ID is automatically inferred from the branch name. You only need to provide `--param demandSchemeId=<id>` when not on a feature branch or when the auto-resolution fails.

> **Pipeline Name Auto-Detection:** If `pipelineName` is not provided, the skill will automatically read the `name` field from `package.json` in the current directory and use it as the pipeline name. If `package.json` does not exist or does not contain a valid `name`, the user will be prompted to provide the pipeline name.

## Execution

Always use `--json` flag for machine-parseable output.

### Pipeline Name Resolution

If `pipelineName` is not provided by the user, automatically detect it:

1. Read `package.json` from the current directory
2. Extract the `name` field value
3. Use it as the pipeline name

**Implementation:**
```bash
# Check if pipelineName was provided by user
# If not, read from package.json
if [ -z "$pipelineName" ]; then
    if [ -f "package.json" ]; then
        pipelineName=$(cat package.json | grep -o '"name"[[:space:]]*:[[:space:]]*"[^"]*"' | head -1 | sed 's/.*"\([^"]*\)".*/\1/')
    fi
fi
```

### Command Execution

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
