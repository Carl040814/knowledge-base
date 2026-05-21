# [链感知上下文（Chain-aware Context）](https://aiweb3.school/zh/handbook/bridge/chain-aware-context)

> 约 5 分钟阅读 · 2026-05-12

**核心原则**：Chain-aware Context 让 AI 在回答或行动前能看到正确的链、地址、合约、交易、余额和数据来源，而不是只靠用户一句话猜测链上状态。

普通 AI 的上下文来自文档和聊天历史；AI x Web3 多了一层：链上状态持续变化，且直接关联资产和权限。如果 Agent 不知道当前 chain id、合约地址、授权状态、数据更新时间，就可能给出错误建议甚至危险交易。

## 第一性原理

模型不能凭语言记忆判断链上事实，链上事实必须从工具和索引层读取。

模型知道"Uniswap 是 DEX"没用，真正执行时需要：具体网络、具体合约、具体池子、当前价格、用户余额、allowance、滑点和交易模拟结果。

- **链上状态有时间性**：同一地址的余额、授权和仓位会随区块变化
- **上下文要带来源**：合约地址、区块号、交易哈希、explorer 链接都应可追溯
- **区分事实和解释**：工具返回事实，模型负责解释，不要把模型猜测当事实

## 知识节点

| 节点 | 解释 |
|------|------|
| **On-chain Data** | 链上可直接验证的数据（余额、交易、日志、合约状态、区块信息）。来源：RPC、区块浏览器、索引器、协议 API。Agent 读取时必须带：chain id、block number、contract address、method、返回值、读取时间 |
| **Contract Docs** | 帮助模型理解合约设计意图、参数含义、权限边界。ABI 只给函数签名，不给业务语义。文档（NatSpec、README、审计报告）补语义，但可能过期——要用链上数据验证 |
| **ABI / Event** | ABI 让工具编码函数调用、解码返回值。Event 是合约留下的业务日志。能调用 ≠ 应该调用——写交易前还需权限、余额、allowance、滑点、simulation、policy 检查 |
| **Transaction History** | 帮助 Agent 理解过去操作。保留：tx hash、block number、from、to、method、value、token transfers、logs。模型可总结，但证据必须能回链上 |
| **Explorer Context** | 区块浏览器提供的可视化链上证据。给 explorer link 比给一句"交易成功"更可靠——用户可自己核验 |
| **Indexing Context** | 索引层整理链上事件为可查询数据。必须带时间戳和同步状态——落后 500 个区块的索引结果不能当当前事实 |
| **Citation** | 让模型回答能回到具体链上证据：交易哈希、区块号、合约地址、event log、explorer 链接。没有 citation 的链上解释只是观点 |

## 好的链感知上下文包应包含

1. 用户目标
2. 当前 chain id 和网络名称
3. 用户地址和余额
4. 相关合约地址、ABI、文档和风险提示
5. 最近交易和授权
6. 索引数据更新时间
7. 每条关键结论的 citation

## 在 AI × Web3 中的位置

Chain-aware Context 是所有链上 Agent 的输入层。没有这层，Web3 Tool Use、Agent Workflow、Agent Wallet 都建立在不可靠上下文上。

## 最小实践

1. 找一笔公开交易哈希
2. 收集 chain id、block number、from、to、method、value、token transfers、logs
3. 找到合约 ABI 或 verified source
4. 写一段模型可读的上下文，每个关键结论附上交易哈希或 explorer 链接
5. 标出哪些是链上事实，哪些是你的解释

---
