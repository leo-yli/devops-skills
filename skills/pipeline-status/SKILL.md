---
name: pipeline-status
description: >
  Check DevOps platform pipeline status, build history, and real-time progress.
  Use this skill whenever the user wants to know the status of a pipeline,
  view build history, monitor a running build, check recent executions, or
  query pipeline details. Also use when the user mentions status, history,
  monitor, progress, logs, or build details in the context of pipelines.
---

# Pipeline Status

Query pipeline status and history.

## When to Use

- User wants to check pipeline status
- User wants build history
- User wants to monitor a running build
- User mentions status, history, monitor, progress, logs

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

If not authenticated, prompt to login:

```bash
dops auth login --host <host-url>
```

## Required Information

1. **Pipeline Name** (optional) — e.g. `acc-account`. If not provided, automatically detected from current directory name
2. **Demand Scheme ID** (optional) — For project-scoped queries
3. **Build ID** (optional) — Specific build to inspect
4. **History** (optional) — Number of records. Default: 5
5. **Watch** (optional) — Continuous monitoring. Default: false
6. **Interval** (optional) — Refresh interval in seconds. Default: 5

> **Auto-Resolution:** If the current Git branch matches `feature/<number>` (e.g. `feature/1081265`), the demand scheme ID is automatically inferred from the branch name. You only need to provide `--param demandSchemeId=<id>` when not on a feature branch or when the auto-resolution fails.

> **Pipeline Name Auto-Detection:** If `pipelineName` is not provided, the skill will automatically use the current directory name as the pipeline name.

## Execution

Always use `--json` flag. Use `--param key=value` format.

### Pipeline Name Resolution

If `pipelineName` is not provided by the user, automatically detect it:

1. Get the current directory name
2. Use it as the pipeline name

### Command Execution

```bash
dops --json skill run pipeline-status \
  --param pipelineName=<name> \
  [--param demandSchemeId=<id>] \
  [--param buildId=<id>] \
  [--param history=<count>] \
  [--param watch=true] \
  [--param interval=<seconds>]
```

### Parameter Mapping

| User Says | CLI Flag |
|-----------|----------|
| pipeline / pipeline name | `--param pipelineName=<name>` |
| demand scheme / project | `--param demandSchemeId=<id>` |
| build id / task | `--param buildId=<id>` |
| history / count / records | `--param history=<count>` |
| watch / monitor / follow | `--param watch=true` |
| interval / refresh | `--param interval=<seconds>` |

### Examples

**Check basic status:**
```bash
dops --json skill run pipeline-status --param pipelineName=acc-account
```

**View history with project:**
```bash
dops --json skill run pipeline-status \
  --param pipelineName=acc-account \
  --param demandSchemeId=456 \
  --param history=10
```

**Monitor in real-time:**
```bash
dops --json skill run pipeline-status \
  --param pipelineName=acc-account \
  --param demandSchemeId=456 \
  --param watch=true \
  --param interval=10
```

## Output Handling

Present as:
- Current status (idle/running/failed)
- Recent build history table
- Stage details for specific builds
- Real-time updates (if watching)

## Error Handling

| Scenario | Action |
|----------|--------|
| Missing pipelineName | Ask user for the pipeline name |
| Pipeline not found | Suggest checking the name or running `dops pipeline list` |
| Not authenticated | Suggest `dops auth login` |
| Build ID not found | Report "build not found", suggest checking history |