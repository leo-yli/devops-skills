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

## Required Information

1. **Demand Scheme ID** (required)
2. **Pipeline Name** (optional) — e.g. `acc-account`
3. **Environment** (optional) — `dev`, `staging`, `prod`. Default: `staging`

## Execution

Always use `--json` flag. Use `--param key=value` format.

```bash
dops --json skill run deploy-checker \
  --param demandSchemeId=<id> \
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
| Missing demand-scheme-id | Ask user for it |
| Demand scheme not found | Report "project not found", verify ID |
| Not authenticated | Suggest `dops auth login` |
| Checks fail | Report failures clearly, advise fixes before deploying |
