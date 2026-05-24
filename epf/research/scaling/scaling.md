# 扩展 (Scaling)

在计算机系统中，可扩展性 (scalability) 是指系统在增加或扩大的工作负载 (workloads) 下保持良好性能的能力。区块链的主要目的是处理用户的交易 (transactions) 并管理网络的账本状态 (ledger state)。工作负载的增加可能是由于现有用户对交易的需求增加，或者是由于用户数量的增长。

> **区块链系统的可扩展性 (scalability) 可以定义为它处理日益增加的操作量的能力，即在不提高对网络节点运营商 (node operators) 要求的前提下，处理更多的交易 (transactions)**。

## 可扩展性限制 (Scalability Limits)

对于链中包含的每个区块 (block)，必须在一定比例的验证者节点 (validator nodes) 之间就其有效性达成通用共识，而共识机制 (consensus mechanism) 就是达成这一共识的方法。**区块延迟 (block latency)** 是指将一个有效区块包含进链中所需的时间。

以太坊 (Ethereum) 使用基于权益证明 (Proof of Stake, PoS) 的共识协议，称为 [Gasper](https://eips.ethereum.org/assets/eip-2982/arxiv-2003.03052-Combining-GHOST-and-Casper.pdf)，其理想的区块延迟在规范中通过常量 `SECONDS_PER_SLOT`（12 秒）固定，然而实际的区块延迟可能会略有不同，因为特定验证者可能会遗漏区块，导致其无法包含在特定的时隙 (slot) 中。

另一个基本的区块链设计参数是**区块大小 (block size)**，即单个区块中可存储的数据量上限。以太坊有一个名为 `gas_limit` 的配置参数，用于限制处理区块所需的计算开销 (computational effort)，这实际上也限制了区块的大小。以太坊中发生的每项操作都需要一定数量的“Gas”才能完成。一个区块内所有操作的 Gas 总额不能超过以太坊的 `gas_limit`。这种方法确保了网络的处理能力不会过度扩展。[EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) 定义了以太坊 Gas 定价机制，允许动态调整区块大小 (dynamically adjust block size) 以应对瞬时网络拥堵 (transient network congestions)（同时永远不超过 3000 万 Gas 单位的上限），并激励网络将 Gas 目标值定位在 `gas_target`（1500 万）所定义的数额。Gas 限制约束了可包含在以太坊区块中的交易数量（及其计算复杂性 (computational complexity)）。

区块延迟 (block latency) 和区块大小 (block size) 直接决定了区块链的交易吞吐量，这通过 **TPS**（每秒交易数 (transactions per second)）指标来衡量。区块延迟可能受到多种因素的影响，例如网络节点的计算能力、共识机制 (consensus mechanism) 的复杂性、网络流量拥堵和区块大小。因此，一个由少数相互紧密连接的高性能节点组成并使用大区块大小的网络，可以产生具有高 TPS 和极低区块延迟的出色区块链。然而，这样的网络很可能是高度中心化 (centralized) 的，并需要用户做出显著的信任假设 (trust assumptions)。

去中心化 (decentralization) 的另一个障碍是区块链状态 (blockchain state) 的大小。**数据在需要时可被访问的特性被称为数据可用性 (data availability)**。必须保证状态数据的可用性，包括提供足够的数据，以允许网络中的任何新节点在没有任何信任假设的情况下重建最新版本的区块链。为了保证数据可用性 (data availability) 而必须保留所有数据，可能会导致节点面临极高的存储要求，并且新节点的同步过程 (synchronization process) 会异常漫长，这会阻碍去中心化。处理大量数据在网络拥堵 (network congestion) 方面也是一个问题，并且会要求节点运营商 (node operators) 具有极高的连接带宽 (connection bandwidth)。以太坊通过让读取或写入状态的操作变得极其昂贵，来遏制无节制的状态增长 (state growth)。

注意区块链的设计和调优（共识机制、节点要求、状态结构和大小……）与其去中心化 (decentralization) 能力之间的紧密关系。运行节点的高硬件或网络带宽要求会导致苛刻且昂贵的条件，使得运行节点变得十分困难，从而直接影响区块链拥有的节点数量。**为了确保去中心化 (decentralization)，必须让每个人都能够负担得起且可行地运行节点，并激励用户去这样做**。

作为博弈论机制 (game-theory mechanism) 的一部分，为了维护网络的可持续性 (network sustainability)，用户必须支付费用以使其交易包含在内。在以太坊中，费用价格由协议的基础费用 (base fee)、特定交易所消耗的 Gas 量以及一个优先级市场 (priority market) 决定，支付更高优先费用 (priority fees) 的用户将被优先考虑。由于 TPS 有限，在像以太坊这样高度去中心化 (decentralization) 的区块链中包含交易已成为一项有价值的服务。在有限且固定的 TPS 下，交易需求的增加会将费用推高至网络可用性门槛，从而需要牺牲去中心化和/或安全性以实现扩展。

> **回顾可扩展性 (scalability) 的定义，传统的区块链系统在不牺牲一定程度的去中心化 (decentralization) 的情况下，是无法实现大规模采用的可扩展性的。**

这个问题被广泛称为**区块链三难困境 (Blockchain Trilemma)**，它指出区块链网络必须牺牲安全性 (security)、去中心化 (decentralization) 或可扩展性 (scalability) 中的某一项，而且同时最大化这三者是极其困难的。区块链技术的终极圣杯是创建一个既安全又去中心化的交易网络，且该网络能够实现极高的 TPS 率。

!["Blockchain trilemma - Bankless.com"](../img/scaling/blockchain-trilemma.png "Blockchain trilemma - Bankless.com")

## 区块链模块化 (Blockchain Modularity)

现代区块链设计提出了“分而治之”的方法，将系统划分为不同的特定功能组件，以独立最大化三难困境的每个顶点。该方法将系统各个部分的复杂性进行封装，并通过为系统组件定义更简单的交互接口来降低系统性复杂性。
区块链的功能组件：

- 共识 (Consensus)：节点之间在区块构建和交易排序方面的达成一致的机制。
- 执行 (Execution)：定义状态演变的交易执行机制。
- 数据可用性 (Data availability)：确保在需要时可以查询到可用数据的机制。
- 结算 (Settlement)：保证交易最终性与不可篡改性 (finality and immutability) 的机制。

模块化区块链 (modular blockchain) 方法建议将区块链系统划分为不同的逻辑层 (logical layers)。这带来了一种系统复杂性降低的设计，以及更简单的组件，这些组件可以更容易地进行优化和重新设计，以引入新的扩展解决方案。

## 扩展以太坊 (Scaling Ethereum)

以太坊可扩展性 (scalability) 工作的首要目标是提高 TPS 指标，同时不妥协去中心化 (decentralization) 或安全性 (security)。

一层网络扩展 (Layer 1 scaling) 是指所有能提高底层区块链协议 (underlying blockchain protocol) 本身 TPS 的技术。一种天真的扩展解决方案是使用具有“更大区块 (bigger blocks)”的区块链，这可以达到更高的 TPS 数值。然而，这种方法的问题在于，增加在给定时间段内需要被验证的交易数量，可能会导致这样一种场景：只有数量有限 of 节点运营商 (node operators)——那些拥有足够强大硬件的节点——能够参与网络。这反过来又会导致中心化程度的提高。因此，每个区块容量 of 增加必须伴随着对协议的各种修改，以使去中心化得以保持。一层网络扩展 (layer 1 scaling) 解决方案的示例包括分片 (sharding)、权益证明 (Proof of Stake) 共识机制以及旨在优化区块处理效率的协议升级。

!["Layer 1 Scaling"](../img/scaling/layer-1-scaling.png "Layer 1 Scaling")

> **增加 Gas 限制 (gas limits) 将有效地扩大以太坊可以容纳的计算量和数据量，但也会提高对网络节点运营商 (node operators) 的要求。另一个直接影响区块大小 (block size) 的设计选择是操作的 Gas 定价。更便宜的操作将允许在相同的 Gas 限制边界内包含更多数量和更复杂的交易。**

二层网络扩展 (Layer 2 scaling) 是指构建在一层网络 (Layer 1) 区块链协议之上的一系列解决方案，它们可以在不增加一层网络资源消耗 (resource consumption) 的情况下实现更高的 TPS，同时仍然依赖于一层网络的安全模型 (security model)。这些解决方案通常在链下 (off-chain) 处理交易，或者通过使用比主区块链更快且更具可扩展性的替代共识机制来处理交易。二层网络扩展解决方案的示例包括状态通道 (State Channels)、Plasma 链 (Plasma Chains) 或 Rollups。

!["Layer 2 Scaling"](../img/scaling/layer-2-scaling.png "Layer 2 Scaling")

以太坊社区为解决可扩展性 (scalability) 问题而普遍采用的解决方案，是以多 Rollup 为中心的途径 (multi-rollup-centric approach)。在此途径中，以太坊充当结算 (Settlement) 和数据可用性 (Data Availability) 的基本安全层，而大部分执行 (Execution) 任务则被委托给称为 Rollups 的上层。Rollups 将多个二层网络 (Layer 2) 交易打包成单个一层网络 (Layer 1) 交易。这些交易被存储在一层网络 (Layer 1) 中，但它们的执行则被委托给链下 (off-chain) 机制。这允许在不牺牲安全性的情况下实现显著的可扩展性改进。以太坊的高度可编程性 (programmability) 使得通过以太坊智能合约 (smart contracts) 在平台之上创建可扩展解决方案成为可能。如前所述，为了保持高度的去中心化 (decentralization)，以太坊通过 Gas 限制来对区块中包含的交易可以使用的计算和存储资源施加限制。因此，在一层网络 (Layer 1) 中存储的数据被最小化，仅存储原始交易，以及由于执行这些交易而产生的二层网络状态 (Layer 2 state) 的哈希承诺 (hash commitment)。

> **Rollups 可以被看作是一种压缩交易执行的方法，从而减轻了一层网络 (Layer 1) 区块链上的计算负担。这提供了在仍然保持一层网络 (Layer 1) 对二层网络交易的安全水平的前提下，提高 TPS 的可能性，因为它们的数据和二层网络状态过渡 (state transition) 承诺都存储在一层网络 (Layer 1) 中。**

## 迈向以 Rollup 为中心的路线图的以太坊核心修改 (Ethereum Core Changes Towards a Rollup-Centric Roadmap)

参见[下一章节 (Next Section)](/wiki/research/scaling/core-changes/core-changes.md)。

## 以太坊二层网络扩展 (Ethereum Layer 2 scaling)

鉴于目前以 Rollup 为中心的路线图，涌现出了多个二层网络 (Layer 2)。根据它们的安全模型，这些网络可以归纳为两大主要类别：乐观 Rollups (optimistic rollups) 和 zk-Rollups (zk-rollups)。

这两种解决方案都有各自的优势、缺点和独特的技术栈。然而，它们共享几个核心组件：
  - 链上合约 (Onchain contracts)：这些合约控制各种 Rollup，并可能包括跟踪用户存款、监控状态更新等的智能合约。
  - 链下虚拟机 (Offchain virtual machines, VMs)：通常是一个执行交易的经过修改且 EVM 兼容 (EVM-compatible) 的链。

以太坊既充当数据可用性 (DA) 层，又充当结算层，这意味着二层网络 (Layer 2s) 继承了一层网络 (Layer 1) 的安全性。一旦 Rollup 交易被提交到以太坊的基础层，它就无法被回滚。  
历史上，Rollups 将其交易数据发布在以太坊交易的 `calldata` 部分，但这导致了历史节点存储的过度增长。EIP-4844（原始 Danksharding (proto-danksharding)）引入了一种新机制，允许 Rollups 在 Blob (blobs)（一个独立的、更高效的数据空间）中提交数据，并为 Blob 的使用配备了独立的 Gas模型 (gas model)，以及共识节点处理这些数据的方式的微小改变。您可以在[以下章节](https://epf.wiki/#/wiki/research/scaling/core-changes/eip-4844)中了解更多关于 Blob (blobs)、Danksharding 以及 EIP-4844 的信息。

### 乐观 Rollups (Optimistic Rollups)

乐观 Rollups (Optimistic rollups) 的出现是为了提高交易吞吐量，同时仍然依靠加密经济激励来继承以太坊的安全性。
它们的验证模型基于一种称为乐观验证 (optimistic verification) 的技术——提交到一层网络 (Layer 1) 的每笔交易默认被假定为有效。
一个称为运营者或聚合器 (operator or aggregator) 的实体在称为锚定 (anchoring) 的过程中，将一批 L2 交易提交到一层网络 (Layer 1)。此后，会有一个挑战期 (challenge period)，在此期间，任何人都可以通过提交欺诈证明 (fraud proof) 来对交易批次的有效性提出异议。
如果挑战成功，挑战者将获得奖励，而运营者将通过称为罚没 (slashing) 的过程受到惩罚。该机制激励运营者仅提交不会被挑战的交易。

### 零知识 Rollups (ZK Rollups)

零知识 Rollups (ZK rollups) 是使用先进密码学技术以数学确定性提高以太坊吞吐量的扩展解决方案。
二层网络 (Layer 2) 上的交易被打包成批次，并生成一个零知识证明来验证其正确性。然后，该证明被提交到一层网络 (Layer 1) 并在此进行验证。一旦通过验证，所有批次中的交易在数学上都被保证是有效的。
虽然因为使用零知识证明而被称为 ZK rollups，但其主要优势实际上是简洁性 (succinctness)——能够生成一个比所有交易的实际大小要小得多的紧凑证明。
在实践中主要使用两种类型的零知识证明系统：zk-SNARKs (零知识简洁非交互式知识论证 (Zero-Knowledge Succinct Non-Interactive Argument of Knowledge)) 和 zk-STARKs (零知识可扩展透明知识论证 (Zero-Knowledge Scalable Transparent Argument of Knowledge))。  
zk-SNARKs 目前在 ZK rollup 领域有着更广泛的采用，但 zk-STARKs 由于其可扩展性 (scalability) 和无需可信设置 (trusted setup) 而正在获得强劲的发展势头。

## 资源 (Resources)：

- [以太坊共识机制 (Ethereum Consensus Mechanism)](https://ethereum.org/developers/docs/consensus-mechanisms), [已存档](https://web.archive.org/web/20240214225609/https://ethereum.org/developers/docs/consensus-mechanisms)
- [以太坊权益证明共识规范 (Ethereum Proof-of-Stake Consensus Specifications)](https://github.com/ethereum/consensus-specs/tree/dev?tab=readme-ov-file#ethereum-proof-of-stake-consensus-specifications), [已存档](https://web.archive.org/web/20240208050731/https://github.com/ethereum/consensus-specs/tree/dev)
- [以太坊权益证明共识规范 (Ethereum Proof-of-Stake Consensus Specifications)](https://ethereum.github.io/consensus-specs/), [已存档](https://web.archive.org/web/20240217155014/https://ethereum.github.io/consensus-specs/)
- [Vitalik：以太坊可扩展性的极限 (Vitalik The Limits to Blockchain Scalability)](https://vitalik.eth.limo/general/2021/05/23/scaling.html), [已存档](https://web.archive.org/web/20240205202358/https://vitalik.eth.limo/general/2021/05/23/scaling.html)
- [以太坊扩展 (Ethereum Scaling)](https://ethereum.org/en/developers/docs/scaling), [已存档](https://web.archive.org/web/20240209083702/https://ethereum.org/en/developers/docs/scaling)
- [Rollups 不全指南 (An Incomplete Guide to Rollups)](https://vitalik.eth.limo/general/2021/01/05/rollup.html), [已存档](https://web.archive.org/web/20240212014637/https://vitalik.eth.limo/general/2021/01/05/rollup.html)
- [Gemini Cryptopedia 区块链三难困境：快速、安全和可扩展网络 (Gemini Cryptopedia The Blockchain Trilemma: Fast, Secure, and Scalable Networks)](https://www.gemini.com/cryptopedia/blockchain-trilemma-decentralization-scalability-definition), [已存档](https://web.archive.org/web/20240209073156/https://www.gemini.com/cryptopedia/blockchain-trilemma-decentralization-scalability-definition)
- [结合 GHOST 和 Casper (Combining GHOST and Casper)](https://eips.ethereum.org/assets/eip-2982/arxiv-2003.03052-Combining-GHOST-and-Casper.pdf), [已存档](https://web.archive.org/web/20230907004049/https://eips.ethereum.org/assets/eip-2982/arxiv-2003.03052-Combining-GHOST-and-Casper.pdf)
- [以太坊区块 (Ethereum Blocks)](https://ethereum.org/developers/docs/blocks), [已存档](https://web.archive.org/web/20240214052915/https://ethereum.org/developers/docs/blocks)
- [论区块大小、Gas 限制和可扩展性 (On Block Sizes, Gas Limits and Scalability)](https://ethresear.ch/t/on-block-sizes-gas-limits-and-scalability/18444), [已存档](https://web.archive.org/web/20240220230246/https://web.archive.org/web/20240220230246/https://ethresear.ch/t/on-block-sizes-gas-limits-and-scalability/18444)
- [L2beat：所有 L2 Rollup 的概述 (L2beat: an overview of all L2 rollups)](https://l2beat.com/scaling/summary)
