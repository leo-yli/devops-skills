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

1. **Pipeline Name** — e.g. `acc-account`. Use auto-detected value if available.
2. **Demand Scheme ID** — For project-scoped queries. Use auto-detected value from Git branch if available.
3. **Build ID** (optional) — Specific build to inspect
4. **History** (optional) — Number of records. Default: 5
5. **Watch** (optional) — Continuous monitoring. Default: false
6. **Interval** (optional) — Refresh interval in seconds. Default: 5

> **Important:** Only ask the user for values that could NOT be auto-detected. If auto-detection succeeds, use those values directly without prompting.

## Execution

Always use `--json` flag. Use `--param key=value` format.

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
| Missing pipelineName (auto-detection failed) | Ask user for the pipeline name |
| Pipeline not found | Suggest checking the name or running `dops pipeline list` |
| Not authenticated | Suggest `dops auth login` |
| Build ID not found | Report "build not found", suggest checking history |