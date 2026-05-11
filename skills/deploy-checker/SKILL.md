---
name: deploy-checker
description: >
  Run pre-deployment validation checks on the DevOps platform. Use this skill
  whenever the user wants to verify deployment readiness, check if they can
  deploy, validate pre-deployment conditions, or confirm everything is ready
  before releasing. Also use when the user mentions deploy check, readiness,
  can I deploy, or pre-deployment validation.
---

# Deploy Checker

Pre-deployment validation tool.

## When to Use

- User wants to check before deploying
- User wants deployment readiness verification
- User asks "can I deploy?"
- User mentions deploy check, readiness, validation

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

1. **Demand Scheme ID** — The project to check. Use auto-detected value from Git branch if available.
2. **Pipeline Name** — e.g. `acc-account`. Use auto-detected value if available.
3. **Environment** (optional) — `dev`, `staging`, `prod`. Default: `staging`

> **Important:** Only ask the user for values that could NOT be auto-detected. If auto-detection succeeds, use those values directly without prompting.

## Execution

Always use `--json` flag. Use `--param key=value` format.

```bash
dops --json skill run deploy-checker \
  [--param demandSchemeId=<id>] \
  [--param pipelineName=<name>] \
  [--param environment=<dev|staging|prod>]
```

### Parameter Mapping

| User Says | CLI Flag |
|-----------|----------|
| demand scheme / project | `--param demandSchemeId=<id>` |
| pipeline / pipeline name | `--param pipelineName=<name>` |
| environment / env / target | `--param environment=<env>` |

### Examples

**Check staging deployment:**
```bash
dops --json skill run deploy-checker --param demandSchemeId=123
```

**Check production deployment:**
```bash
dops --json skill run deploy-checker \
  --param demandSchemeId=123 \
  --param environment=prod
```

**Check specific pipeline:**
```bash
dops --json skill run deploy-checker \
  --param demandSchemeId=123 \
  --param pipelineName=acc-account
```

## Production Checks

For `--environment prod`:
- Extra checks are performed (code must be merged)
- Warn user about production impact
- Report success/failure clearly

## Output Handling

Present check results as:
- Pass/fail/warning for each check item
- Summary counts
- Clear go/no-go recommendation

## Error Handling

| Scenario | Action |
|----------|--------|
| Missing demandSchemeId (auto-detection failed) | Ask user for it or suggest switching to a `feature/<number>` branch |
| Demand scheme not found | Report "project not found", verify ID |
| Not authenticated | Suggest `dops auth login` |
| Checks fail | Report failures clearly, advise fixes before deploying |
