# 05 · Hooks 生命周期钩子

## 5.1 概述

Hooks 是事件驱动的自动化脚本，在 Claude Code 生命周期的特定时刻执行。可用于验证、拦截、修改、扩展或记录代理行为。

## 5.2 配置结构

```json
{
  "hooks": {
    "<事件名>": [
      {
        "matcher": "<匹配模式>",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/script.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

## 5.3 所有 Hook 事件 (27 个)

| 事件 | 触发时机 | 可阻止 | 典型用途 |
|------|----------|--------|----------|
| `SessionStart` | 会话开始或恢复 | ❌ | 加载 git 状态、TODO |
| `UserPromptSubmit` | 用户提交 prompt 后，Claude 处理前 | ✅ | 危险 prompt 过滤 |
| `UserPromptExpansion` | 命令展开为 prompt 时 | ❌ | 记录 |
| `PreToolUse` | 工具调用之前 | ✅ | 安全层、输入验证、输入修改 |
| `PostToolUse` | 工具调用成功后 | ❌ | 日志、lint、格式化 |
| `PostToolUseFailure` | 工具调用失败后 | ❌ | 错误追踪 |
| `PostToolBatch` | 一批并行工具调用全部完成后 | ❌ | 批量后处理 |
| `PermissionRequest` | 权限弹窗即将显示时 | ✅ | 自动批准/拒绝 |
| `PermissionDenied` | 自动模式拒绝工具调用时 | ❌ | 安全日志 |
| `Notification` | 发送通知时 | ❌ | 自定义通知渠道 |
| `Stop` | Claude 完成响应时 | ✅ | 后响应验证 |
| `StopFailure` | 轮次因 API 错误结束时 | ❌ | 错误处理 |
| `SubagentStart` | 子代理任务开始时 | ❌ | 记录、资源分配 |
| `SubagentStop` | 子代理任务完成时 | ❌ | 结果收集 |
| `PreCompact` | 上下文压缩前 | ❌ | 保存会话状态 |
| `PostCompact` | 上下文压缩后 | ❌ | 上下文重建 |
| `SessionEnd` | 会话终止时 | ❌ | 清理、版本标记 |
| `Setup` | `/setup` 命令时 | ❌ | 环境初始化 |
| `TeammateIdle` | 队友代理空闲时 | ❌ | 任务分配 |
| `TaskCreated` | 后台任务创建时 | ❌ | 任务追踪 |
| `TaskCompleted` | 后台任务完成时 | ❌ | 通知 |
| `ConfigChange` | 配置文件变更时 | ❌ | 配置热加载 |
| `WorktreeCreate` | 创建 git worktree 时 | ❌ | 环境初始化 |
| `WorktreeRemove` | 删除 git worktree 时 | ❌ | 清理 |
| `InstructionsLoaded` | CLAUDE.md 或规则加载时 | ❌ | 指令审计 |
| `CwdChanged` | 工作目录变更时 | ❌ | 上下文切换 |
| `FileChanged` | 监听到文件变更时 | ❌ | 自动操作 |
| `Elicitation` / `ElicitationResult` | MCP 请求/响应用户输入 | ❌ | 自定义 UI |

## 5.4 处理器类型

| 类型 | 说明 | 默认超时 | 可用事件 |
|------|------|----------|----------|
| `command` | Shell 命令，stdin 接收 JSON | 600s | 全部 |
| `http` | POST JSON 到 URL | 600s | 全部 |
| `mcp_tool` | 调用 MCP 工具 | 600s | 全部 |
| `prompt` | 单轮 LLM 评估 | 30s | PreToolUse, PostToolUse, PostToolUseFailure, PermissionRequest, Stop, SubagentStop, TaskCompleted, UserPromptSubmit |
| `agent` | 带 Read/Grep/Glob 的子代理 | 60s | 同上 |

## 5.5 Matcher 匹配语法

| 语法 | 行为 | 示例 |
|------|------|------|
| `"*"`, `""`, 或省略 | 匹配所有 | 每条都触发 |
| `"Bash"` | 精确匹配 | 仅 Bash 工具 |
| `"Write\|Edit"` | 管道分隔多匹配 | Write 或 Edit |
| `"mcp__.*"` | JavaScript 正则 | 所有 MCP 工具 |
| `"^Notebook"` | JS 正则 (startsWith) | Notebook 开头 |

## 5.6 退出码与输出协议

| 退出码 | 行为 |
|--------|------|
| `0` | 允许 — 正常继续 |
| `2` | 阻止 — 停止操作，stderr 显示给 Claude |
| 其他非零 | 非阻塞错误 — 记录日志后继续 |

**高级 JSON 输出** (stdout):

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "不允许强制推送到 main 分支"
  }
}
```

`permissionDecision` 取值: `"allow"` (绕过权限) / `"deny"` (阻止) / `"ask"` (询问用户) / `"defer"` (延迟)

**修改工具输入** (仅 PreToolUse):

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "updatedInput": {
      "command": "npm test -- --coverage"
    }
  }
}
```

## 5.7 stdin JSON 输入示例

```json
{
  "session_id": "abc123",
  "cwd": "/path/to/project",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": { "command": "git push origin main" },
  "transcript_path": "/tmp/transcript.jsonl",
  "last_assistant_message": "..."
}
```

## 5.8 环境变量

| 变量 | 可用事件 |
|------|----------|
| `CLAUDE_TOOL_NAME` | Pre/PostToolUse |
| `CLAUDE_TOOL_INPUT` | Pre/PostToolUse |
| `CLAUDE_TOOL_OUTPUT` | PostToolUse |
| `CLAUDE_EVENT` | 全部 |
| `CLAUDE_PROJECT_DIR` | 全部 |
| `CLAUDE_FILE_PATH` | Edit/Write |
| `CLAUDE_LINE_START` | Edit/Write |
| `CLAUDE_LINE_END` | Edit/Write |
| `CLAUDE_NOTIFICATION_TEXT` | Notification |
| `CLAUDE_SUBAGENT_NAME` | SubagentStart/Stop |
| `CLAUDE_SUBAGENT_RESULT` | SubagentStop |
| `CLAUDE_ENV_FILE` | SessionStart |

## 5.9 禁用 Hooks

```json
// settings.local.json — 全部禁用
{ "disableAllHooks": true }

// hooks-config.json — 按类型禁用
{
  "disablePreToolUseHook": true,
  "disablePostToolUseHook": true,
  "disableLogging": true
}
```

## 5.10 实战示例

**安全守卫 (PreToolUse):**
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "node .claude/hooks/safety-guard.js"
      }]
    }]
  }
}
```

**自动格式化 (PostToolUse):**
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
      }]
    }]
  }
}
```

**桌面通知 (Stop):**
```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "osascript -e 'display notification \"Claude 已完成\"'"
      }]
    }]
  }
}
```
