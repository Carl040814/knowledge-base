# 09 · 子代理系统

## 9.1 概述

子代理是 Claude Code 将独立任务委派给专门代理执行的能力。子代理可在隔离的 git worktree 中运行，支持并行和后台模式。

## 9.2 代理类型

| 类型 | 可用工具 | 适用场景 |
|------|----------|----------|
| `claude` | 全部 | 通用任务、代码编写 |
| `Explore` | 只读 (Read, Grep, Glob, Bash) | 代码探索、搜索 |
| `Plan` | 全部 (除 Agent/Edit/Write) | 架构设计 |
| `general-purpose` | 全部 | 复杂多步骤任务 |
| `claude-code-guide` | Read, Bash, WebFetch, WebSearch | Claude Code 问题 |

## 9.3 使用方式

### 会话内

```bash
# 创建子代理处理独立任务
/agent 搜索所有使用 JWT 的文件并报告安全问题

# 管理代理
/agents    # 查看所有代理状态

# 批量并行重构
/batch 同时重构 src/api/ 下 5 个模块
```

### 批量代理 (`/batch`)

- 启动 5-30 个代理并行工作
- 每个代理在隔离 worktree 中运行
- 结果汇总后统一报告
- 适合: 批量重构、多模块迁移、大规模清理

### 后台代理

- 非阻塞运行，完成后自动通知
- 适用: 长时间任务、CI 监控、定时检查

## 9.4 隔离模式

使用 `--worktree` 标志为代理创建隔离环境:

```bash
# CLI 启动时
claude -w feature-x

# 会话内
/worktree feature-x
```

worktree 中的代理:
- 拥有独立的 git 工作树
- 不影响主工作区
- 完成后可选择保留或清理

## 9.5 代理最佳实践

- **探索任务** → 用 `Explore` 代理（只读安全，并行高效）
- **代码审查** → 用独立代理获得无偏见意见
- **批量操作** → `/batch` 将大任务拆分为并行小块
- **长时间任务** → 后台代理，不阻塞主会话
- **敏感操作** → 在 worktree 中隔离执行

## 9.6 Agent SDK

Claude Code 的代理系统也通过 SDK 暴露给开发者:

- 编程式创建和管理代理
- 自定义代理类型和工具集
- 集成到 CI/CD 流水线
- 参考: [Claude Agent SDK 文档](https://code.claude.com/docs/en/agent-sdk)
