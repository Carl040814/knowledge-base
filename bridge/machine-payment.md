# [机器支付（Machine Payment）](https://aiweb3.school/zh/handbook/bridge/machine-payment)

> 约 9 分钟阅读 · 2026-05-12

**核心原则**：Agent、API、服务之间如何自动完成报价→授权→付款→收据→预算控制。重点不是"AI 会花钱"，而是让机器支付可限制、可验证、可追踪。

## 第一性原理

Agent 不应该拥有无限支付能力，只应拿到具体任务、预算和收款方范围内的支付权限。

- **预算先于执行**：没有预算边界，就没有安全自动支付
- **报价必须可比较**：Agent 要知道价格、币种、有效期、服务范围、退款条件
- **收据必须可验证**：付款后证明付给谁、为什么付、交付了什么

## 知识节点

| 节点 | 说明 |
|------|------|
| **Stablecoin Payment** | 稳定币适合机器支付：价格稳定、结算快、可编程。区分"计价币种"和"结算币种"，Agent 不能只看"0.1"——必须知道是 0.1 USDC/USDT/ETH |
| **Budget** | 按时间/任务/服务方/币种/额度定义。多层：全局→任务→单次→服务方→紧急停止。常见错误：只设总预算不设频率和范围 |
| **Quote** | 服务方给 Agent 的可执行报价。含：服务内容、价格、币种、收款地址、有效期、交付条件、退款条件、quote id。过期 quote 不能被使用 |
| **Payment Intent** | "用户授权这类付款"，不等同于已结算。绑定任务/金额/收款方/有效期/可接受结果。是后续 payment/escrow/receipt 的上下文 |
| **x402** | HTTP 402 Payment Required → 互联网原生支付。Agent 像处理 401 登录一样处理 402 付款：读付款要求→查预算→付款→重放请求 |
| **MPP** | Machine Payments Protocol：机器间支付协商→结算协议化。服务发现/价格协商/支付凭证/交付回执/失败重试 |
| **Subscription** | 持续服务支付模型。需：可见当前授权、下次扣款时间、剩余额度、随时停止。小心"静默续费" |
| **Micropayment** | 高频小额服务：L2/payment channel/批量结算/预付余额。先算经济账：单次价值 vs 链上手续费 |

## 在 AI × Web3 中的位置

Machine Payment 是 Agentic Commerce 的基础。完整链路：用户授权预算→Agent 获取 quote→系统查 policy→付款 escrow/结算→服务交付→收据留证。

## 最小实践

设计 Agent 购买 API 支付流程：用户授权 3 USDC → API quote 0.1 USDC/5min → Agent 查预算和身份 → 钱包付款 → API 返结果+receipt → 系统记录 quote/intent/tx hash/剩余预算。

---
