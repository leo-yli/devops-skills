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

## Auto-Detection (MUST RUN FIRST)

Before asking the user for any information, you MUST attempt automatic detection. Only ask the user if auto-detection fails.

### Step 1: Auto-Detect Pipeline Name

If the user did not provide a pipeline name, automatically detect it:

**On Windows (PowerShell):**
```powershell
$pipelineName = Split-Path -Leaf (Get-Location)
```

**On macOS/Linux (Bash):**
```bash
pipelineName=$(basename "$PWD")
```

Alternatively, read `package.json` from the current directory and use the `name` field:
```bash
# If package.json exists
pipelineName=$(cat package.json | grep '"name"' | head -1 | cut -d'"' -f4)
```

### Step 2: Auto-Detect Demand Scheme ID

If the user did not provide a demand scheme ID, automatically detect it:

1. Extract the short number from current Git branch (`feature/<number>`)
2. Resolve the real demand scheme ID using `dops schemes demand resolve`

**On Windows (PowerShell):**
```powershell
$branch = git rev-parse --abbrev-ref HEAD
# Step 1: Extract short number from feature/12345 pattern
if ($branch -match "feature/(\d+)") {
    $shortId = $matches[1]
    # Step 2: Resolve real demand scheme ID
    $result = dops --json schemes demand resolve $shortId | ConvertFrom-Json
    $demandSchemeId = $result.id
}
```

**On macOS/Linux (Bash):**
```bash
branch=$(git rev-parse --abbrev-ref HEAD 2>/dev/null)
# Step 1: Extract short number from feature/12345 pattern
if [[ "$branch" =~ feature/([0-9]+) ]]; then
    shortId="${BASH_REMATCH[1]}"
    # Step 2: Resolve real demand scheme ID
    demandSchemeId=$(dops --json schemes demand resolve "$shortId" | jq -r '.id')
fi
```

> **Note:** The number in the branch name (e.g., `1753408` from `feature/1753408`) is a **short demand ID**, not the real demand scheme ID. Always run `dops schemes demand resolve <shortId>` to get the actual ID before passing it to pipeline commands.

## Required Information

After attempting auto-detection above:

1. **Pipeline Name** — The pipeline to abort, e.g. `acc-account`. Use auto-detected value if available.
2. **Demand Scheme ID** — For project-scoped pipelines. Use auto-detected value from Git branch if available.
3. **Force** (optional) — Skip confirmation prompt
4. **All** (optional) — Abort all running instances of this pipeline

> **Important:** Only ask the user for values that could NOT be auto-detected. If auto-detection succeeds, use those values directly without prompting.

## Execution

Always use `--json` flag. Use `--param key=value` format for parameters.

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
| Missing pipelineName (auto-detection failed) | Ask user for the pipeline name |
| Pipeline not running | Report "pipeline is not currently running" |
| Not authenticated | Suggest `dops auth login` |
| User lacks permission | Report "insufficient permissions to stop this pipeline" |
