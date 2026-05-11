---
name: deploy-workflow
description: >
  Execute a complete deployment workflow on the DevOps platform: check readiness,
  trigger pipelines, and wait for completion. Use this skill whenever the user
  wants to deploy code, release to an environment, perform a full deployment,
  or run the complete CI/CD workflow. Also use when the user mentions deploy,
  release, push to production, go live, or full deployment workflow.
---

# Deploy Workflow

Complete deployment workflow: check → trigger → wait.

## When to Use

- User wants to deploy to an environment
- User wants a full end-to-end deployment
- User wants to release code with validation
- User wants automated deploy with checks
- User mentions deploy, release, go live, push to production

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
# or pnpm add -g devops-cli
```

If not authenticated, prompt to login:

```bash
dops auth login --host <host-url>
```

## Auto-Detection (MUST RUN FIRST)

Before asking the user for any information, you MUST attempt automatic detection. Only ask the user if auto-detection fails.

### Step 1: Auto-Detect Pipeline Name

If the user did not provide a pipeline name, automatically detect it:

**Read from package.json (preferred):**
```bash
# If package.json exists
pipelineName=$(cat package.json | grep '"name"' | head -1 | cut -d'"' -f4)
```

**Fallback to directory name:**

**On Windows (PowerShell):**
```powershell
$pipelineName = Split-Path -Leaf (Get-Location)
```

**On macOS/Linux (Bash):**
```bash
pipelineName=$(basename "$PWD")
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

1. **Demand Scheme ID** — The project to deploy. Use auto-detected value from Git branch if available.
2. **Pipeline Name** — Specific pipeline name (e.g. `acc-account`). Use auto-detected value if available.
3. **Environment** (optional) — Target environment. Default: `staging`
4. **Wait** (optional) — Wait for completion. Default: `true`

> **Important:** Only ask the user for values that could NOT be auto-detected. If auto-detection succeeds, use those values directly without prompting.

## Execution

Always use `--json` flag. Use `--param key=value` format.

```bash
dops --json skill run deploy-workflow \
  [--param demandSchemeId=<id>] \
  [--param pipelineName=<name>] \
  [--param environment=<dev|staging|prod>] \
  [--param wait=true|false]
```

### Parameter Mapping

| User Says | CLI Flag |
|-----------|----------|
| demand scheme / project | `--param demandSchemeId=<id>` |
| pipeline / pipeline name | `--param pipelineName=<name>` |
| environment / env / target | `--param environment=<env>` |
| wait / sync / block | `--param wait=true` |
| no-wait / async / fire-and-forget | `--param wait=false` |

### Examples

**Deploy to staging:**
```bash
dops --json skill run deploy-workflow --param demandSchemeId=123
```

**Deploy to production:**
```bash
dops --json skill run deploy-workflow \
  --param demandSchemeId=123 \
  --param environment=prod
```

**Deploy specific pipeline without waiting:**
```bash
dops --json skill run deploy-workflow \
  --param demandSchemeId=123 \
  --param pipelineName=acc-account \
  --param wait=false
```

## Workflow Steps

The skill executes these steps internally:
1. **Check** — Validate demand scheme and code merge status
2. **Trigger** — Start pipeline builds
3. **Wait** — Optionally monitor until completion

## Output Handling

The JSON response contains:
- `data.environment` — Target environment
- `data.triggered` — List of triggered pipelines with task IDs
- `data.results` — Step-by-step results

Present as:
```
## Deployment Workflow Results

### Triggered Pipelines
<pipeline-name>: task=<task-id>

### Step Results
1. Check: <status>
2. Trigger: <status>
3. Wait: <status>

### Summary
<message>
```

## Production Deployments

For `--environment prod`:
- Extra checks are performed (code must be merged)
- Warn user about production impact
- Report success/failure clearly

## Error Handling

| Scenario | Action |
|----------|--------|
| Missing demandSchemeId (auto-detection failed) | Ask user for it or suggest switching to a `feature/<number>` branch |
| Demand scheme not found | Report "project not found", verify ID |
| Check step fails | Stop workflow, report check failures, do not deploy |
| Trigger step fails | Report "deployment failed to start" with error details |
| Wait timeout | Report timeout, provide task ID for manual follow-up |
| Not authenticated | Suggest `dops auth login` |
