---
name: pipeline-stopper
description: >
  Abort and stop running DevOps platform pipelines. Use this skill whenever
  the user wants to cancel a build, stop a running pipeline, terminate an
  active deployment, or abort any in-progress CI/CD job. Also use when the
  user mentions cancel, stop, abort, kill, or terminate in the context of
  pipelines or builds.
---

# Pipeline Stopper

Abort running pipelines on the DevOps platform.

## When to Use

- User wants to cancel a running build
- User wants to stop a pipeline that's stuck or failing
- User wants to abort a deployment
- User mentions cancel/stop/abort/kill/terminate for a pipeline

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

If the user is not authenticated, prompt them to login:

```bash
dops auth login --host <host-url>
```

## Required Information

1. **Pipeline Name** (optional) — The pipeline to abort, e.g. `acc-account`. If not provided, automatically detected from current directory name
2. **Demand Scheme ID** (optional) — For project-scoped pipelines
3. **Force** (optional) — Skip confirmation prompt
4. **All** (optional) — Abort all running instances of this pipeline

> **Auto-Resolution:** If the current Git branch matches `feature/<number>` (e.g. `feature/1081265`), the demand scheme ID is automatically inferred from the branch name. You only need to provide `--param demandSchemeId=<id>` when not on a feature branch or when the auto-resolution fails.

> **Pipeline Name Auto-Detection:** If `pipelineName` is not provided, the skill will automatically use the current directory name as the pipeline name.

## Execution

Always use `--json` flag. Use `--param key=value` format for parameters.

### Pipeline Name Resolution

If `pipelineName` is not provided by the user, automatically detect it:

1. Get the current directory name
2. Use it as the pipeline name

### Command Execution

```bash
dops --json skill run pipeline-stopper --param pipelineName=<name> [options]
```

### Parameter Mapping

| User Says | CLI Flag |
|-----------|----------|
| pipeline / pipeline name | `--param pipelineName=<name>` |
| demand scheme / project | `--param demandSchemeId=<id>` |
| force / yes / no confirm | `--param force=true` |
| all / everything | `--param all=true` |

### Examples

**Abort specific pipeline:**
```bash
dops --json skill run pipeline-stopper --param pipelineName=acc-account
```

**Force abort without confirmation:**
```bash
dops --json skill run pipeline-stopper --param pipelineName=acc-account --param force=true
```

**Abort project pipeline:**
```bash
dops --json skill run pipeline-stopper \
  --param pipelineName=acc-account \
  --param demandSchemeId=456
```

## Output Handling

- `success: true` → Pipeline aborted. Confirm to user.
- `success: false` → Report `error` and suggestions.
- Warn user if the pipeline was already completed.

## Error Handling

| Scenario | Action |
|----------|--------|
| Missing pipelineName | Ask user for the pipeline name |
| Pipeline not running | Report "pipeline is not currently running" |
| Not authenticated | Suggest `dops auth login` |
| User lacks permission | Report "insufficient permissions to stop this pipeline" |
