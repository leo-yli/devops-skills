---
name: git-cleanup
description: >
  Clean up merged or stale Git branches from repositories. Use this skill
  whenever the user wants to delete branches, prune old branches, remove
  merged branches, or perform Git repository maintenance. Also use when the
  user mentions branch cleanup, git prune, delete branches, or repository
  maintenance.
---

# Git Cleanup

Clean up merged or stale Git branches.

## When to Use

- User wants to delete merged branches
- User wants to prune old branches
- User wants Git repository maintenance
- User mentions branch cleanup, git prune, delete branches

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

If the user wants to clean remote branches (`--remote`), check authentication. If not authenticated, prompt to login:

```bash
dops auth login --host <host-url>
```

## Required Information

1. **Path** (optional) — Repository path. Default: current directory
2. **Dry Run** (optional) — Preview without deleting. Default: true
3. **Older Than** (optional) — Days threshold. Default: 30
4. **Exclude** (optional) — Branches to skip. Default: `main,master,develop`
5. **Remote** (optional) — Also clean remote branches. Default: false

## Execution

Always use `--json` flag. Use `--param key=value` format.

```bash
dops --json skill run git-cleanup \
  [--param path=<repo-path>] \
  [--param dryRun=<true|false>] \
  [--param olderThan=<days>] \
  [--param exclude=<branch1,branch2>] \
  [--param remote=true]
```

### Parameter Mapping

| User Says | CLI Flag |
|-----------|----------|
| path / repo / directory | `--param path=<path>` |
| dry run / preview / test | `--param dryRun=<true|false>` (default: true) |
| older than / days / age | `--param olderThan=<days>` |
| exclude / skip / keep | `--param exclude=<branches>` |
| remote / origin | `--param remote=true` |

### Examples

**Preview cleanup:**
```bash
dops --json skill run git-cleanup --param path=./my-project
```

**Execute cleanup (not dry run):**
```bash
dops --json skill run git-cleanup \
  --param path=./my-project \
  --param dryRun=false \
  --param olderThan=7
```

**Clean remote branches too:**
```bash
dops --json skill run git-cleanup \
  --param path=./my-project \
  --param dryRun=false \
  --param remote=true
```

## Important Notes

- **Default is dry-run mode** — no branches are deleted unless `--dry-run false` is set
- Always show the preview first and ask for confirmation before deleting
- Warn about irreversible deletions

## Output Handling

- Dry run: Show list of branches that would be deleted
- Real run: Report deleted/failed counts

## Error Handling

| Scenario | Action |
|----------|--------|
| Path is not a git repo | Report "not a Git repository" |
| No branches to clean | Report "no stale branches found" |
| Remote auth fails | Suggest `dops auth login` |
| User cancels | Report cancellation gracefully |
