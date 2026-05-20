# [索引（Indexing）](https://aiweb3.school/zh/handbook/web3/indexing)

> 约 4 分钟阅读 · 2026-05-12

**核心原则**：链上数据是公开的，但不等于好用。Indexing 的作用，是把区块、交易、事件和合约状态整理成产品、分析工具和 AI Agent 能快速查询的结构化数据。

链上是事实来源，索引层是可用数据层。

## 第一性原理

产品需要的是面向问题的数据模型，而不是原始区块流。

区块链按区块和交易组织数据，用户和产品却关心"这个地址的仓位""这个协议的 TVL""这个 Agent 执行过哪些动作"。索引层负责把底层事实转换成这些查询对象。

- **事件是重要入口**：合约 event 是索引器构建状态的主要信号
- **RPC 不是数据库**：RPC 适合读取链状态和发送交易，不适合承载所有复杂历史查询
- **索引要能重放**：合约升级、reorg、bug 修复时，需要从某个区块重新构建数据

## 知识节点

| 节点 | 解释 |
|------|------|
| **Event Indexing** | 监听合约日志，把链上动作整理成可查询记录。设计 event 时考虑：是否包含关键地址、是否需要 indexed 参数、能否从 event 还原业务状态、失败交易不会产生成功 event、合约升级后 event 兼容性 |
| **Subgraph** | The Graph 的声明式索引方式。三部分：要监听的合约和事件、事件到实体的 mapping、GraphQL schema。需要维护：合约地址变更、事件结构变化、reorg、同步延迟 |
| **RPC** | 应用和节点交互接口（eth_call、eth_getLogs、eth_sendRawTransaction）。常见问题：rate limit、节点不同步、archive 数据不可用、多 RPC 不一致、WebSocket 不稳定 |
| **Data Pipeline** | 完整链路：RPC/节点 → event listener → ABI 解码 → 数据库写入 → reorg 处理 → 数据校验 → API/GraphQL/vector store → dashboard/alert/Agent context |

## 在 AI × Web3 中的位置

AI Agent 需要上下文，链上上下文通常来自索引层。交易历史、合约事件、用户仓位、协议状态、风险信号，都不适合每次临时从原始区块搜索。

好的索引层给 Agent 提供结构化、带来源、带时间戳、可回溯的数据。模型负责解释推理，索引层负责提供事实。

## 最小实践

1. 选一个简单合约（投票/计数器/NFT mint）
2. 列出它应发出的 event
3. 设计一张查询表（votes/transfers/mints）
4. 标出每个字段来自哪个 event 参数或交易字段
5. 写出如何处理 reorg、重复事件和合约升级
6. 问：如果给 AI Agent 使用，需要附带哪些来源字段和更新时间？

---
