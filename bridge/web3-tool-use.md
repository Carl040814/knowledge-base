# [Web3 工具调用（Web3 Tool Use）](https://aiweb3.school/zh/handbook/bridge/web3-tool-use)

> 约 6 分钟阅读 · 2026-05-12

**核心原则**：Web3 Tool Use 是把 RPC、合约读取、交易生成、钱包确认、区块浏览器和 DeFi 操作变成 Agent 可调用工具的过程。真正难的不是"能调用"，而是权限、参数、模拟和日志。

Web3 工具的风险比普通查询工具更高：读错数据会误导判断，写错交易会改变资产和权限。

## 第一性原理

模型可以选择工具，但工具必须用确定性边界限制模型。

不要让 Agent 直接拼接任意 calldata 或调用任意地址。工具应该把危险能力封装成受限接口，并在执行前检查网络、地址、额度、方法、模拟结果和用户确认。

- **读写分离**：读取链上状态和发送交易必须是不同工具、不同权限
- **参数结构化**：chain id、contract address、method、args、value、slippage 不能埋在自然语言里
- **日志不可省**：每次工具调用都要记录输入、输出、时间、来源和错误

## 知识节点

| 节点 | 风险 | 说明 |
|------|------|------|
| **RPC Tool** | 低（只读）/ 高（写入） | 读取链状态、查询区块、估算 gas。只读可开放更宽；写入能力必须拆出去，不能混在"万能 RPC"里。返回应含 chain id、provider、block number、method、result、error |
| **Contract Read** | 低 | 调用 view/pure 函数（余额、allowance、owner、pool 状态）。最常用最安全。但仍可能误导：读错网络、ABI 不匹配、RPC 数据滞后 |
| **Contract Write** | **高** | 改变链上状态。需：chain id + 合约地址、ABI method + args、value/token 预估、gas 估算、simulation、policy 检查、用户/Smart Account 授权、tx hash 追踪。**不应给 Agent 任意合约写入能力**——限制白名单合约、方法和额度 |
| **Wallet Tool** | **最高** | 连接账户、请求签名、发送交易、管理授权。必须分清楚"连接""签名""发送交易""授权 token""撤销"。AI 生成的交易草稿不能绕过钱包确认 |
| **Explorer Tool** | 低 | 查询交易、合约源码、event、token transfer。提供可验证证据 |
| **DeFi Tool** | **高** | 封装 swap、借贷、仓位查询等。必须：协议白名单、最大交易额、滑点上限、价格来源、simulation、allowance 检查、人工确认 |
| **Tool Permission** | — | 定义 Agent 能调什么工具、什么条件下、传什么参数。按工具/合约/方法/金额/时间/频率/确认级别分层 |
| **Tool Log** | — | 可审计基础。每次调用记录：用户目标、工具名、输入、输出、错误、时间、chain id、block number、tx hash、确认人、policy 判断 |

## 权限分层示例

| 动作 | 权限 |
|------|------|
| 查询余额 | 自动允许 |
| 生成交易草稿 | 自动允许 |
| 小额白名单支付 | session key 允许 |
| 大额转账或授权 | **必须人工确认** |
| 任意合约调用 | 默认禁止 |

## 在 AI × Web3 中的位置

从"AI 能解释链上信息"走向"AI 能参与链上执行"的关键层。如果工具只读，风险是解释错误；如果工具能写链，风险进入资产和权限层。

## 最小实践

1. 写出一个只读工具：读取某地址在某链上的 ETH 余额
2. 写出一个合约读取工具：读取 ERC-20 allowance(owner, spender)
3. 写出一个交易草稿工具：生成 ERC-20 approve calldata（不发送）
4. 写出写交易工具的权限规则：只允许特定 token、特定 spender、最大额度
5. 为每个工具定义输入 schema、输出字段、错误类型、日志字段

---
