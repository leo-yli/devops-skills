# devops-skills

Claude Code 技能集合，用于 CI/CD 流水线管理、Git 仓库维护和部署工作流。

> **定位**：本项目是 **Claude Code 插件市场格式的技能集**，依赖 `devops-cli` (`dops` 命令) 执行实际操作。


---

## 目录

- [安装](#安装)
- [使用](#使用)
- [可用技能列表](#可用技能列表)
- [技能参数](#技能参数)
- [项目结构](#项目结构)
- [常见问题](#常见问题)

---

## 安装

### 先决条件

- [Node.js](https://nodejs.org/) >= 18
- [Git](https://git-scm.com/)
- Claude Code 已安装并配置

### 第一步：安装 devops-cli

这些 skills 依赖 `devops-cli` (`dops` 命令) 执行实际操作。

```bash
# 检查是否已安装
dops --version

# 未安装则执行以下任一命令
npm install -g devops-cli
# 或: pnpm add -g devops-cli
# 或: yarn global add devops-cli
```

验证安装：

```bash
dops --version
# 应输出版本号，如: 2.5.1
```

登录 DevOps 平台：

```bash
dops auth login --host <your-devops-host>
```

> 替换 `<your-devops-host>` 为你的 DevOps 平台地址，如 `https://ci.example.com`。

### 第二步：安装 devops-skills

选择以下任一方式安装。

#### 方式一：项目本地安装（推荐，最简单）

将本项目的 `skills/` 目录放到你的项目根目录，Claude Code 会自动识别。

**macOS / Linux：**

```bash
# 符号链接（推荐，更新只需 pull 一次）
git clone https://github.com/leo-yli/devops-skills.git ~/tools/devops-skills
cd your-project
ln -s ~/tools/devops-skills/skills ./skills
```

**Windows（需要管理员权限）：**

```powershell
# 以管理员身份运行 PowerShell，然后执行：
git clone https://github.com/leo-yli/devops-skills.git C:\tools\devops-skills
cd your-project
New-Item -ItemType SymbolicLink -Path skills -Target C:\tools\devops-skills\skills
```

> Windows 创建符号链接需要管理员权限。如果无法使用符号链接，也可以直接复制 `skills` 目录，但更新时需要手动同步。

**项目结构：**

```
my-project/
├── src/
├── package.json
└── skills/                      ← 指向 devops-skills/skills
    ├── pipeline-runner/
    │   └── SKILL.md
    ├── deploy-workflow/
    │   └── SKILL.md
    └── ...
```

#### 方式二：Plugin 安装（全局可用）

所有项目共享，无需在每个项目重复配置。

1. 打开 Claude Code 全局设置文件 `~/.claude/settings.json`
2. 添加 marketplace 源并启用 plugin：
   ```json
   {
     "extraKnownMarketplaces": {
       "devops-skills": {
         "source": {
           "source": "github",
           "repo": "leo-yli/devops-skills"
         }
       }
     },
     "enabledPlugins": {
       "devops-skills@devops-skills": true
     }
   }
   ```
3. 保存配置并完全重启 Claude Code
4. 运行 `/skills` 确认 devops-skills 已加载

---

## 使用

安装完成后，直接在 Claude Code 中说：

| 你想做的事 | 说的话 |
|---|---|
| 运行流水线 | `"帮我运行流水线 123"` |
| 查看状态 | `"查看流水线 456 的状态"` |
| 停止构建 | `"停止构建"` |
| 清理分支 | `"清理 Git 分支"` |
| 部署 | `"部署项目 789 到 staging"` |

Claude Code 会自动匹配 skill、拼接 `dops` 命令、解析结果并汇报。

### 执行流程示例

```
用户: 帮我部署项目 123 到 staging 环境

Claude Code:
  1. 匹配到 deploy-workflow skill
  2. 检查 dops 是否安装
  3. 检查是否已登录
  4. 执行: dops --json skill run deploy-workflow --param demandSchemeId=123 --param environment=staging
  5. 解析 JSON 输出，用中文汇报结果
```

---

## 可用技能列表

| 技能名 | 功能 | 触发场景 |
|---|---|---|
| **pipeline-runner** | 触发流水线执行 | "运行流水线"、"开始构建" |
| **pipeline-stopper** | 终止正在运行的流水线 | "停止构建"、"取消部署" |
| **pipeline-analyzer** | 分析流水线执行历史 | "为什么构建失败了"、"查看统计" |
| **pipeline-status** | 查询流水线状态和历史 | "查看状态"、"构建历史" |
| **git-cleanup** | 清理已合并或过期的分支 | "清理分支"、"删除旧分支" |
| **deploy-checker** | 部署前检查 | "能部署吗"、"检查发布条件" |
| **deploy-workflow** | 完整部署工作流 | "部署"、"发布"、"上线" |

---

## 技能参数

### pipeline-runner

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `pipelineName` | string | 是 | - | 流水线名称，如 `acc-account` |
| `demandSchemeId` | number | 否 | - | 需求项目 ID |
| `branch` | string | 否 | - | 要构建的分支 |
| `environment` | string | 否 | - | 部署环境：dev/test/staging/prod |
| `parameters` | string | 否 | - | 额外的构建参数（JSON格式） |
| `wait` | boolean | 否 | false | 是否等待构建完成 |
| `timeout` | number | 否 | 10 | 等待超时时间（分钟） |

### pipeline-stopper

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `pipelineName` | string | 是 | - | 流水线名称 |
| `demandSchemeId` | number | 否 | - | 需求项目 ID |
| `force` | boolean | 否 | false | 强制终止，不提示确认 |
| `all` | boolean | 否 | false | 终止所有运行实例 |

### pipeline-analyzer

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `pipelineName` | string | 是 | - | 流水线名称 |
| `demandSchemeId` | number | 是 | - | 需求项目 ID |
| `limit` | number | 否 | 10 | 分析最近多少次执行记录 |
| `focus` | string | 否 | all | 关注重点：all/failures/duration |

### pipeline-status

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `pipelineName` | string | 是 | - | 流水线名称 |
| `demandSchemeId` | number | 否 | - | 需求项目 ID |
| `buildId` | string | 否 | - | 特定构建 ID |
| `history` | number | 否 | 5 | 显示历史记录数量 |
| `watch` | boolean | 否 | false | 持续监控状态变化 |
| `interval` | number | 否 | 5 | 监控刷新间隔（秒） |

### git-cleanup

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `path` | string | 否 | . | Git 仓库路径 |
| `dryRun` | boolean | 否 | true | 试运行模式（默认只查看不删除） |
| `olderThan` | number | 否 | 30 | 清理多少天未更新的分支 |
| `exclude` | string | 否 | main,master,develop | 排除的分支名称（逗号分隔） |
| `remote` | boolean | 否 | false | 是否清理远程分支 |

### deploy-checker

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `demandSchemeId` | number | 是 | - | 需求项目 ID |
| `pipelineName` | string | 否 | - | 流水线名称 |
| `environment` | string | 否 | staging | 目标环境：dev/staging/prod |

### deploy-workflow

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `demandSchemeId` | number | 是 | - | 需求项目 ID |
| `pipelineName` | string | 否 | - | 流水线名称 |
| `environment` | string | 否 | staging | 目标环境：dev/staging/prod |
| `wait` | boolean | 否 | true | 是否等待构建完成 |

---

## 项目结构

**插件市场格式**（`skills/<name>/SKILL.md`）：

```
devops-skills/
├── README.md                     # 本文件
├── SKILL.md                      # 根 skill 入口（路由到子 skill）
├── skills/                       # 插件市场标准 skills 目录
│   ├── pipeline-runner/
│   │   └── SKILL.md              # 触发流水线 skill
│   ├── pipeline-stopper/
│   │   └── SKILL.md              # 终止流水线 skill
│   ├── pipeline-analyzer/
│   │   └── SKILL.md              # 分析历史 skill
│   ├── pipeline-status/
│   │   └── SKILL.md              # 查询状态 skill
│   ├── git-cleanup/
│   │   └── SKILL.md              # 清理分支 skill
│   ├── deploy-checker/
│   │   └── SKILL.md              # 部署检查 skill
│   └── deploy-workflow/
│       └── SKILL.md              # 完整部署 skill
└── references/
    └── devops-cli-commands.md    # dops 命令参考文档
```

---

## 常见问题

### Claude Code 没有触发 skill

**插件市场安装**：
- 确认已在插件市场安装 devops-skills
- 重启 Claude Code 会话

**项目本地安装**：
- 确认 `skills/` 目录在项目根目录下（注意是 `skills/` 不是 `devops-skills/`）
- 确认 `skills/<skill-name>/SKILL.md` 文件存在
- 尝试使用更明确的触发词，如 "运行流水线"、"部署项目"

### dops 命令未找到

```bash
npm install -g devops-cli
```

安装后重启 Claude Code 会话。

### 登录已过期

```bash
dops auth login --host https://ci.jlpay.com
```

### 流水线不存在

- 检查 `pipeline-id` 是否正确
- 使用 `dops --json pipeline list` 查看可用流水线

### 无权限执行此操作

- 确认登录用户有执行该操作的权限
- 联系平台管理员申请权限

---

## License

MIT
