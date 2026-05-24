# 13 · Git 与会话管理

## 13.1 Git 集成

### 基本操作

| 命令 | 功能 |
|------|------|
| `/commit` | AI 生成提交信息并提交 |
| `/pr` | 创建/管理 PR |
| `/pr_comments` | 查看 PR 评论 |
| `/autofix-pr` | 监控 PR 并自动修复 CI 失败 |
| `/review` | 代码审查 |
| `/security-review` | 安全审计 |
| `/diff` | 查看变更 |
| `/rewind` | 回退变更 (`/checkpoint`) |
| `/branch` | 创建对话分支 (`/fork`) |

### 归属标记

```json
{
  "attribution": {
    "commit": "🤖 Generated with Claude Code",
    "pr": ""
  },
  "prUrlTemplate": "https://gitlab.internal/{{owner}}/{{repo}}/-/merge_requests/{{number}}"
}
```

- `attribution.commit: ""` — 完全隐藏提交归属
- `attribution.pr: ""` — 完全隐藏 PR 归属
- `prUrlTemplate` — 适配私有 GitLab/Bitbucket 等

### 撤销操作

- `Esc` + `Esc` — 撤销最后一次文件更改
- `/rewind` — 回退到之前的检查点（同时回退代码和对话）

## 13.2 Worktree 隔离

```bash
# CLI 启动隔离环境
claude -w feature-x

# 带 tmux
claude -w feature-x --tmux

# 会话内
/worktree feature-x
```

Hooks 事件: `WorktreeCreate`, `WorktreeRemove`

## 13.3 会话管理

### 基本操作

| 命令 | 功能 |
|------|------|
| `/resume` | 恢复之前会话 |
| `/sessions` | 列出所有保存的会话 |
| `/rename` | 重命名当前会话 |
| `/name` | 命名当前会话 |
| `/export` | 导出为 Markdown |
| `/teleport [session-id]` | 从 claude.ai 导入会话 |

### 会话清理

```json
{
  "cleanupPeriodDays": 14   // 14 天后自动清理非活跃会话
}
```

清理范围 (v2.1.117+):
- `~/.claude/projects/` — 会话文件
- `~/.claude/tasks/` — 后台任务
- `~/.claude/shell-snapshots/` — Shell 快照
- `~/.claude/backups/` — 备份

设置 `CLAUDE_CODE_SKIP_PROMPT_HISTORY` 环境变量可完全不保存会话。

### 费用控制

```bash
/cost                           # 查看当前费用和 token
claude --max-budget-usd 5 -p "..."  # 设置上限（非交互模式）
```

## 13.4 GitHub App 集成

```bash
/install-github-app    # 安装 GitHub App
```

安装后: 自动 PR 管理、CI 监控、评论交互。

## 13.5 PR 工作流示例

```bash
# 1. 开发完成后
/commit                    # AI 生成提交信息

# 2. 创建 PR
/pr                        # AI 生成 PR 标题和描述

# 3. 审查
/review                    # 或 /review-pr

# 4. CI 失败时
/autofix-pr               # 自动监控并修复
```
