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

## Required Information

1. **Demand Scheme ID** (optional) — The project to deploy
2. **Pipeline Name** (optional) — Specific pipeline name (e.g. `acc-account`), or auto-detect all
3. **Environment** (optional) — Target environment. Default: `staging`
4. **Wait** (optional) — Wait for completion. Default: `true`

> **Auto-Resolution:** If the current Git branch matches `feature/<number>` (e.g. `feature/1081265`), the demand scheme ID is automatically inferred from the branch name. You only need to provide `--param demandSchemeId=<id>` when not on a feature branch or when the auto-resolution fails.

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
| Missing demandSchemeId (and not on feature branch) | Ask user for it or suggest switching to a `feature/<number>` branch |
| Auto-resolution fails | Ask user to provide `--param demandSchemeId=<id>` explicitly |
| Demand scheme not found | Report "project not found", verify ID |
| Check step fails | Stop workflow, report check failures, do not deploy |
| Trigger step fails | Report "deployment failed to start" with error details |
| Wait timeout | Report timeout, provide task ID for manual follow-up |
| Not authenticated | Suggest `dops auth login` |
