# [Agent 工作流（Agent Workflow）](https://aiweb3.school/zh/handbook/bridge/agent-workflow)

> 约 5 分钟阅读 · 2026-05-12

**核心原则**：Agent Workflow 是把"用户目标 → 上下文读取 → 计划生成 → 工具调用 → 风险检查 → 执行 → 记录复盘"组织成可控流程，而不是让模型自由发挥。

Agent Workflow 的核心，是把概率模型放进确定性流程里。

## 第一性原理

高风险 Agent 不能只有"下一步推理"，必须有状态、边界和停止条件。

模型可以规划，但系统要知道当前到了哪一步、哪些工具已调用、哪些结果可信、哪些动作需要确认、失败后如何停止。

- **流程要显式**：不要把完整执行链路藏在一段长 prompt 里
- **状态要可恢复**：工具失败、用户拒绝、交易 pending 时，系统要知道如何继续或停止
- **评估要可回放**：没有 trace 和 regression set，很难知道改模型后是否更安全

## 知识节点

| 节点 | 说明 |
|------|------|
| **Task Graph** | 把目标拆成节点和依赖。例：swap 拆为 读取目标→获取余额→查价格→生成交易→模拟→展示风险→用户确认→发送→追踪。每步有输入、输出、权限和停止条件 |
| **State Machine** | 链上工作流状态：draft → context_loaded → plan_ready → simulation_failed → waiting_user_confirmation → submitted → confirmed → reverted → cancelled。用户刷新页面、交易 pending、RPC 失败时系统不迷失 |
| **Human-in-the-loop** | 把人在关键风险点介入，不是每步都确认。分层：只读自动 / 交易草稿自动 / 小额 session key / 高风险必须确认 / 超 policy 直接拒绝 |
| **Retry / Fallback** | 读余额失败可重试；发交易失败先判断是否已广播；pending 不能简单再发；RPC 异常可切 provider 但需记录来源。模型不可用时降级成只读模式 |
| **Trace** | 每步记录：用户目标、模型版本、上下文来源、工具输入输出、policy 判断、simulation、人工确认、tx hash、最终状态。无 trace 就只能看聊天记录 |
| **Evaluation Harness** | 不只测回答好不好，还要测：是否拒绝越权请求、是否识别错误链/合约、是否缺少数据时停止、是否要求 human check、是否记录 citation、是否避免生成危险 calldata |
| **Regression Set** | 固定测试用例防安全退化。包含：正常 swap、错误链请求、无限 approve、恶意诱导、余额不足、价格过旧、用户拒绝、pending 超时 |

## 一个完整的 Swap 工作流

```
读取用户目标和限制
    ↓
获取余额和 allowance
    ↓
查询价格和流动性
    ↓
生成候选交易
    ↓
模拟交易 ←── simulation 失败则回到上一步
    ↓
展示风险 ←── 人在这步确认
    ↓
发送交易
    ↓
追踪结果 → 记录 trace
```

## 在 AI × Web3 中的位置

Workflow 是桥接层的流程骨架。Chain-aware Context 提供事实，Web3 Tool Use 提供能力，Agent Wallet 提供权限边界——Workflow 把它们组织成可执行但可控的路径。

没有 workflow，项目就是"模型直接调用工具"——demo 很快，真实资产前不够用。

## 最小实践

1. 选择任务：解释并准备一笔小额 ERC-20 swap
2. 画出 task graph：读上下文→查价格→生成计划→模拟→确认→执行→记录
3. 为每步写输入、输出、可用工具和失败处理
4. 标出哪些步骤必须 human-in-the-loop
5. 写 5 个 regression case：正常、错误链、滑点过大、余额不足、用户拒绝

---
