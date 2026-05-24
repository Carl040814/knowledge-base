# 12 · 模型与上下文管理

## 12.1 模型选择

| 别名 | 完整模型 | 特点 |
|------|----------|------|
| `sonnet` | claude-sonnet-4-6 | 平衡性能与速度（默认） |
| `opus` | claude-opus-4-7 | 最强推理能力 |
| `haiku` | claude-haiku-4-5 | 最快响应速度 |

### 切换方式

```bash
# CLI 参数
claude --model opus

# 环境变量
export ANTHROPIC_MODEL=sonnet

# 会话内
/model opus
```

### 限制可用模型

```json
{
  "availableModels": ["claude-sonnet-4-6", "claude-haiku-4-5"]
}
```

### 模型映射 (Bedrock 等)
```json
{
  "modelOverrides": {
    "claude-sonnet-4-6": "arn:aws:bedrock:..."
  }
}
```

## 12.2 快速模式 (Fast Mode)

- `/fast` 切换
- 同模型下更快输出（不减配到小模型）
- 可设置 `fastModePerSessionOptIn: true` 要求每会话确认

## 12.3 扩展思考 (Extended Thinking)

- `alwaysThinkingEnabled: true` — 全局启用
- `/effort low|medium|high|max` — 会话内调整推理深度
- `showThinkingSummaries: true` — 显示思考摘要

## 12.4 计划模式 (Plan Mode)

- `/plan` 手动进入
- Claude 先读代码出方案，审查后才执行
- 适合: 大型重构、架构变更、多文件修改

## 12.5 上下文管理

### 上下文窗口
- Claude 模型从 200K tokens 起步
- 对话历史 + 代码文件 + 工具输出共享此窗口
- 接近上限时系统提示压缩

### 关键命令

```bash
/context    # 可视化上下文使用（颜色网格 + 优化建议）
/compact    # 压缩对话历史释放空间
/clear      # 完全清空 (等效 /reset, /new)
```

### 优化策略

1. **定期压缩** — 长对话中用 `/compact`
2. **排除无关 CLAUDE.md** — `claudeMdExcludes` 过滤
3. **精确引用** — 用 `@path/to/file` 而非整个目录
4. **启用 Tool Search** — 减少 MCP 工具定义占用
5. **拆分会话** — 大任务分多个会话，中间用 `/resume` 衔接
6. **使用计划模式** — 复杂任务先规划，避免反复试错浪费上下文

### 上下文可视化

`/context` 输出带颜色网格的视图:
- 🟢 绿色 — 系统提示
- 🟡 黄色 — 用户消息
- 🔵 蓝色 — 助手响应
- 🟣 紫色 — 工具输出
- 空白 — 可用空间

底部显示优化建议（可压缩哪些部分）。
