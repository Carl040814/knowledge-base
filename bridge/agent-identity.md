# [Agent 身份（Agent Identity）](https://aiweb3.school/zh/handbook/bridge/agent-identity)

> 约 7 分钟阅读 · 2026-05-12

**核心原则**：Agent Identity 不是起名字——是让用户/服务/其他 Agent 能验证：它是谁、谁控制它、能提供什么能力、服务入口在哪、历史记录能否追溯。

## 第一性原理

Agent 身份必须绑定控制权、能力声明和服务入口。回答：谁拥有这个 Agent、能做什么、如何调用、使用哪些钱包/密钥、历史声誉在哪。

- **身份要可解析**：从 identifier 找到 profile 和 endpoint
- **控制权要可证明**：更新 profile 或收款的必须是 owner
- **能力要可验证**：能力声明需要测试、证明、评价或历史记录支撑

## 知识节点

| 节点 | 说明 |
|------|------|
| **Agent Profile** | 公开说明：名称、描述、服务范围、价格、接口、钱包地址、能力列表、模型说明、隐私政策、owner。同时给人读和机器解析，更新历史留痕 |
| **Capability** | 描述 Agent 能完成什么任务。绑定 schema、价格、限制、测试记录、失败条件。标记风险等级（只读低/生成中/自动执行高） |
| **Service Endpoint** | 外部调用入口：HTTPS API/A2A/MCP server/Webhook。需认证、限流、版本、可用性和日志。endpoint 更新需 owner 签名 |
| **Registry** | 登记/发现/更新 Agent 身份。链上 registry 提供公开可查锚点（agent id/owner/profile URI/endpoint/更新记录）。能证明"谁注册的"，不能证明"一定好用" |
| **DID / VC** | 去中心化身份+可验证声明。DID 跨平台身份，VC 承载能力证明/组织隶属/审计通过。VC 可信度取决于 issuer |
| **A2A** | Agent-to-Agent 协议：发现、通信、协商任务、交换结果。身份系统告诉你"和谁说话"，A2A 负责"怎么协作" |
| **Ownership** | 谁控制 Agent profile/收款地址/endpoint 更新。高价值 Agent 不应由单热钱包控制。operator（运行服务）和 owner（控制身份）可分离 |

## 在 AI × Web3 中的位置

Agent Identity 是 Agent Trust、Machine Payment、Agentic Commerce 的前置条件。身份是信任系统第一层：先知道对象是谁，再看做过什么、谁评价过、是否有 stake。

## 最小实践

设计一个 Agent Profile：名称/描述/owner/endpoint、3 个 capability（含输入/输出/价格/限制）、profile URI、更新权限和通知机制、endpoint 归属证明。

---
