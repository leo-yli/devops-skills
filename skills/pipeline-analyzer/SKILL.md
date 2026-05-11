---
name: pipeline-analyzer
description: >
  Analyze DevOps platform pipeline execution history and identify failure
  patterns. Use this skill whenever the user wants to see build statistics,
  analyze pipeline failures, get a failure report, review build trends, or
  understand why builds are failing. Also use when the user mentions pipeline
  analysis, failure report, build stats, or performance history.
---

# Pipeline Analyzer

Analyze pipeline build history and failure patterns.

## When to Use

- User wants build statistics or reports
- User wants to understand failure patterns
- User wants a failure analysis
- User mentions pipeline analysis, stats, trends, or history review

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
3. **Limit** (optional) — Number of records to analyze. Default: 10
4. **Focus** (optional) — `all`, `failures`, or `duration`. Default: `all`

> **Important:** Only ask the user for values that could NOT be auto-detected. If auto-detection succeeds, use those values directly without prompting.

## Execution

Always use `--json` flag. Use `--param key=value` format.

### Command Execution

```bash
dops --json skill run pipeline-analyzer \
  --param pipelineName=<name> \
  [--param demandSchemeId=<id>] \
  [--param limit=<number>] \
  [--param focus=<all|failures|duration>]
```

### Parameter Mapping

| User Says | CLI Flag |
|-----------|----------|
| pipeline / pipeline name | `--param pipelineName=<name>` |
| demand scheme / project | `--param demandSchemeId=<id>` |
| limit / count / records | `--param limit=<number>` |
| focus / type / view | `--param focus=<type>` |

### Examples

**Basic analysis:**
```bash
dops --json skill run pipeline-analyzer \
  --param pipelineName=acc-account \
  --param demandSchemeId=456
```

**Focus on failures only:**
```bash
dops --json skill run pipeline-analyzer \
  --param pipelineName=acc-account \
  --param demandSchemeId=456 \
  --param focus=failures
```

**Analyze last 20 builds:**
```bash
dops --json skill run pipeline-analyzer \
  --param pipelineName=acc-account \
  --param demandSchemeId=456 \
  --param limit=20
```

## Output Handling

Parse JSON and present as:
- Summary statistics (success rate, avg duration)
- Failure breakdown (if any)
- Duration trends
- Actionable recommendations

## Error Handling

| Scenario | Action |
|----------|--------|
| Missing pipelineName (auto-detection failed) | Ask user for it |
| Auto-resolution fails | Ask user to provide `--param demandSchemeId=<id>` explicitly |
| Pipeline not found | Suggest checking ID or running `dops pipeline list` |
| Not authenticated | Suggest `dops auth login` |
| No build history | Report "no records found" gracefully |
