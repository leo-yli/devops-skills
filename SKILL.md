---
name: devops-skills
description: >
  DevOps platform skills collection for CI/CD pipeline management, Git
  repository maintenance, and deployment workflows. These skills depend on the
  devops-cli `dops` command. Use this skill when the user wants to perform any
  DevOps platform operation including: running pipelines, stopping builds,
  checking pipeline status, analyzing build history, cleaning Git branches,
  checking deployment readiness, or executing full deployment workflows. Also use
  when the user mentions CI/CD, pipeline, build, deploy, release, Git cleanup,
  or DevOps platform operations. This is the root skill that routes to
  sub-skills under the skills/ directory.
---

# DevOps Skills Collection

Claude Code skills for the DevOps platform. Each skill delegates to the
`devops-cli` (`dops`) command-line tool.

## Prerequisites

**These skills require `devops-cli` to be installed.**

Check installation:

```bash
dops --version
```

If not found, install it:

```bash
npm install -g devops-cli
# or: pnpm add -g devops-cli
```

If authentication fails, prompt the user to login:

```bash
dops auth login --host <url>
```

## Available Skills

每个子 skill 位于 `skills/<name>/SKILL.md`，是独立的 Claude Code skill：

| Skill | Purpose | Trigger Keywords |
|-------|---------|------------------|
| pipeline-runner | Trigger pipeline builds | run pipeline, start build, trigger CI |
| pipeline-stopper | Abort running pipelines | stop pipeline, cancel build, abort |
| pipeline-analyzer | Analyze build history | analyze pipeline, failure report, stats |
| pipeline-status | Check status and history | pipeline status, build history, monitor |
| git-cleanup | Clean up Git branches | delete branches, git cleanup, prune |
| deploy-checker | Pre-deployment validation | deploy check, readiness, can I deploy |
| deploy-workflow | Full deploy with checks | deploy, release, go live |

## Routing Guide

Based on user intent, read the appropriate sub-skill file and follow its instructions:

1. **Start a build** → Read `skills/pipeline-runner/SKILL.md`
2. **Stop a build** → Read `skills/pipeline-stopper/SKILL.md`
3. **View build history** → Read `skills/pipeline-status/SKILL.md`
4. **Analyze failures** → Read `skills/pipeline-analyzer/SKILL.md`
5. **Clean Git branches** → Read `skills/git-cleanup/SKILL.md`
6. **Check before deploying** → Read `skills/deploy-checker/SKILL.md`
7. **Deploy now** → Read `skills/deploy-workflow/SKILL.md`

Each sub-skill contains:
- Detailed parameter mapping
- Exact CLI command templates
- Output parsing instructions
- Error handling guidance

## Common Patterns

### Finding Pipeline Names

If the user doesn't know a pipeline name:

```bash
dops --json pipeline list
```

### Auto-Detecting Pipeline Name

For all pipeline-related skills, if `pipelineName` is not provided by the user, the skill will automatically:

1. Read `package.json` from the current directory
2. Extract the `name` field value
3. Use it as the pipeline name

This means users can simply say "帮我运行流水线" without specifying a name, and it will use the project name from `package.json`.

### Authentication

If commands fail with auth errors:

```bash
dops auth login --host <url>
```

### JSON Mode

All skills use `dops --json` for machine-parseable output. Parse the JSON
response and present results in a user-friendly format.

## Error Handling Reference

| Error | Meaning | Fix |
|-------|---------|-----|
| `登录已过期` | Session expired | Run `dops auth login` |
| `流水线不存在` | Wrong pipeline name | Verify name with `dops pipeline list` |
| `缺少必要参数` | Missing required param | Check skill SKILL.md for required params |
| `无权限执行此操作` | 403 Forbidden | Check user permissions in DevOpsPlatform |
