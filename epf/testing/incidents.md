# 著名主网事件 (Notable mainnet incidents)

> :warning: 本文是一个待完善的草稿 (stub)，欢迎通过 [做出贡献](/contributing.md) 并对其进行扩展来帮助维基。
> 事件是指网络所面临的中断、漏洞和攻击，这些事件曾让人们对其稳定性和安全性产生过疑问。以下是影响网络的一些事件及其来源。

## 概述 (Overview)

以太坊网络在其历史上面临过各种挑战和事件。通过仔细的分析和预防措施的实施，这些事件帮助提高了网络的弹性和安全性。本页面记录了影响以太坊的著名事件。

如需查看以太坊事件的完整列表及其详细分析，您可以参考 [EthStaker 事件页面 (EthStaker Incidents Page)](https://ethstaker.org/incidents)。

## 近期事件 (Recent Incidents)

- [Holesky 最终性问题事后分析 (Post-Mortem, Holesky Finality Issue) - 2025/02/24](https://github.com/ethereum/pm/blob/master/Pectra/holesky-postmortem.md)
  在 2025 年 2 月 Holesky 测试网进行 Pectra 升级后，由于许多执行层 (EL) 客户端配置了错误的存款合约地址，导致区块无法达成最终性 (finality)。这导致部分执行层客户端拒绝了无效区块，而其他客户端接受了它们，从而导致了网络分裂 (network split)。

- [Blob 传播问题事后分析 (Post-Mortem, Blob Propagation Issues) - 2024/03/27](https://gist.github.com/benhenryhunter/687299bcfe064674537dc9348d771e83)
  在 2024 年 3 月 Dencun 升级后，来自某些构建者 (builders) 的区块所附带的 Blob 在点对点 (p2p) 网络上传播速度过慢，导致某客户端实现错过了几个时隙 (slots)。

- [主网拒绝服务 (DOS) 事件事后分析报告：以太坊主网拒绝服务事件 - 2024/02/07](https://blog.ethereum.org/2024/03/21/sepolia-incident)
  研究发现，从合并 (Merge) 发生到 Dencun 硬分叉 (hard fork) 期间，存在遭受拒绝服务攻击 (Denial-of-service, DOS) 的可能性。攻击者可以创建一个超过 5MB 指定限制的区块，并在区块中添加多个大小均不超过 128KB 的交易，同时确保区块内交易的总体 Gas 低于 3000 万。通过这种操作，大多数节点会拒绝这些区块，从而导致少数派节点接受它们，产生分叉区块并导致提议者错失奖励。

- [主网最终性事后分析报告：以太坊主网最终性 - 2023/11/05](https://medium.com/offchainlabs/post-mortem-report-ethereum-mainnet-finality-05-11-2023-95e271dfd8b2)
  主网发生了一些中断，导致无法出块，进而导致交易达成最终性 (finality) 出现严重延迟。这种情况持续了两天，并导致了不活跃惩罚 (inactivity leak)，网络最终在没有人工干预的情况下完全恢复。

- [Reth 主网状态根不匹配 (Reth Mainnet State Root Mismatch) - 2025/09/01](https://laced-king-de5.notion.site/Incident-Post-Mortem-Reth-Mainnet-State-Root-Mismatch-26732f2c348480dea8b8c2a8753696dc)
  Reth 对特里树更新 (trie updates) 处理的漏洞导致 Reth 节点中的 trie 表包含错误信息，致使节点在后续区块计算出错误的状态根 (state root)。

## 历史事件 (Historical Incidents)

- [少数派分裂事后分析报告 (Post-Mortem Report: Minority Split) - 2021/08/27](https://github.com/ethereum/go-ethereum/blob/master/docs/postmortems/2021-08-22-split-postmortem.md)
  当 Geth 尝试在 `datacopy` 操作之后将数据分配回内存时发生了这一事件。它没有将数据保存在新位置，而是意外覆盖了原始数据，导致其损坏。

- [The DAO 攻击 - 2016](https://www.coindesk.com/learn/understanding-the-dao-attack)
  以太坊历史上最重大的事件之一，其中 The DAO 智能合约 (smart contract) 中的一个漏洞被利用，导致损失了大约 360 万个 ETH。该事件最终导致以太坊区块链的硬分叉 (hard fork)，从而产生了以太坊经典 (Ethereum Classic, ETC) 和现在的以太坊 (Ethereum, ETH) 链。

- [上海拒绝服务 (DOS) 攻击 - 2016](https://ethos.dev/shanghai-attacks)
  在上海举办的 DevCon2 期间，网络面临了一系列拒绝服务 (DOS) 攻击，攻击者利用价格过低的 EVM 操作码 (EVM opcodes)（特别是 EXTCODESIZE）来减慢区块处理速度，导致网络拥堵。这导致了随后的硬分叉 (hard forks)（Tangerine Whistle 和 Spurious Dragon），调整了针对性操作码的 Gas 成本，以防止类似攻击。
