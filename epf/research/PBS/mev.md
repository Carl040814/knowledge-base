# 最大可提取价值 (Maximal Extractable Value) (前身为 矿工可提取价值 (Miner Extractable Value))

最大可提取价值 (Maximal Extractable Value, MEV) 是指通过在区块 (block) 中策略性地排序 (strategically ordering)、包含 (including) 或排除 (excluding) 交易 (transactions)，从区块生产 (block production) 中提取的超出标准区块奖励 (block reward) 和 Gas费用 (gas fees) 的最大价值。

在以太坊 (Ethereum) 中，随着验证者 (validators) 提取越来越多价值，特别是在去中心化金融 (Decentralized Finance, DeFi) 应用中，MEV 获得了更多关注。通过在区块中对交易进行排序，诸如抢跑 (front-running)、夹子攻击 (sandwiching) 或尾随 (back-running) 等策略促进的套利 (arbitrage) 机会得以实现。这也会导致负面后果，例如大型资金池 (pools) 的不公平优势、审查 (censorship) 或去中心化金融 (DeFi) 用户滑点 (slippage) 的增加。

[提议者-构建者分离 (Proposer-Builder Separation, PBS)](/wiki/research/PBS/pbs.md) 可以改变 MEV 提取的动态，即在这两个角色之间可能会对 MEV 进行重新分配，从而可能改变与每个角色相关的激励 (incentives) 和奖励 (rewards)。由于区块构建者 (block builders) 负责交易排序 (transaction ordering) 和包含，他们可能会开发 (develop) 新的策略或促进竞争，从而提高效率并在整个网络 (network) 中更公平地分配 MEV。

## MEV 的演变 (Evolution of MEV)

最大可提取价值 (Maximal Extractable Value, MEV) 起源于工作量证明 (proof-of-work) 时代，当时被称为“矿工可提取价值 (Miner Extractable Value)”。这一术语反映了矿工 (miners) 影响区块中交易顺序的能力，包括它们的包含、排除和排序。随着以太坊通过合并 (The Merge) 过渡到权益证明 (proof-of-stake)，验证者 (validators) 接管了共识 (consensus)，使协议 (protocol) 内的挖矿 (mining) 变得过时。尽管发生了这些变化，价值提取的机制依然存在，导致人们采用了“最大可提取价值 (Maximal Extractable Value)”这一术语来指代这些活动。

尽管自以太坊诞生以来 MEV 就一直存在，但随着去中心化金融 (DeFi) 的兴起和更易于使用的工具的出现，它获得了巨大的关注。在早期，MEV 机会主要是通过在公开内存池 (mempool) 中出价比竞争对手更高来获取的，这标志着被称为优先Gas拍卖 (Priority Gas Auction, PGA) 的时代。关于这个混乱时代的细节记录在 [Flashboys 2.0](https://arxiv.org/abs/1904.05234) 中。在那段时间里，[Flashbots](https://www.flashbots.net/) 作为一个开放的研发倡议 (R&D initiative) 出现，旨在提高公众对 MEV 工具的认知和获取渠道。

在合并后 (Post-Merge) 的世界中，矿工的概念不复存在，但他们的构建者 (builder) 和提议者 (proposer) 角色由验证者 (validators) 担任，后者以同样的方式负责向链中添加区块。预见到合并后的变化，Flashbots 与客户端团队 (client teams) 和以太坊基金会 (Ethereum Foundation) 开始开发 [mev-boost](/wiki/research/PBS/mev-boost.md)。MEV-boost 是提议者-构建者分离 (Proposer-Builder Separation) 的一种协议外 (out-of-protocol) 实现。请参阅 [关于 PBS 的章节](/wiki/research/PBS/pbs.md)。

## 资源 (Resources)

- [从 CryptoKitties 到 MEV-Boost 再到 PBS 的交易供应链研究 (A study of the transaction supply chain from CryptoKitties to MEV-Boost to PBS) - Barnabé Monnot](https://www.youtube.com/watch?v=jQjBNbEv9Mg)
- [MEV Day 视频列表 (MEV Day playlist)](https://www.youtube.com/playlist?list=PLRHMe0bxkuel3w3C7P_WVvp9ShLi3HKRI)
- [如何照亮黑暗森林 (How to light up dark forest) - Flashbots](https://collective.flashbots.net/t/how-to-light-up-the-dark-forest-a-walkthrough-of-building-a-cryptopunk-mev-inspector/881)