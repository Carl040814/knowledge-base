# [智能体商业（Agentic Commerce）](https://aiweb3.school/zh/handbook/tracks/agentic-commerce)

> 约 6 分钟阅读 · 2026-05-12

**核心原则**：当 Agent 能代表用户发现商品、调用 API、比较报价、发起付款和验收结果时，商业流程如何重新设计。关键不在"AI 自动买东西"，而在交易边界。

## 第一性原理

Agent 可以替用户发起商业动作，但不能替用户承担无限责任。把购买意图、预算、验收标准和争议规则变成结构化协议。

- **购买前要有 intent**：买什么、为什么买、最多花多少、接受什么结果
- **购买中要有约束**：报价、有效期、服务条款、数据使用范围、退款条件
- **购买后要有证据**：任务结果、收据、日志、验收记录和争议路径

## 知识节点

| 节点 | 说明 |
|------|------|
| **Agents Purchasing APIs** | Agent 动态发现服务并完成调用。流程：识别需求→查询可用服务/价格/能力→生成购买请求+预算→服务方返回报价→Agent 授权内付款调用→记录结果和费用 |
| **Payment Intent** | 用户授权 Agent 花钱的结构化表达：任务目标、预算（单次/日/总）、支付资产、可接受服务方、质量要求、退款条件、有效期。把用户意愿变机器可检查边界 |
| **Budget Control** | 分层：任务预算、服务预算、时间预算（时/日/周）、风险预算（高风险须确认）、失败预算（连续失败停止）。界面要可理解 |
| **Proof of Task Completion** | 不同服务的证据不同：API→请求ID+响应；数据→hash+版本；推理→模型版本+输出hash；执行→tx hash+event。低价值用日志，高价值加签名/链上事件/托管仲裁 |
| **On-chain Receipt** | 链上收据含：payer/provider/agent id、金额/资产/链/tx hash、服务类型/quote id、输出hash、验收/争议状态。跨平台结算和声誉的锚点 |

## 在 AI × Web3 中的位置

Agentic Commerce 是 Bridge 层的商业化落地。依赖 Machine Payment、Settlement & Escrow、Agent Trust & Reputation。

---
