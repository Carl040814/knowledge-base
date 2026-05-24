# 01 · 入门指南

## 一、什么是 Claude Code

Claude Code 是 Anthropic 推出的官方命令行 AI 编程工具，将 Claude 大模型直接集成到终端工作流中。

**核心能力:**
- 读写文件 — 创建、编辑、重构代码
- 执行命令 — 运行 shell、测试、构建
- 搜索代码库 — grep、glob 模式、语义搜索
- Git 操作 — 提交、PR、代码审查
- 自主决策 — 理解任务、拆解步骤、执行并验证
- 并行处理 — 子代理并行处理独立任务

**五种产品形态，共享同一引擎:**

| 界面 | 特点 |
|------|------|
| 终端 CLI | 功能最全、CI/CD、SSH、管道 |
| VS Code 扩展 | 内联 diff、多标签页、@-mentions |
| JetBrains 插件 | IDE 内集成 |
| 桌面应用 | 独立窗口、计划模式、定时任务 |
| Web (claude.ai/code) | 远程仓库、无需克隆 |

## 二、安装

```bash
# 全局安装
npm install -g @anthropic-ai/claude-code

# 更新到最新
claude update

# 安装指定版本
claude install stable       # 稳定版
claude install latest        # 最新版
claude install 2.1.0         # 指定版本
```

## 三、启动会话

```bash
# 交互式 REPL
claude

# 带初始 prompt
claude "修复 src/auth.ts 的类型错误"

# 非交互模式 — 单次输出后退出
claude -p "解释这个项目"

# 继续最近的会话
claude -c

# 恢复指定会话
claude --resume <session-id>

# 管道使用
cat error.log | claude -p "分析这些错误"

# 从 stdin 生成文件
claude -p "生成 Dockerfile" > Dockerfile
```

## 四、CLI 启动参数

### 会话控制

| 参数 | 简写 | 说明 |
|------|------|------|
| `--print` | `-p` | 非交互模式，单次输出后退出 |
| `--continue` | `-c` | 继续最近的会话 |
| `--resume [id]` | `-r` | 恢复到指定会话 |
| `--max-turns <N>` | — | 限制交互轮数（脚本模式） |
| `--max-budget-usd <$>` | — | 费用上限（仅非交互模式） |

### 模型与能力

| 参数 | 说明 |
|------|------|
| `--model <name>` | 指定模型 (`sonnet` / `opus` / `haiku`) |
| `--effort <level>` | 推理深度: `low` / `medium` / `high` / `max` |
| `--agent <name>` | 覆盖默认 agent |

### 上下文与控制

| 参数 | 说明 |
|------|------|
| `--add-dir <path>` | 启动时添加目录到上下文 |
| `--system-prompt <text>` | 覆盖系统提示词 |
| `--append-system-prompt <text>` | 追加到系统提示词 |
| `--allowedTools <tools>` | 限制可用工具 |
| `--disallowedTools <tools>` | 禁用特定工具 |
| `--settings <path>` | 加载额外配置文件 |

### 隔离与集成

| 参数 | 简写 | 说明 |
|------|------|------|
| `--worktree [name]` | `-w` | 创建 git worktree 隔离环境 |
| `--tmux` | — | 配合 `-w` 创建 tmux 会话 |
| `--ide` | — | 自动连接 IDE |
| `--bare` | — | 最小模式（跳过 hooks/LSP/插件） |
| `--plugin-dir <path>` | — | 加载本地开发中的插件 |

### 输出与调试

| 参数 | 说明 |
|------|------|
| `--output-format <fmt>` | `text` / `json` / `stream-json` |
| `--input-format <fmt>` | `text` / `stream-json` |
| `--verbose` | 详细日志 |
| `--debug [filter]` | 调试模式 |
| `--no-session-persistence` | 不保存会话 |

### 安全相关

| 参数 | 说明 |
|------|------|
| `--dangerously-skip-permissions` | 跳过所有权限检查 ⚠️ |
| `--disable-slash-commands` | 禁用所有技能/斜杠命令 |
| `--mcp-config <path>` | 加载 MCP 配置（可多次） |
| `--mcp-debug` | MCP 调试日志 |

## 五、环境诊断

```bash
claude doctor           # 诊断安装问题
claude --version        # 查看版本
claude --verbose        # 详细日志
```
