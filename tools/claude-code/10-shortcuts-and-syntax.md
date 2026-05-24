# 10 · 快捷键与特殊语法

## 10.1 通用快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+C` | 中断当前 AI 生成 |
| `Ctrl+R` | 搜索命令历史 |
| `Ctrl+O` | 切换详细输出（显示思考过程） |
| `Ctrl+S` | 暂存当前草稿 |
| `Shift+Tab` | 循环切换权限模式 |
| `Esc` | 中断当前操作（不丢失上下文） |
| `Esc` + `Esc` | 撤销最后文件更改 |
| `↑` / `↓` | 浏览输入历史 |
| `\` + `Enter` | 多行输入 |
| `Option+Enter` (macOS) | 多行输入 |
| `Shift+Enter` | 多行输入（跨平台） |
| `Tab` | 自动补全文件、命令、MCP 资源 |
| `?` | 显示快捷键覆盖层 |

## 10.2 Vim 模式 (`/vim`)

| 快捷键 | 功能 |
|--------|------|
| `Esc` | 进入普通模式 |
| `i` | 进入插入模式 |
| `h/j/k/l` | 左右上下移动 |
| `Ctrl+J/K` | 上下移动（替代方案） |
| `w/b` | 按词移动 |
| `0/$` | 行首/尾 |
| `dd` | 删除当前行 |
| `u` | 撤销 |
| `Ctrl+R` | 重做 |

## 10.3 自定义快捷键

编辑 `~/.claude/keybindings.json`:

```json
{
  "bindings": [
    {
      "keys": ["ctrl+a"],
      "command": "cursorHome"
    },
    {
      "keys": ["ctrl+e"],
      "command": "cursorEnd"
    }
  ]
}
```

## 10.4 特殊语法

| 语法 | 用途 | 示例 |
|------|------|------|
| `! <command>` | 直接执行 shell（绕过 AI） | `! npm test` |
| `? <question>` | 搜索查询（避免工具调用） | `? MCP 是什么` |
| `@<filepath>` | 在上下文引用文件 | `@src/main.py` |
| `@<dirpath>` | 在上下文引用目录 | `@src/components` |

## 10.5 环境变量

| 变量 | 说明 |
|------|------|
| `ANTHROPIC_MODEL` | 设置默认模型 |
| `ANTHROPIC_API_KEY` | API 密钥 |
| `ANTHROPIC_BASE_URL` | 自定义 API 端点 |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | 启用遥测 |
| `CLAUDE_CODE_SKIP_PROMPT_HISTORY` | 不保存会话 |
| `ENABLE_TOOL_SEARCH` | MCP 工具搜索: `true`/`auto`/`auto:N`/`false` |
| `DOTENV_CONFIG_QUIET` | 静默 dotenv 警告 |
| `NO_COLOR` | 禁用彩色输出 |
| `CLAUDE_CODE_API_KEY_HELPER` | 自定义认证脚本路径 |
| `OTEL_METRICS_EXPORTER` | OpenTelemetry 导出器 |
