# 07 · 技能与插件

## 7.1 Skills 技能系统

### 概述
Skills 是 Claude 根据上下文自动识别并调用的领域专长指令，放在 `SKILL.md` 文件中。相比于手动调用命令，Skills 更像"内置专家知识"。

### 技能位置（发现顺序）
1. 插件内: `plugins/<name>/skills/<skill>/SKILL.md`
2. 项目级: `.claude/skills/<skill>/SKILL.md`
3. 用户级: `~/.claude/skills/<skill>/SKILL.md`

### SKILL.md 格式

```markdown
---
name: typescript-conventions
description: 强制执行 TypeScript 编码规范。在编写或审查 TypeScript 代码时使用。
---

当编写 TypeScript 代码时:
- 导出函数必须显式声明返回类型
- 优先使用 interface 而非 type 定义对象结构
- 对字面类型使用 const assertions
- 启用 strict 模式，不使用 `as any`
- 使用 `unknown` 而非 `any` 处理未知类型
```

### 内置技能

| 技能 | 功能 |
|------|------|
| `claude-api` | 构建/调试 Claude API 应用 |
| `review` | 审查 PR |
| `security-review` | 安全审计 |
| `simplify` | 代码优化简化 |
| `init` | 初始化项目 CLAUDE.md |
| `fewer-permission-prompts` | 减少权限提示 |
| `update-config` | 配置文件管理 |
| `keybindings-help` | 快捷键自定义 |
| `loop` | 定时任务 |

### 命令
- `/skills` — 列出所有可用技能
- `/find-skills` — 搜索技能

## 7.2 Plugins 插件系统

### 概述
插件是可共享的扩展包，将多种组件捆绑为一个可安装单元。生态已发展到 9000+ 插件，43+ 社区市场。

### 插件可包含

| 组件 | 目录 | 用途 |
|------|------|------|
| 斜杠命令 | `commands/` | `/command` 快捷方式 |
| 技能 | `skills/` | 自动调用领域专长 |
| 子代理 | `agents/` | 委派任务 |
| Hooks | `hooks/hooks.json` | 事件自动化 |
| MCP 服务器 | `.mcp.json` | 外部工具连接 |
| LSP 服务器 | `.lsp.json` | 语言服务器 |

### 插件结构

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # 必需: 元数据 (name, version, author)
├── commands/
│   └── review.md            # 斜杠命令: /my-plugin:review
├── skills/
│   └── code-review/
│       └── SKILL.md          # 自动调用技能
├── hooks/
│   └── hooks.json            # 事件处理器
└── .mcp.json                 # MCP 服务器配置
```

### 安装与管理

```bash
# 添加市场
/plugin marketplace add user-or-org/repo-name

# 浏览并安装
/plugin menu
/plugin install context7@claude-plugins-official

# 管理
/plugin list                    # 列表
/plugin enable <name>           # 启用
/plugin disable <name>          # 禁用

# 测试本地插件
claude --plugin-dir ./my-plugin
```

## 7.3 自定义斜杠命令

### 创建
在以下位置放置 Markdown 文件:

- **项目级**: `.claude/commands/<name>.md` → `/name`
- **用户级**: `~/.claude/commands/<name>.md`
- **命名空间**: `.claude/commands/learn/quiz.md` → `/learn:quiz`

### 命令文件格式

```markdown
---
description: 部署项目到生产环境
allowed-tools: Bash, Read
model: sonnet
---

你是一个部署助手。按以下步骤:
1. 确认测试通过: `npm test`
2. 构建: `npm run build`
3. 检查未提交更改: `git status`
4. 如干净，打标签: `git tag v$ARGUMENTS`
5. 推送: `git push origin main --tags`

若任何步骤失败，停止并报告原因。
```

### 参数替换

| 变量 | 含义 |
|------|------|
| `$ARGUMENTS` / `$0` | 命令后所有文本 |
| `$1`, `$2`, ... | 位置参数 |

### Frontmatter 字段

| 字段 | 说明 |
|------|------|
| `description` | 命令描述 (`/help` 中显示) |
| `allowed-tools` | 限制可用工具 |
| `model` | 指定模型 |
| `agent` | 指定代理类型 |

## 7.4 组件对比: 什么时候用什么

| 机制 | 最适合 | 分发方式 |
|------|--------|----------|
| **Plugins** | 打包多种组件共享 | 市场安装 |
| **Skills** | 教 Claude 领域知识 (自动调用) | SKILL.md 文件 |
| **MCP** | 连接外部 API/数据库 | .mcp.json |
| **Hooks** | 事件响应自动化 | hooks.json |
| **斜杠命令** | 重复任务的快捷 prompt | commands/ 目录 |
