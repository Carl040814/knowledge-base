# 14 · 最佳实践

## 14.1 项目初始化

1. 新项目运行 `/init` 创建 CLAUDE.md
2. 填写项目概述、命令、架构、规范
3. 在 `.claude/settings.json` 配置团队共享权限
4. 添加 `.claude/settings.local.json` 到 `.gitignore`
5. 提交 `.claude/` 目录（除 `.local` 文件）

## 14.2 日常开发工作流

| 场景 | 推荐模式 | 说明 |
|------|----------|------|
| 简单修复 | `default` | 直接描述需求 |
| 大型重构 | `plan` | `/plan` 先出方案，审查后执行 |
| 信任迭代 | `acceptEdits` | 编辑自动批准，命令仍需确认 |
| 高频日常 | `auto` | 仅高风险操作审批 |
| CI/CD | `-p` 非交互 | `claude -p "任务"` |

## 14.3 减少权限提示

```bash
/fewer-permission-prompts   # 自动分析并建议 allow 规则
```

手动策略:
```json
{
  "permissions": {
    "allow": [
      "Bash(npm test *)",
      "Bash(npm run dev)",
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(git log *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(sudo *)",
      "Bash(git push --force *)"
    ]
  }
}
```

## 14.4 团队协作

| 文件 | 用途 | Git |
|------|------|-----|
| `.claude/CLAUDE.md` 或 `CLAUDE.md` | 项目指令 | ✅ 提交 |
| `.claude/settings.json` | 共享权限和 hooks | ✅ 提交 |
| `.claude/rules/*.md` | 编码规范 (路径限定) | ✅ 提交 |
| `.claude/skills/*.md` | 领域专长技能 | ✅ 提交 |
| `.claude/commands/*.md` | 团队快捷命令 | ✅ 提交 |
| `.mcp.json` | MCP 服务器配置 | ✅ 提交 |
| `.claude/settings.local.json` | 个人覆盖 | ❌ gitignore |
| `CLAUDE.local.md` | 个人记忆覆盖 | ❌ gitignore |

## 14.5 Hooks 使用建议

| 事件 | 建议用途 |
|------|----------|
| `PreToolUse` | 安全守卫 — 阻止 `rm -rf`, `force push`, `sudo` |
| `PostToolUse` | 自动化 — lint, 格式化, 测试 |
| `SessionStart` | 加载上下文 — git 状态, TODO, 项目状态 |
| `Stop` | 验证 — 确认无遗留问题 |
| `Notification` | 自定义通知渠道 |
| `PreCompact` | 保存关键状态 |
| `SessionEnd` | 清理、版本标记 |

## 14.6 记忆管理原则

1. **聚焦"为什么"** — 记录决策原因而非代码细节
2. **定期审查** — 每月清理过时记忆
3. **MEMORY.md 保持200行以内** — 详细内容放主题文件
4. **一个文件一个话题** — 按需加载
5. **使用 feedback 记忆** — 每次纠正 Claude 后保存，避免重复犯错
6. **使用 reference 记忆** — 外部资源指针 (监控面板、文档链接)

## 14.7 上下文优化

1. 长对话中定期 `/compact`
2. Monorepo 用 `claudeMdExcludes` 排除无关子项目
3. 用 `@path/to/file` 精确引用而非整个目录
4. 启用 Tool Search 减少 MCP 工具定义占用
5. 大任务拆分多个会话，用 `/resume` 衔接

## 14.8 安全建议

- 始终在 `deny` 列表中加入 `rm -rf`, `sudo`, `force push`
- 敏感项目使用 `plan` 模式审查所有变更
- 用 PreToolUse hook 做额外安全层
- 定期用 `/security-review` 审计变更
- 隔离实验性工作: `claude -w experiment`
- 不要在 `settings.json` 中存储密钥 — 使用环境变量或 `.local.json`

## 14.9 效率技巧

```bash
# 管道直接分析日志
tail -f app.log | claude -p "实时监控并报告异常"

# 批量处理文件
ls *.md | claude -p "为每个文件生成目录"

# 生成 JSON 输出用于脚本
claude -p "列出所有 TODO" --output-format json | jq .

# 使用后台代理处理长任务
/agent 运行完整的测试套件并生成覆盖率报告
```
