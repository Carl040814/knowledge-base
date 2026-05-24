# Claude Code 使用指南

## 概述

Claude Code 是 Anthropic 官方推出的 CLI 工具，将 Claude AI 直接集成到终端中。支持代码编写、调试、重构、Git 操作等软件工程任务。

## 安装

```bash
npm install -g @anthropic-ai/claude-code
cd your-project
claude
```

## 核心概念

### 权限系统

Claude Code 有三种权限模式，控制工具调用的审批：

| 模式 | 行为 |
|------|------|
| `default` | 大部分读写操作需审批 |
| `acceptEdits` | 自动批准文件编辑，Bash 仍需审批 |
| `bypassPermissions` | 完全跳过审批（仅限受信任环境） |

### 工作目录

- 命令在**当前工作目录**的上下文中执行
- 支持 git 仓库感知
- 可通过 `EnterWorktree` 创建隔离的工作树

## 常用命令

### 会话控制

| 命令 | 功能 |
|------|------|
| `/help` | 显示帮助信息 |
| `/clear` | 清空会话上下文 |
| `/compact` | 压缩长对话，释放上下文窗口 |
| `/context` | 查看当前上下文使用情况 |
| `/cost` | 查看当前会话的 token 费用 |
| `/doctor` | 诊断环境问题 |
| `/mcp` | 管理 MCP 服务器 |

### 模式切换

| 命令 | 功能 |
|------|------|
| `/fast` | 切换到快速模式（Opus，低延迟输出） |
| `/plan` | 进入计划模式（手动触发） |
| `/config` | 打开配置面板（模型、主题、权限） |

### 权限管理

| 命令 | 功能 |
|------|------|
| `/permissions` | 查看和管理权限设置 |
| `/login` | 登录 Anthropic 账号 |
| `/logout` | 登出 |

### 自动化

| 命令 | 功能 |
|------|------|
| `/loop` | 循环执行某个命令（如 `/loop 5m /test`） |
| `/cron` | 定时任务 |

### Git 集成

| 命令 | 功能 |
|------|------|
| `/pr-comments` | 查看 PR 评论 |
| `/review-pr` | 审查 PR |

### 其他

| 命令 | 功能 |
|------|------|
| `/bug` | 提交 bug 报告 |
| `/feedback` | 发送反馈 |
| `/init` | 初始化项目的 CLAUDE.md 配置文件 |
| `/export` | 导出对话 |
| `/resume` | 恢复之前的会话 |
| `/vim` | 切换 Vim 编辑模式 |
| `/statusline` | 配置状态栏 |

## 配置文件

### 设置层级 (优先级从高到低)

1. **用户级**: `~/.claude/settings.json`
2. **项目级**: `.claude/settings.json`（项目根目录）
3. **本地覆盖**: `.claude/settings.local.json`

### settings.json 结构

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(git status)",
      "Bash(git diff)"
    ],
    "deny": [
      "Bash(rm -rf *)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $FILE_PATH"
          }
        ]
      }
    ]
  },
  "model": "claude-sonnet-4-6",
  "theme": "dark"
}
```

### 常用设置项

| 设置 | 说明 |
|------|------|
| `model` | 默认模型 (opus/sonnet/haiku) |
| `theme` | 主题 (dark/light) |
| `permissions` | 权限白名单/黑名单 |
| `hooks` | 自动化钩子 |
| `env` | 环境变量 |
| `statusLine` | 状态栏类型 |
| `enableAllProjectMcpServers` | 自动加载 MCP servers |
| `showTokensCounter` | 显示 token 计数器 |

## Hooks 系统

在事件触发时自动执行命令。可用事件：

| 事件 | 触发时机 |
|------|----------|
| `PreToolUse` | 工具调用之前 |
| `PostToolUse` | 工具调用之后 |
| `Notification` | 收到通知时 |
| `Stop` | Claude 会话结束时 |
| `PreCompact` | 上下文压缩之前 |

### Hook 配置示例

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "echo '文件已修改'"
        }]
      }
    ],
    "Stop": [
      {
        "hooks": [{
          "type": "command",
          "command": "osascript -e 'display notification \"Claude 已完成\"'"
        }]
      }
    ]
  }
}
```

## CLAUDE.md

项目级指令文件，放在项目根目录，用于：

- 记录项目架构和约定
- 指定构建/测试命令
- 设定代码风格偏好
- 定义长期行为规则

```
# CLAUDE.md 示例

本项目使用 TypeScript + React。

## 命令
- `npm run dev` — 启动开发服务器
- `npm test` — 运行测试

## 规则
- 使用函数组件，不要用 class 组件
- 测试文件放在 __tests__ 目录
```

## 记忆系统

Claude Code 有持久化记忆功能，存储在 `~/.claude/projects/<project-path>/memory/`。

四种记忆类型：
- **user** — 用户角色、偏好、知识背景
- **feedback** — 用户反馈的行为指导
- **project** — 项目目标、决策、上下文
- **reference** — 外部资源指针（仪表盘、文档链接等）

记忆会在跨会话时保留，自动提升协作效率。

## IDE 集成

- **VS Code**: 安装 Claude Code 扩展
- **JetBrains**: 安装 Claude Code 插件
- **Web**: [claude.ai/code](https://claude.ai/code)

## 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+C` | 中断当前操作 |
| `Ctrl+D` | 退出会话 |
| `Ctrl+R` | 搜索历史命令 |
| `↑/↓` | 浏览命令历史 |
| `Ctrl+J/K` | 上下移动（Vim 模式） |
| `Esc` | 退出 Vim 插入模式 |

自定义快捷键：编辑 `~/.claude/keybindings.json`

## MCP (Model Context Protocol)

通过 MCP 服务器扩展 Claude Code 能力：

```bash
# 添加 MCP 服务器
claude mcp add <name> <command>

# 列出已配置的 MCP 服务器
claude mcp list
```

## 实用技巧

1. **用 `!` 前缀直接在终端执行命令**: `! npm run build`
2. **批量操作**: 一次性发送多个独立工具调用，并行执行
3. **项目级 CLAUDE.md**: 团队成员共享 Claude 的行为约定
4. **权限白名单**: 将常用安全命令加入 allow 列表减少审批
5. **Vim 模式**: `/vim` 开启 Vim 键位编辑
6. **Plan 模式**: 重大改动前先让 Claude 出方案，审查后再执行
7. **/compact**: 长对话后压缩上下文，保持性能
8. **恢复会话**: `/resume` 可以继续之前的对话

## 相关链接

- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
- [GitHub Issues](https://github.com/anthropics/claude-code/issues)
