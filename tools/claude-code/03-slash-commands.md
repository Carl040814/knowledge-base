# 03 · 斜杠命令大全

## 3.1 会话控制

| 命令 | 别名 | 功能 |
|------|------|------|
| `/help` | — | 显示所有可用命令 |
| `/clear` | `/reset`, `/new` | 清空会话，重新开始 |
| `/compact` | — | 压缩对话历史，释放上下文窗口 |
| `/context` | — | 可视化上下文使用情况（颜色网格） |
| `/cost` | `/usage`, `/stats` | token 消耗和估算费用 |
| `/resume` | `/continue` | 切换/恢复已保存会话 |
| `/sessions` | — | 列出所有已保存会话 |
| `/export` | — | 导出对话为 Markdown |
| `/rename` | — | 重命名当前会话 |
| `/name` | — | 为会话命名以便检索 |
| `/goal` | — | 设定完成条件，持续工作直到满足 |
| `/exit` | `/quit` | 退出 Claude Code |
| `/insights` | — | 使用分析报告 |

## 3.2 模式切换

| 命令 | 功能 |
|------|------|
| `/model <name>` | 切换模型 (`sonnet` / `opus` / `haiku`) |
| `/effort <level>` | 推理深度 (`low` / `medium` / `high` / `max`) |
| `/fast` | 切换快速模式 |
| `/plan` | 进入计划模式 |
| `/vim` | 切换 Vim 编辑模式 |
| `/voice` | 语音模式 |
| `/output-style` | 控制输出格式 |
| `/view` | 切换视图 (`default` / `verbose` / `focus`) |

## 3.3 配置与状态

| 命令 | 功能 |
|------|------|
| `/config` | 交互式配置编辑器 |
| `/settings` | 打开 settings.json 编辑 |
| `/status` | 会话状态（模型、上下文、模式） |
| `/version` | Claude Code 版本 |
| `/doctor` | 诊断配置问题 |
| `/statusline` | 自定义终端状态栏 |
| `/permissions` | 管理工具权限 |
| `/update-config` | 更新设置（hooks、权限、环境变量） |
| `/keybindings-help` | 自定义快捷键 |
| `/fewer-permission-prompts` | 添加常用命令到 allow 列表 |
| `/scroll-speed` | 调节 TUI 滚动速度 |
| `/terminal-setup` | 配置终端集成 (bash/zsh/WezTerm) |

## 3.4 代码与开发

| 命令 | 功能 |
|------|------|
| `/diff` | 交互式 diff 查看器 |
| `/review` | 代码审查（PR、文件、代码片段） |
| `/security-review` | 对当前分支变更安全审计 |
| `/simplify` | 检测过度工程，建议自动修复 |
| `/commit` | 暂存并 AI 生成提交信息 |
| `/pr` | 创建/管理 GitHub PR |
| `/pr_comments` | 查看 PR 评论 |
| `/autofix-pr` | 监控 PR 并自动修复 CI 失败 |
| `/test [pattern]` | 运行测试 |
| `/lint [file]` | 运行 linter/formatter |
| `/build` | 执行构建命令 |
| `/run` | 启动并驱动项目应用 |
| `/verify` | 验证代码变更是否正常 |
| `/todos` | 查看当前任务列表 |
| `/tasks` | 管理后台任务 |

## 3.5 Git 与工作区

| 命令 | 功能 |
|------|------|
| `/worktree [name]` | 创建/管理 git worktree |
| `/branch` (`/fork`) | 从当前节点创建对话分支 |
| `/rewind` (`/checkpoint`) | 回退代码变更和对话历史 |
| `/init` | 初始化项目 CLAUDE.md |
| `/add-dir <path>` | 添加目录到工作区上下文 |
| `/agents` | 创建和管理子代理 |
| `/agent <prompt>` | 为独立任务创建子代理 |
| `/batch` | 并行 worktree 重构（5-30 代理） |

## 3.6 MCP 与插件

| 命令 | 功能 |
|------|------|
| `/mcp` | 管理 MCP 服务器连接与状态 |
| `/plugin install <name>` | 安装插件 |
| `/plugin enable <name>` | 启用插件 |
| `/plugin disable <name>` | 禁用插件 |
| `/plugin list` | 列出已安装插件 |
| `/plugin marketplace` | 浏览插件市场 |
| `/skills` | 列出所有可用技能 |
| `/find-skills` | 搜索技能 |
| `/claude-api` | 构建/调试 Claude API 应用 |

## 3.7 自动化

| 命令 | 功能 |
|------|------|
| `/loop <interval> <cmd>` | 定时循环执行 (`/loop 5m /test`) |
| `/btw [question]` | 只读副代理（旁路问题） |
| `/copy [N]` | 交互式选择代码块复制到剪贴板 |
| `/teleport [session-id]` | 从 claude.ai 导入会话 |
| `/schedule` | 云端定时任务 |
| `/sandbox` | 隔离执行沙箱 |
| `/debug` | 调试模式（详细执行日志） |

## 3.8 账号与系统

| 命令 | 功能 |
|------|------|
| `/login` | 登录 Anthropic 账号 |
| `/logout` | 登出 |
| `/bug` | 提交 bug 报告 |
| `/feedback` | 发送反馈 |
| `/upgrade` | 升级 Claude Code |
| `/release-notes` | 版本更新说明 |
| `/docs [topic]` | 查看相关文档 |
| `/install-github-app` | 安装 GitHub App 以进行 PR 集成 |
| `/memory` | 在系统编辑器中打开记忆文件 |
| `/listen` | 监听模式 — 外部输入 |
