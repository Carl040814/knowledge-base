# 08 · MCP 协议与服务器

## 8.1 概述

MCP (Model Context Protocol) 是开放标准，让 Claude Code 连接外部工具、数据库和 API。MCP 服务器暴露"工具"供 Claude 调用 — 读取数据库、调用 API、管理文件、浏览器自动化等。

## 8.2 传输方式

| 传输 | 说明 | 状态 |
|------|------|------|
| **stdio** | Claude 作为子进程启动服务器 (stdin/stdout) | 本地首选 |
| **HTTP** | 可流式 HTTP 传输 | ✅ 推荐（远程） |
| **SSE** | Server-Sent Events | ⚠️ 已弃用，建议迁移 HTTP |

## 8.3 CLI 命令

```bash
# stdio 服务器
claude mcp add my-server -- node /path/to/server.js

# HTTP 远程服务器
claude mcp add --transport http my-server https://example.com/mcp

# 带环境变量
claude mcp add my-server -e API_KEY=sk-abc123 -- node server.js

# 带认证头标
claude mcp add --transport http my-api https://mcp.example.com/ \
  --header "Authorization: Bearer $MY_API_KEY"

# 指定作用域
claude mcp add --scope project --transport http stripe https://mcp.stripe.com

# 从 JSON 添加
claude mcp add-json my-server '{"type":"stdio","command":"node","args":["server.js"]}'

# 从 Claude Desktop 导入
claude mcp add-from-claude-desktop

# 管理
claude mcp list              # 列出
claude mcp get <name>        # 查看详情
claude mcp remove <name>     # 删除

# Claude Code 作为 MCP 服务器
claude mcp serve
```

## 8.4 配置作用域

| 作用域 | 存储位置 | 可见性 | 命令 |
|--------|----------|--------|------|
| **local** (默认) | `~/.claude.json` (按项目路径) | 你 + 此项目 | `--scope local` |
| **project** | `.mcp.json` (项目根) | 团队 | `--scope project` |
| **user** | `~/.claude.json` (全局) | 你 + 所有项目 | `--scope user` |

优先级: local > project > user

## 8.5 .mcp.json 格式

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "my-db": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@bytebase/dbhub", "--dsn", "postgresql://localhost/mydb"]
    },
    "private-api": {
      "type": "http",
      "url": "https://mcp.example.com/",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    },
    "unused-server": {
      "command": "npx",
      "args": ["-y", "some-server"],
      "disabled": true
    }
  }
}
```

**环境变量展开:** `${VAR}` 和 `${VAR:-default}` 语法均支持。

**动态头标:** `headersHelper` 字段支持 Kerberos、短期 token 等场景。

## 8.6 Tool Search (工具搜索)

自动延迟工具定义加载，将上下文消耗从 ~72K tokens 降至 ~8.7K tokens (减少 85%)。

配置 `ENABLE_TOOL_SEARCH`:

| 值 | 行为 |
|----|------|
| `true` | 始终延迟（强制启用） |
| `auto` | 工具不超过上下文 10% 时即时加载 |
| `auto:5` | 自定义阈值 (5%) |
| `false` | 始终即时加载（旧行为） |

需要 Sonnet 4+ 或 Opus 4+。

## 8.7 会话内管理

`/mcp` 命令显示:
- 各服务器连接状态
- OAuth 认证状态
- 可用工具列表
- 重连/断开选项

## 8.8 故障排除

| 问题 | 解决方案 |
|------|----------|
| "Connection closed" | PATH 不匹配 — 使用绝对路径或加载 nvm |
| Windows npx 失败 | 用 `cmd /c` 包裹: `claude mcp add srv -- cmd /c npx -y pkg` |
| 服务器超时 | 从 SSE 切换到 HTTP 传输 |
| "Command not found" | `export PATH="$(npm config get prefix)/bin:$PATH"` |
| JSON 语法错误 | 检查尾逗号，用 linter 验证 |
| 无工具出现 | 重启会话；某些服务器需要 `--enable-write` |
| dotenv v17+ 损坏 | `DOTENV_CONFIG_QUIET=true` |
