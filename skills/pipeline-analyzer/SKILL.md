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

## Required Information

1. **Pipeline Name** (required) — e.g. `acc-account`
2. **Demand Scheme ID** (optional) — For project-scoped queries
3. **Limit** (optional) — Number of records to analyze. Default: 10
4. **Focus** (optional) — `all`, `failures`, or `duration`. Default: `all`

> **Auto-Resolution:** If the current Git branch matches `feature/<number>` (e.g. `feature/1081265`), the demand scheme ID is automatically inferred from the branch name. You only need to provide `--param demandSchemeId=<id>` when not on a feature branch or when the auto-resolution fails.

## Execution

Always use `--json` flag. Use `--param key=value` format.

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
| Missing pipelineName | Ask user for it |
| Auto-resolution fails | Ask user to provide `--param demandSchemeId=<id>` explicitly |
| Pipeline not found | Suggest checking ID or running `dops pipeline list` |
| Not authenticated | Suggest `dops auth login` |
| No build history | Report "no records found" gracefully |
