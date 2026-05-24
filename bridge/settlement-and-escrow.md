# [结算与托管（Settlement & Escrow）](https://aiweb3.school/zh/handbook/bridge/settlement-and-escrow)

> 约 7 分钟阅读 · 2026-05-12

**核心原则**：解决 Agent 经济里"钱什么时候释放、服务怎么算完成、失败怎么退款、争议怎么处理"。把支付从一次转账变成完整交易流程。

## 第一性原理

自动化交易必须有明确完成条件，否则支付无法安全自动化。Escrow 设计先定义状态机，再写付款代码。

- **资金状态要清楚**：pending、locked、released、refunded、disputed
- **交付证明要可保存**：结果、hash、日志、模型输出、交易哈希
- **争议流程要提前设计**：不要等失败后才讨论谁有权判定

## 知识节点

| 节点 | 说明 |
|------|------|
| **Escrow** | 资金锁在合约里等交付条件满足再释放。绑定任务 ID/付款方/服务方/金额/截止时间/验收规则/退款条件。状态机：Created→Funded→Delivered→Accepted→Released / Refunded→Disputed |
| **Receipt** | 支付+交付凭证。含：谁付谁、金额、币种、任务 ID、quote id、交易哈希、服务结果引用、验收状态。同时服务人和机器解析 |
| **Delivery Proof** | 证明服务方确实交付。可以是文件 hash、API 返回日志、模型输出签名、链上 event、TEE attestation。必须和原始任务对应 |
| **Acceptance** | 确认交付满足要求。拆成可检查条件：字段完整、测试通过、到期前提交、hash 匹配。AI 初审 + challenge window + 人工复核 |
| **Refund** | 交付失败/超时/取消时退款。触发条件：超时、格式错误、验收失败、任务取消、quote 过期。需考虑部分退款 |
| **Dispute** | 对交付质量的分歧处理。人工仲裁/多签/DAO/optimistic challenge。设计需答：谁发起、成本、证据格式、谁裁决、可上诉否 |
| **Evaluator** | 判断交付是否合格的角色。脚本/测试套件/模型/人工/多验证者。AI 初判+人复核。Evaluator 本身也需被评估 |
| **ERC-8183** | Agentic Commerce 标准草案。把 Agent Commerce 从"发一笔钱"推进到结构化交易模型：任务→状态→proof→settlement→dispute |

## 在 AI × Web3 中的位置

Settlement & Escrow 是 Machine Payment 的后半段。前者解决"如何付款"，这里解决"付款后如何确认价值交换完成"。

## 最小实践

设计"Agent 付费生成报告"的 escrow：锁定 2 USDC→服务方 10min 交付→提交 IPFS hash→Evaluator 检查指定字段→通过释放/失败退款→争议进入多签仲裁。

---
