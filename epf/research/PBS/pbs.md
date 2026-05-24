# 提议者-构建者分离 (Proposer-Builder Separation, PBS)

在以太坊当前的系统中，验证者 (validators) 既创建区块也广播区块。它们将通过流言网络 (gossip network) 发现的交易打包在一起，并将它们封装成一个区块，然后发送给以太坊网络上的同行。**提议者-构建者分离 (Proposer-Builder Separation, PBS)** 将这些任务拆分给多个验证者。区块构建者 (block builders) 负责创建区块，并在每个时隙 (slot) 中将其提供给区块提议者 (block proposer)。区块提议者无法看到区块的内容，它们只需选择利润最高的一个，在向区块构建者支付费用后，再将区块发送给同行。

本节将涵盖关于 PBS 的细节、区块提议者和区块构建者的角色、现状（MEV-boost、中继）、挑战和安全问题、拟议的解决方案以及与该主题相关的更深层资源收集。

## 为什么 PBS 很重要？ (Why is PBS important?)

PBS 对于以太坊的去中心化非常重要，因为它最小化了成为验证者所需的计算开销 (compute overhead)。通过这样做，网络降低了成为验证者的准入门槛 (barrier to entry)，并激励了更多样化的参与者群体。PBS 也反映了合并 (The Merge) 的总体目标，即推动以太坊网络走向更加模块化 (modular) 的未来。具体而言，向权益证明 (PoS) 的过渡是通过模块化走向去中心化的积极举措。

当您拆解区块构建的不同部分时，您可以单独对它们进行去中心化。这使得拥有不同专长的主体能够专注于自己的优势。最终结果是一个能力更强、外部依赖更少且参与门槛更低的网络。

## 理解 PBS 和共识层 (Understanding PBS and the Consensus layer)

正如这篇 [文章](https://ethos.dev/beacon-chain) 所解释的，时隙 (slots) 是共识层 (consensus layer) 中允许区块添加到链中的时间框架，它们持续 12 秒，每个时段 (epoch) 包含 32 个时隙。时段对于共识机制具有重要意义，它是网络最终确定区块、更新验证者委员会等的检查点 (checkpoints)。对于每个时隙，会通过 [RANDAO](https://inevitableeth.com/home/ethereum/network/consensus/randao) 选择一个验证者来提议区块。一旦提议并添加到规范链 (canonical chain) 中，为该时隙委员会选择的验证者就会对区块的有效性进行见证 (attest)，该区块最终将达到最终性 (finality)。共识层支持以太坊网络的安全性和完整性。PBS 通过划分/隔离提议区块和构建区块的职责与该层进行交互，从而简化了交易验证流程。

### 构建者的角色 (The Role of the builder)

**区块构建者 (Block builders)** 收集、验证交易并将其组装成区块体。它们审查内存池 (mempool)，通过确保交易满足 Gas 限制和随机数 (nonce) 等要求来验证交易，并创建一个包含交易数据的数据结构。区块构建者还负责对交易进行排序，以优化区块空间和Gas使用。然后，它们将区块体提供给区块提议者。

### 提议者的角色 (The Role of the proposer)

**区块提议者 (Block proposers)** 采用区块构建者提供的区块体，并通过添加必要的元数据（如区块头 (block header)）来创建完整的区块。区块头包含诸如父区块哈希 (hash)、时间戳和其他数据之类的细节。它们还通过检查构建者提供的区块体的正确性来确保区块的有效性。

## 现状 (Current State)

目前，PBS（提议者-构建者分离）存在于协议之外，构建者通过中继 (relays) 等实体协助进行区块构建。请参阅 [mev-boost](/wiki/research/PBS/mev-boost.md) 了解关于被广泛使用的协议外解决方案之一的更多细节。这种设计依赖于极少数受信任的中继和构建者，这引入了中心化风险，并使以太坊更容易受到审查。
PBS 尚未在以太坊主网 (mainnet) 中实现，这意味着验证者同时扮演提议者和构建者的角色。因此，每个验证者负责：

1. **选择交易**：验证者根据 Gas 费用 (gas fees) 和交易优先级 (transaction priority) 等因素选择要包含在区块中的交易。
2. **构建区块**：验证者将选定的交易组装到区块中，并执行必要的计算，例如验证签名 (verifying signatures) 和更新状态。
3. **提议区块**：验证者将构建的区块提议给网络，以进行验证并包含在区块链中。

然而，一些客户端正在积极开发和测试 PBS 实现。这些实现旨在分离构建者和提议者角色，允许验证者将区块构建外包给专业的构建者。这可以带来几个潜在的益处：

- **增加验证者奖励**：构建者竞相为提议者创建利润最高的区块，这可能为验证者带来更高的奖励。
- **提高网络效率**：专业的构建者可以优化区块构建，从而带来更高效的区块传播 (block propagation) 和处理。
- **减少中心化**：通过解耦这些角色，PBS 可能会减少目前在区块构建和提议中均占主导地位的大型矿池 (mining pools) 或质押服务商 (staking providers) 的影响。

### PBS 以及中继、构建者和验证者之间的关系 (PBS and the Relationship Between Relays, Builders, and Validators)

提议者-构建者分离 (PBS) 还在以太坊网络的不同参与者之间引入了更加错综复杂的关系：

1. **搜索者 (Searchers)**：
   - 搜索者是以太坊协议外的实体。搜索者不直接与区块链交互。相反，它们将构建好的捆绑包 (bundles) 提交给构建者。
   - 搜索者持续扫描公共内存池，以寻找抢跑 (Frontrunning)、夹子 (Sandwich) 或套利 (Arbitrage) 等 MEV 机会。
   - 它们构建捆绑包，这些捆绑包是交易的有序列表，执行某些 MEV 策略，并向构建者进行竞价。
2. **构建者 (Builders)**：
   - 构建者是专业的实体，专注于构建具有最佳交易排序和包含的区块。它们相互竞争，为提议者创建利润最高的区块，同时考虑 Gas 费用、交易优先级和潜在的 MEV（最大可提取价值）。
   - 构建者不直接与区块链交互。相反，它们将构建的区块提交给中继。
   - 这种提交包括区块的数据（交易、执行负载 (execution payload) 等）以及它们愿意为提议其区块而支付的竞价 (bid)。
3. **中继 (Relays)**：
   - 中继从多个构建者那里接收区块，确认它们的有效性，并将具有最高竞价的有效区块提交给托管机构，供验证者签名。
   - 中继充当构建者和提议者之间的中介。它们从构建者那里接收区块并将其转发给提议者。
   - 中继可以执行诸如区块验证和过滤之类的附加功能，以确保只有有效且高质量的区块被发送给提议者。
   - 某些中继可能专门处理特定类型的区块，例如具有高 MEV 潜力的区块。
4. **验证者 (验证者/提议者) (Validators (Proposers))**：
   - 在 PBS 下，验证者承担提议者的角色。它们从中继接收区块，并根据预定义标准（通常是提供最高奖励的区块）选择最好的区块。
   - 一旦提议者选定区块，它们就会将其提议给网络，以进行验证并包含在区块链中。
   - 验证者仍然负责保护网络安全并确保对区块链状态达成共识。

这整个过程如下图所示。有关进一步的解释，请参见 [Flashbots 的文档](https://docs.flashbots.net/)。

![MEV-Boost architecture](https://epf.wiki/images/mev-boost-architecture.png)
*MEV-Boost PBS 参与者之间的通信大纲。来源：[ethresear.ch](https://ethresear.ch/t/mev-boost-merge-ready-flashbots-architecture/11177)*

这种角色分离创造了更加动态和专业化的区块构建过程。构建者可以专注于优化区块构建和提取 MEV，而提议者可以专注于选择最佳区块并维护网络安全。

然而，这种新关系也引入了新的挑战。

### 中继隐忧 (Relay Concerns)

有理由认为中继违背了以下以太坊的核心原则：

- 去中心化：[六个中继](https://www.relayscan.io/overview?t=7d) 处理了 99% 的 MEV-Boost 区块（这几乎占以太坊区块的 90%），这一事实引发了合理的中心化担忧。
- 抗审查性：中继 _可以_ 审查区块，并且由于是中心化的，它们可能会被监管机构强迫这样做。例如，当它们被施压要求审查与 [OFAC 制裁名单](https://home.treasury.gov/news/press-releases/jy0916) 上的地址进行交互的交易时，就发生了这种情况。
- 无信任性：验证者信任中继提供有效的区块头，并在签名后发布完整区块；构建者信任中继不窃取 MEV。尽管背叛任何一方都会被察觉，但即便是一次性的攻击，不诚实也可能是暴利的。

### 第三方依赖 (Third party dependency)

PBS 涉及将区块的构建外包给不直接参与以太坊共识的实体，这一事实可能会因为依赖第三方而带来意外或不受欢迎的后果，例如信任问题、运营依赖以及单点故障 (single points of failure) 的引入。特别是 MEV-Boost 的使用如此广泛，可能被视为一种危险的第三方依赖，因为以太坊如此庞大比例的新区块都是使用 Flashbots 的软件创建的。

最近值得注意的一点是，在 3 月 27 日至 28 日，由于来自 bloXroute 中继区块的 blob 传播缓慢，以太坊经历了漏块（missed slots）的激增。该问题源于 Lighthouse 客户端期望从同一来源获取 blob 和区块，这与 bloXroute 的 BDN（区块链分布式网络 (Blockchain Distributed Network, BDN)）不匹配，后者仅传输区块。这种分歧导致节点忽略了没有随附 blob 的区块，尤其是在 BDN 更新加速了没有 blob 的区块传播之后。将 blob 与这些区块集成的尝试没有成功，通常会导致 202 响应，其中数据已被确认，但由于先前已收到区块而未使用。

该问题的核心被确定在 Lighthouse 的 HTTP API 处理 blob 分发（与 P2P 网络分离）上。这一事件凸显了依赖外部服务（如 bloXroute）进行关键区块链操作的潜在陷阱，强调了在区块链网络中进行细致组件管理对于维持运营完整性和避免漏洞的重要性。更多细节可以在 [这里](https://gist.github.com/benhenryhunter/687299bcfe064674537dc9348d771e83) 找到。

### 安全隐忧 (Security Concerns)

正如在本条目中所看到的，PBS 涉及许多不同的实体参与向链中添加新区块的过程，这不可避免地增加了可以被利用的潜在攻击向量 (attack vectors) 的数量。

与 PBS 相关的脆弱性（如发生故障的中继或托管机构）存在导致丢失区块的风险，但不会危及以太坊的完整性。这些丢失的区块可能会影响用户和验证者。尽管如此，如果外部构建者失败，以太坊客户端可以恢复到常规的区块构建，从而确保网络稳定性。

### 抗审查性被削弱 (Undermined Censorship Resistance)

构建者中心化可能带来的另一个问题是危及以太坊的抗审查性和完整性，因为这些主导构建者在理论上可以合谋或被强迫操纵交易流，或将特定交易排除在区块之外，从而破坏以太坊网络的开放和无许可性质。虽然在目前情况下，即使大多数参与方选择审查，仍然无法阻止提交这些交易，而只会延迟它们的包含。

为了提高抗审查性，目前正在考虑诸如匿名区块提案（这将保护参与者免于因其处理的交易而被单独针对）之类的机制。此外，目前还在探索在区块提案中强制要求承诺包含特定交易的机制，以确保基本交易不会被审查。简要概述可以在 [这里](https://censorship.pics) 找到。

有关 PBS 内部这些反审查措施的详细讨论，请参见 [Vitalik Buterin 的综合分析](https://notes.ethereum.org/@vbuterin/pbs_censorship_resistance)。

## 研究与拟议的解决方案 (Research and Proposed Solutions)

PBS 是以太坊生态系统中活跃的研究领域之一。它提出了几项挑战，包括潜在的安全漏洞和中心化风险。正在进行的研究重点是通过封装式 PBS (ePBS)、包含列表、协议强制提议者承诺 (PEPC) 等创新来解决这些问题。

封装式提议者-构建者分离 (ePBS) 旨在解决与 MEV-Boost 相关的一些局限性和中心化隐忧，目前 MEV-Boost 促成了大约 90% 的以太坊区块的 PBS。

![Evolution of MEV-Boost slot share since The Merge](https://epf.wiki/images/MEV-Boost blocks.png)
*自合并以来通过 MEV-Boost 构建的区块比例演变。来源：[mevboost.pics](https://mevboost.pics/)*

### 封装式提议者-构建者分离 (Enshrined Proposer-Builder Separation, ePBS)

封装式 PBS 涉及将 PBS 机制直接嵌入到以太坊的共识层中，这将有两个主要的潜在优势：

- 减少中心化风险：将 PBS 移入协议层可能会减少对具有中心化倾向的第三方的依赖，这符合以太坊去中心化和抗审查的核心价值。
- 安全性和稳定性：外部依赖和协议外软件（如中继）已经表现出了漏洞（例如“低碳十字军 (Low-Carb Crusader)”攻击）。将 PBS 集成到以太坊协议中可以减轻这些风险，并减少与维护各种组件之间兼容性相关的协调成本。

提到这个 [以太坊研究讨论 (eth research discussion)](https://ethresear.ch/t/why-enshrine-proposer-builder-separation-a-viable-path-to-epbs/15710)，ePBS（特别是通过双区块头锁 (TBHL) 和乐观中继）提供了一条解决当前挑战并增强区块生产和 MEV 提取过程的效率、安全性和去中心化的途径。

有关 ePBS 的更多详细信息，请查看此 [ePBS 维基条目](/wiki/research/PBS/ePBS.md)。

### 协议强制提议者承诺 (Protocol-Enforced Proposer Commitments, PEPC)

PEPC 是针对 PBS 和 MEV-Boost 缺点的另一个拟议解决方案，它倡导比传统 PBS 更开放和灵活的设计。正如在这篇 [Mirror 文章](https://mirror.xyz/ohotties.eth/lBEXiiU7yK91OuSn8QyJPM9Db8GuyDFzCEUAj60BWyI) 中所解释的那样，PEPC 旨在为提议者对任何外包的区块构建任务做出可信承诺提供通用的基础设施，这可以更好地适应以太坊网络不断演变的需求，特别是在预期的数 sharding 扩展和 rollup 采用的情况下。

其目标是允许提议者注册任意承诺（通过 EVM 执行来表达），外部各方可以依赖这些承诺。这样，任何人都可以向提议者提供服务，只要它们满足注册的承诺，从而促进无许可创新。此外，与 MEV-Boost 等外部解决方案不同，PEPC 将把承诺满足集成到核心协议中，只有履行了提议者注册承诺的区块才被视为有效，从而提高了安全性和可信度。

关于 PEPC 旨在提供的对提议者职责的灵活方法，值得注意的是，支持广泛的外包智能合约，从完整的区块构建到特定的有效交易包含以及 rollup 区块的有效性证明。

所有这些也将与现有的协议外机制（如 EigenLayer）互补，通过从乐观模型转变为悲观强制执行模型（违反承诺本质上会使区块失效）来增强提议者承诺的可信度。

有关更详细的解释，请查看 [PEPC](/wiki/research/PBS/ePBS?id=protocol-enforced-proposer-commitments-pepc)。

### EIP-7547：包含列表 (Inclusion Lists)

正如在这篇 [ethereum-magicians.org 帖子](https://ethereum-magicians.org/t/eip-7547-inclusion-lists/17474) 中所解释的，包含列表 (inclusion lists) 旨在提供一种机制，通过允许提议者指定一组必须及时包含的交易（以便随后的区块被视为有效）来提高以太坊的抗审查性。

通过这种强制包含交易的机制，提议者在不牺牲 MEV 奖励的情况下保留了对区块构建的一些控制权。最简单的方法是，提议者指定一个它们自己在内存池中发现的交易列表，如果构建者希望其区块在下一个时隙被提议，就必须包含这些交易。尽管由此产生了一些问题，如激励不兼容和免费数据可用性的暴露，但已经提出并正在开发诸如前向和多重包含列表之类的解决方案来应对这些挑战，展示了以太坊社区致力于改进和推进协议以维护其去中心化、公平和抗审查性核心价值的决心。

### 进一步阅读和资源 (Further Reading and Resources)

以下是有关 PBS 及相关主题的一些进一步阅读材料：

- [关于提议者-构建者分离 (PBS) 的笔记 (Notes on Proposer-Builder Separation (PBS))](https://barnabe.substack.com/p/pbs)
- [时间游戏及对 MEV 提取的影响 (Timing Games and Implications on MEV extraction)](https://chorus.one/articles/timing-games-and-implications-on-mev-extraction)
- [为什么选择 ePBS？ (Why ePBS?)](https://ethresear.ch/t/why-enshrine-proposer-builder-separation-a-viable-path-to-epbs/15710)
- [Vitalik 关于 PBS 审查的看法 (Vitalik on pbs censorship)](https://notes.ethereum.org/@vbuterin/pbs_censorship_resistance)
- [用于 ePBS 的负载时效性委员会 (PTC) 设计 (Payload timeliness committee(PTC) design for ePBS)](https://ethresear.ch/t/payload-timeliness-committee-ptc-an-epbs-design/16054)
- [双时隙 PBS (2-slot PBS)](https://ethresear.ch/t/two-slot-proposer-builder-separation/10980)
- [前向包含列表 (Forward Inclusion Lists)](https://notes.ethereum.org/@fradamt/forward-inclusion-lists)
