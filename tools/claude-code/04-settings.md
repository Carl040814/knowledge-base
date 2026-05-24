# 04 · 配置文件 settings.json

## 4.1 文件层级 (优先级从高到低)

| 优先级 | 路径 | Git | 用途 |
|--------|------|-----|------|
| 1 (最高) | 托管策略 `managed-settings.json` | ❌ | 组织强制，不可覆盖 |
| 2 | `managed-settings.d/*.json` | ❌ | 模块化策略追加 |
| 3 | `~/.claude/settings.json` | ❌ | 用户全局偏好 |
| 4 | `~/.claude/settings.local.json` | ❌ | 机器特定覆盖 |
| 5 | `.claude/settings.json` | ✅ | 项目级，团队共享 |
| 6 (最低) | `.claude/settings.local.json` | ❌ | 个人本地覆盖 (gitignore) |

IDE 自动补全:
```json
{ "$schema": "https://json.schemastore.org/claude-code-settings.json" }
```

## 4.2 常规 / 运行时

| 键 | 类型 | 默认值 | 说明 |
|----|------|--------|------|
| `model` | `string` | `"default"` | 默认模型，支持别名 `sonnet`/`opus`/`haiku` |
| `agent` | `string` | — | 将主线程作为命名子代理运行 |
| `language` | `string` | `"english"` | 响应语言 |
| `cleanupPeriodDays` | `number` | `30` | 非活跃会话清理天数 (最小 1) |
| `autoUpdatesChannel` | `string` | `"latest"` | 更新频道: `"stable"` / `"latest"` |
| `minimumVersion` | `string` | — | 阻止自动更新降级到指定版本以下 |
| `alwaysThinkingEnabled` | `boolean` | `false` | 默认启用扩展思考 |
| `effortLevel` | `string` | — | 持久化推理深度: `"low"` / `"medium"` / `"high"` / `"xhigh"` |
| `fastModePerSessionOptIn` | `boolean` | `false` | 快速模式需每会话确认 |
| `defaultShell` | `string` | `"bash"` | `!` 命令使用的 shell: `"bash"` / `"powershell"` |
| `tui` | `string` | `"default"` | 渲染模式: `"fullscreen"` / `"default"` |
| `viewMode` | `string` | — | 默认视图: `"default"` / `"verbose"` / `"focus"` |
| `editorMode` | `string` | `"normal"` | 键位: `"normal"` / `"vim"` |
| `autoScrollEnabled` | `boolean` | `true` | 全屏模式跟随输出滚动 |
| `awaySummaryEnabled` | `boolean` | `true` | 回来后显示会话摘要 |
| `feedbackSurveyRate` | `number` | — | 调查概率 (0–1) |
| `companyAnnouncements` | `string[]` | — | 启动时随机展示公告 |
| `showTurnDuration` | `boolean` | `true` | 显示耗时 |
| `respectGitignore` | `boolean` | `true` | `@` 文件选择器遵循 .gitignore |
| `terminalProgressBarEnabled` | `boolean` | `true` | 终端进度条 |
| `spinnerVerbs` | `object` | — | 自定义加载动画文字 |
| `spinnerTipsEnabled` | `boolean` | `true` | 工作时显示提示 |
| `skipWebFetchPreflight` | `boolean` | `false` | 跳过 WebFetch 预检 |
| `includeGitInstructions` | `boolean` | `true` | 系统提示中包含 git 指令 |
| `showThinkingSummaries` | `boolean` | `false` | 显示扩展思考摘要 |
| `showTokensCounter` | `boolean` | — | 显示 token 计数器 |
| `disableDeepLinkRegistration` | `string` | — | `"disable"` 阻止 claude-cli:// 注册 |

## 4.3 记忆与 CLAUDE.md

| 键 | 类型 | 默认值 | 说明 |
|----|------|--------|------|
| `claudeMdExcludes` | `string[]` | — | glob 匹配要跳过的 CLAUDE.md |
| `claudeMd` | `string` | — | (托管) 注入组织级 CLAUDE.md |
| `autoMemoryEnabled` | `boolean` | `true` | 启用/禁用自动记忆 |
| `autoMemoryDirectory` | `string` | — | 自定义记忆存储目录 |

## 4.4 归属标记 (Attribution)

| 键 | 类型 | 说明 |
|----|------|------|
| `attribution.commit` | `string` | 追加到 git commit 文本，`""` 隐藏 |
| `attribution.pr` | `string` | 追加到 PR 描述，`""` 隐藏 |
| `prUrlTemplate` | `string` | PR 徽章 URL 模板 (私有 GitLab/Bitbucket) |

## 4.5 MCP 服务器

| 键 | 类型 | 说明 |
|----|------|------|
| `allowedMcpServers` | `object[]` | (托管) 服务器白名单 |
| `deniedMcpServers` | `object[]` | (托管) 服务器黑名单 |
| `enableAllProjectMcpServers` | `boolean` | 自动批准 .mcp.json 中所有服务器 |
| `enabledMcpjsonServers` | `string[]` | 从 .mcp.json 批准特定服务器 |
| `disabledMcpjsonServers` | `string[]` | 从 .mcp.json 拒绝特定服务器 |

## 4.6 Hooks

| 键 | 类型 | 说明 |
|----|------|------|
| `disableAllHooks` | `boolean` | 禁用所有 hooks |
| `allowedHttpHookUrls` | `string[]` | HTTP hook URL 白名单 (支持 `*` 通配符) |
| `allowManagedHooksOnly` | `boolean` | (托管) 仅托管/SDK hooks |

## 4.7 其他

| 键 | 类型 | 说明 |
|----|------|------|
| `env` | `object` | 应用于每个会话的环境变量 `{"FOO": "bar"}` |
| `plansDirectory` | `string` | 计划文件存储路径 |
| `voice` | `object` | 语音听写: `{enabled, mode ("hold"/"tap"), autoSubmit}` |
| `channelsEnabled` | `boolean` | (托管) 允许频道 |
| `blockedMarketplaces` | `object[]` | (托管) 市场来源黑名单 |

## 4.8 配置示例

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-sonnet-4-6",
  "cleanupPeriodDays": 20,
  "language": "english",
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true,
  "autoMemoryEnabled": true,
  "claudeMdExcludes": [
    "**/vendor/**/CLAUDE.md",
    "**/node_modules/**/CLAUDE.md"
  ],
  "attribution": {
    "commit": "🤖 Generated with Claude Code",
    "pr": ""
  },
  "env": {
    "MY_CUSTOM_VAR": "value"
  },
  "permissions": {
    "allow": [
      "Bash(npm test *)",
      "Bash(git *)",
      "Read"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(sudo *)"
    ]
  }
}
```
