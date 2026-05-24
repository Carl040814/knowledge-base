# [AI 安全（AI Security）](https://aiweb3.school/zh/handbook/bridge/ai-security)

> 约 7 分钟阅读 · 2026-05-12

**核心原则**：AI×Web3 安全不是"AI 安全"和"Web3 安全"简单叠加。当 Agent 有工具和钱包时，Prompt Injection 可能直接变成资产损失。

## 第一性原理

AI 安全在 Web3 场景里必须从"模型层面"延伸到"工具、权限和链上执行"。

- **模型输入不可信**：用户消息、网页内容、合约返回值都可能含恶意指令
- **工具是攻击面**：能调用的工具 = 能到达的攻击面
- **链上不可逆**：AI 错误产生的链上操作不像聊天可以撤回

## 知识节点

| 节点 | 说明 |
|------|------|
| **Prompt Injection** | 恶意输入覆盖系统指令。在 Web3 里更危险：恶意网页让 Agent 调用合约、假 quote 让 Agent 批准恶意交易。防御：输入隔离、工具权限分级、高风险动作强制 human check |
| **Tool Access Control** | 不是每个 Agent 都该拿到所有工具。按任务/风险/场景限制工具集。Payment 工具只在用户授权预算后可用，Wallet 工具不能暴露私钥 |
| **Output Validation** | Agent 输出需要验证层。生成的 calldata 要模拟、交易要查 chain id/合约地址/金额/滑点。不是"模型生成什么就执行什么" |
| **Sandbox** | 高风险 Agent 应在受限环境运行。限制：网络、地址、合约、金额、频率。Agent 破了沙箱 = 攻击成功 |
| **Rate Limiting** | 限制 Agent 调用工具的频率。防无限循环、恶意诱导重复扣款、异常高频调用。和预算系统配合 |
| **Incident Response** | 出问题后的处理：暂停 Agent、撤销 session key、冻结预算、审计日志、通知用户。事故预案比"不出错"更现实 |

## 在 AI × Web3 中的位置

AI Security 贯穿整个 Bridge 层。Agent Wallet 提供权限边界，Workflow 提供停止条件，Tool Use 提供工具隔离，Verifiable AI 提供事后可审计性。

## 最小实践

写一个 Agent 安全威胁模型：列出 Agent 的工具和权限、每个工具的潜在滥用方式、如何检测和拦截、出问题后如何停止和恢复。

---
