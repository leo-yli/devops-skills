# DevOps CLI Commands Reference

## Installation Check

Before using any skill, ensure `devops-cli` (dops) is installed:

```bash
# Check if dops is available
dops --version

# If not installed, install it first
npm install -g devops-cli
# or
pnpm add -g devops-cli
```

If installation fails, report to the user and stop.

## Authentication

```bash
# Login
dops auth login --host <host-url>

# Logout
dops auth logout
```

If commands fail with auth errors (401), prompt the user to run `dops auth login`.

## Pipeline Skills

### pipeline-runner
Trigger a pipeline build.

```bash
dops --json skill run pipeline-runner \
  --param pipelineName=<name> \
  [--param demandSchemeId=<id>] \
  [--param branch=<branch>] \
  [--param environment=<dev|test|staging|prod>] \
  [--param parameters='<json>'] \
  [--param wait=true] \
  [--param timeout=<minutes>]
```

**Required:** `pipelineName`
**Defaults:** `wait=false`, `timeout=10`

### pipeline-stopper
Abort a running pipeline.

```bash
dops --json skill run pipeline-stopper \
  --param pipelineName=<name> \
  [--param demandSchemeId=<id>] \
  [--param force=true] \
  [--param all=true]
```

**Required:** `pipelineName`

### pipeline-analyzer
Analyze pipeline execution history.

```bash
dops --json skill run pipeline-analyzer \
  --param pipelineName=<name> \
  --param demandSchemeId=<id> \
  [--param limit=<number>] \
  [--param focus=<all|failures|duration>]
```

**Required:** `pipelineName`, `demandSchemeId`
**Defaults:** `limit=10`, `focus=all`

### pipeline-status
Query pipeline status and history.

```bash
dops --json skill run pipeline-status \
  --param pipelineName=<name> \
  [--param demandSchemeId=<id>] \
  [--param buildId=<id>] \
  [--param history=<count>] \
  [--param watch=true] \
  [--param interval=<seconds>]
```

**Required:** `pipelineName`
**Defaults:** `history=5`, `watch=false`, `interval=5`

## Git Skills

### git-cleanup
Clean up merged or stale Git branches.

```bash
dops --json skill run git-cleanup \
  [--path <repo-path>] \
  [--dry-run <true|false>] \
  [--older-than <days>] \
  [--exclude <branch1,branch2>] \
  [--remote]
```

**Defaults:** `--path .`, `--dry-run true`, `--older-than 30`, `--exclude main,master,develop`

## Deploy Skills

### deploy-checker
Run pre-deployment checks.

```bash
dops --json skill run deploy-checker \
  --param demandSchemeId=<id> \
  [--param pipelineName=<name>] \
  [--param environment=<dev|staging|prod>]
```

**Required:** `demandSchemeId`
**Defaults:** `environment=staging`

### deploy-workflow
Full deployment workflow (check -> trigger -> wait).

```bash
dops --json skill run deploy-workflow \
  --param demandSchemeId=<id> \
  [--param pipelineName=<name>] \
  [--param environment=<dev|staging|prod>] \
  [--param wait=true|false]
```

**Required:** `demandSchemeId`
**Defaults:** `environment=staging`, `wait=true`

## Parameter Mapping

| User Says | CLI Flag |
|-----------|----------|
| pipeline / pipeline name | `--param pipelineName=<name>` |
| demand scheme / project | `--param demandSchemeId=<id>` |
| branch / git branch | `--param branch=<branch>` |
| environment / env / target | `--param environment=<env>` |
| wait / block / sync | `--param wait=true` |
| timeout / max wait | `--param timeout=<minutes>` |
| parameters / vars / extra | `--param parameters='<json>'` |
| force | `--param force=true` |
| all | `--param all=true` |
| path / repo | `--param path=<path>` |
| dry run / preview | `--param dryRun=true` |
| older than / days | `--param olderThan=<days>` |
| exclude / skip | `--param exclude=<branch1,branch2>` |
| remote | `--param remote=true` |
| history / count / limit | `--param history=<count>` |
| watch / monitor | `--param watch=true` |
| interval / refresh | `--param interval=<seconds>` |

## Finding Pipeline Names

If the user doesn't know a pipeline name:

```bash
dops --json pipeline list
```

## Common Error Patterns

| Error | Meaning | Fix |
|-------|---------|-----|
| `登录已过期` | Session expired | Run `dops auth login` |
| `流水线不存在` | Wrong pipeline ID | Verify ID with `dops pipeline list` |
| `缺少必要参数` | Missing required param | Check SKILL.md for required params |
| `无权限执行此操作` | 403 Forbidden | Check user permissions in DevOpsPlatform |

## Output Handling

All commands use `--json` flag. Parse the JSON response:
- `success: true` -> Operation succeeded
- `success: false` -> Report `error` and any `suggestions`
- Always present results in a user-friendly format
