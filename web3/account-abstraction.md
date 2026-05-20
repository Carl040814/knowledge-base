# [账户抽象（Account Abstraction）](https://aiweb3.school/zh/handbook/web3/account-abstraction)

> 约 5 分钟阅读 · 2026-05-12

**核心原则**：账户抽象把"账户如何验证操作、谁来付 gas、哪些权限可以自动执行"从固定的 EOA 模式里释放出来，让钱包更像可编程账户系统。

## 第一性原理

当账户本身可以编程，权限就可以从"有私钥/没私钥"变成"在什么条件下允许什么动作"。

这对 AI x Web3 特别重要。Agent 不应该拿用户主私钥，也不应该拥有无限交易权限。更合理的方式是给 Agent 一个可限制、可过期、可撤销、可审计的行动空间。

- **验证逻辑可定制**：账户可以用多签、Passkey、社交恢复或模块规则验证操作
- **支付逻辑可定制**：gas 可以由用户、应用、paymaster 或其他资产承担
- **权限可以最小化**：session key 可以只允许特定合约、额度、时间和方法

## 知识节点

| 节点 | 解释 |
|------|------|
| **ERC-4337** | 以太坊最重要的账户抽象标准。用户创建 UserOperation → Bundler 收集 → EntryPoint 合约验证和调用智能账户。流程：生成 UserOp → 智能账户验证签名/nonce/策略 → Bundler 打包 → EntryPoint 执行 → Paymaster 可选赞助 gas |
| **Smart Account** | 由合约控制的账户，可规定：多签转大额、小额自动通过、dApp 限额调用、恢复人找回、批量交易。风险：合约 bug、模块权限、升级逻辑、外部依赖都会变成账户风险 |
| **Bundler** | 收集 UserOperation，模拟验证后提交 EntryPoint。类似交易打包服务。Bundler 不稳定 → 用户操作卡住；模拟不充分 → 失败交易 |
| **Paymaster** | 允许第三方为用户付 gas，或用非原生资产承担费用。适合 onboarding、活动补贴。需风控：赞助哪些方法、每用户额度、是否限制目标合约、防 spam |
| **Session Key** | 给应用或 Agent 的临时权限。可限制：时间段、目标合约、允许方法、额度、链 ID。是 Agent Wallet 的关键基础——Agent 自动执行低风险动作，高风险仍需用户确认 |

## 在 AI × Web3 中的位置

Account Abstraction 是 AI Agent 上链执行的重要底座。没有它，Agent 只能"给建议"或"让用户每一步都签名"。有了智能账户、Paymaster 和 Session Key，Agent 才可能在受限范围内自动执行。

但越自动化，越需要清楚的 policy：能调用什么、额度多少、多久过期、谁能撤销、日志在哪、失败怎么处理。账户抽象不是让 AI 更自由，而是让 AI 的自由被规则包起来。

## 最小实践

设计一个 Agent Session Key 策略：
1. 选择一个具体场景（如"每小时最多再平衡一次测试网小额资产"）
2. 写清允许的合约地址和方法
3. 设置额度、过期时间、链 ID、最大交易次数
4. 写出哪些动作必须回到用户钱包确认
5. 说明如何撤销 session key、如何审计执行记录

---
