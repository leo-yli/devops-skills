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
2. **Demand Scheme ID** — For project-scoped pipelines. Use auto-detected value from Git branch if available.
3. **Branch** (optional) — Override default branch
4. **Environment** (optional) — `dev`, `test`, `staging`, `prod`
5. **Wait for completion** (optional) — Default: no

> **Important:** Only ask the user for values that could NOT be auto-detected. If auto-detection succeeds, use those values directly without prompting.

## Execution

Always use `--json` flag for machine-parseable output.

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
| Missing pipelineName (auto-detection failed) | Ask user for the pipeline name |
| Pipeline not found | Suggest checking the name or running `dops pipeline list` |
| Not authenticated | Suggest `dops auth login` |
| User cancels | Report cancellation gracefully |
