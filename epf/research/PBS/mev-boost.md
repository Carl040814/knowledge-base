# Mev-boost：一种流行的 PBS 实现 (Mev-boost: A popular PBS Implementation)

[Mev-boost](https://github.com/flashbots/mev-boost) 是一个广为人知的、协议外的 (out-of-the-protocol)、开源的 (open-source) 以太坊 提议者-构建者分离 (Proposer-Builder Separation, PBS) 实现。它允许 验证者 (validators) 将区块构建外包给专业的 构建者 (builders)，这可能会带来更高的验证者奖励并提高网络效率。

以下是 mev-boost 的工作原理：

![Block-building](https://epf.wiki/images/pbs/block-building.png)
*当前区块构建。来源：[Barnabé 的文章](https://barnabe.substack.com/p/pbs)*

一方面，mev-boost 实现了以太坊节点 (Ethereum node) 用于外包其区块生产 (block production) 的 [构建者 API (builder API)](https://github.com/ethereum/builder-specs)。另一方面，它连接到一个 中继 (relays) 网络，并处理构建者与提议者之间的通信。

1. **区块构建 (Block Building)：**
   专业的构建者相互竞争，为提议者创建利润最高的区块。他们通过优化 交易排序 (transaction ordering) 和 包含 (inclusion)，并考虑 Gas 费用 (gas fees)、交易优先级 (transaction priority) 和潜在的 [最大可提取价值 (Maximal Extractable Value, MEV)](/wiki/research/PBS/mev.md) 等因素来实现这一点。
   构建者将他们构建的区块提交给中继。
2. **中继网络 (Relay Network)：**
   Mev-boost 运营着一个中继网络，充当构建者和提议者之间的中介。
   中继从构建者那里接收区块，并执行区块验证 (block validation)、过滤 (filtering) 和传播 (propagation) 等各种功能。
   中继确保只有有效且高质量的区块才会发送给提议者。
3. **提议者选择 (Proposer Selection)：**
   验证者运行连接到其 信标节点 (beacon node) 的 mev-boost 软件。当一个验证者被选中提议区块时，他们会从中继接收区块，并根据预定义标准（通常是提供最高奖励的区块）选择最好的一个。
   然后，验证者将选定的区块提议给网络，以进行验证并包含在 区块链 (blockchain) 中。

## PBS 区块创建 (PBS Block Creation)

通过 PBS 创建区块的过程如下：

### 区块构建 (Block Construction)

- 构建者持续监视 交易池 (transaction pool, mempool) 中的新交易。他们根据潜在的 MEV 机会对这些交易进行评估。他们选择最符合其 MEV 优化标准的交易。此外，区块构建者可以从 私有订单流 (private orderflows) 或 MEV 搜索者 (MEV searchers) 那里获取 交易捆绑包 (transaction bundles)，就像矿工在工作量证明以太坊 (PoW Ethereum) 中通过最初的 Flashbots 拍卖所做的那样。在后一种情况下，构建者接受搜索者的 密封价格出价 (sealed-price bids)，并将他们的捆绑包包含在区块中。
- 一旦选定了交易，构建者就会将它们组装成一个区块，确保该区块符合以太坊协议的规则，例如交易是有效的，没有超过 Gas 限制等。

### 区块拍卖 (Block Auction)

与构建者直接以指定价格向验证者提供其组装好的区块不同，标准做法是使用中继。中继在将 区块头 (Headers) 传递给 提议者 (proposer)（验证者）之前，先验证交易捆绑包。提议者接受并用他们的密钥对区块头进行签名，然后将 已签名区块头 (SignedHeader) 返回给中继。此时，中继将 完整区块 (Full Block) 释放给提议者。此外，各种实现还可以引入 托管机构 (escrows)，负责通过存储构建者发送的区块和验证者发送的承诺来提供 数据可用性 (data availability)。

### mev-boost 的优势：

- **增加验证者奖励：** 通过将区块构建外包给专业的构建者，验证者可以通过优化的交易排序和 MEV 提取潜在地获得更高的奖励。
- **减少中心化：** Mev-boost 实现了一个更具竞争力的区块构建格局，减少了大型 矿池 (mining pools) 的规模经济效应，并使 家庭质押者 (home stakers) 能够获得同等水平的奖励。

### 挑战与考量：

虽然 mev-boost 带来了一定的益处，但它也引发了一些担忧：

- **安全性：** 引入新的参与者和依赖关系可能会制造新的 攻击向量 (attack vectors) 和 漏洞 (vulnerabilities)。由于 mev-boost 问题，主网 (mainnet) 上已经发生过多次丢失区块的 [事件](https://collective.flashbots.net/t/post-mortem-april-3rd-2023-mev-boost-relay-incident-and-related-timing-issue/1540)。
- **抗审查性 (Censorship resistance)：** 如果只有少数强大的构建者或中继主导了生态系统，这可能会引发中心化和审查方面的担忧。
- **协调：** 构建者、中继和提议者之间的高效沟通与协调对于 mev-boost 的顺利运行至关重要。

需要注意的是，mev-boost 只是 PBS 的一种实现。其他具有不同设计和特性的实现也正在被开发和探索中，例如 [mev-rs](https://github.com/ralexstokes/mev-rs) 正在开发中。

总体而言，mev-boost 代表了在以太坊中实现 PBS 潜在益处的重要一步。然而，持续的研发对于解决这些挑战并确保安全、去中心化且高效的实现至关重要。通往更稳定的 PBS 模型的一条途径是 [将其封装在协议中 (enshrining it in the protocol)](/wiki/research/PBS/ePBS.md)，将类似于 mev-boost 的特性直接添加到以太坊客户端 (clients) 中。

## 资源 (Resources)

- [Flashbots 关于 mev-boost 的文档 (Flashbots docs on mev-boost)](https://boost.flashbots.net/)
- [审查实体概述 (Overview of censoring entities)](https://censorship.pics/) 
- https://www.mevwatch.info/
- [MEV-Boost：适用于合并的 Flashbots 架构，2021 年 (MEV-Boost: Merge ready Flashbots Architecture, 2021)](https://ethresear.ch/t/mev-boost-merge-ready-flashbots-architecture/11177)
