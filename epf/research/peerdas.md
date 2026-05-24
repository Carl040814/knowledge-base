# 以太坊 PeerDAS 介绍 (Introduction to Ethereum PeerDAS)

> :warning: 本文是一个[存根 (stub)](https://en.wikipedia.org/wiki/Wikipedia:Stub)，请通过[贡献 (contributing)](/contributing.md)和扩充它来帮助本维基 (wiki)。

> :warning: 本文档涵盖了一个活跃的研究领域。在阅读时它可能已经过时，并且随着设计空间的发展，未来可能会进行更新。

**PeerDAS**（对等方数据可用性采样 (Peer Data Availability Sampling)）是在 [EIP-7594](https://eips.ethereum.org/EIPS/eip-7594) 中引入的一种网络协议 (networking protocol)。它旨在优化以太坊网络中的数据分发 (data distribution) 和验证 (verification)。PeerDAS 确保了主要包含来自 Rollups 等二层网络 (Layer 2, L2) 解决方案数据的 Blob (blobs)，在不对节点造成过重负担的情况下，依然保持可靠的可访问性。

## 以太坊路线图中的 PeerDAS (PeerDAS in the Ethereum Roadmap)

扩展 (Scaling) 对以太坊 (Ethereum) 至关重要。网络必须提高其处理更多交易 (transactions) 和存储数据的能力，从而使交易更便宜、更易于用户使用。随着对区块链应用需求的增长，以太坊需要在不妥协去中心化 (decentralization) 或安全性 (security) 的前提下，支持更高的吞吐量 (throughput)。

PeerDAS 是 [Danksharding](https://ethereum.org/en/roadmap/danksharding/) 的关键组成部分，并在以太坊以 Rollup 为中心的路线图 (rollup-centric roadmap) 中发挥着核心作用。它是 [Surge](https://vitalik.eth.limo/general/2024/10/17/futures2.html) 路线图类别的一部分，该类别专注于扩展交易吞吐量 (transaction throughput) 并降低成本。

二层网络 (Layer 2, L2) 解决方案（例如 Rollups）依赖以太坊来提供数据可用性 (data availability) 和安全性 (security)。最初，Rollups 直接将交易数据作为调用数据 (calldata) 发布到以太坊执行层 (execution layer)，这种方法既昂贵又低效。[EIP-4844 (原始 Danksharding, Proto-Danksharding)](https://eips.ethereum.org/EIPS/eip-4844) 引入了 **Blob (blobs)**——一种全新的数据结构，允许 Rollups 以显著更低的成本发布数据。虽然 EIP-4844 是一项重要的改进，但它只是通往 Danksharding 的垫脚石。在 Danksharding 中，Blob (blobs) 将被转化为**数据列 (data columns)** 并使用 PeerDAS 在网络中分发。这种转变将进一步降低成本并提高可扩展性 (scalability)。

以太坊研究人员为 PeerDAS 规划了一个渐进式的、多阶段的路线图，在提高吞吐量与保持网络稳健性 (network robustness) 之间取得了平衡：

- **阶段 0 (EIP-4844) (Stage 0 (EIP-4844))：** 引入用于 Blob 分发的子网 (subnets)，但不采用数据可用性采样 (Data Availability Sampling, DAS)，要求节点下载所有数据。
- **阶段 1 (Stage 1)：** 通过横向扩展 Blob 并引入列子网 (column subnets) 来实现一维 PeerDAS (1D PeerDAS)，启用对等方采样 (peer sampling) 以提高效率。
- **阶段 2 (Stage 2)：** 通过增加纵向 Blob 扩展和轻量级的基于单元格的对等方采样 (cell-based peer sampling)，实现带二维 PeerDAS (2D PeerDAS) 的完整 Danksharding，支持强健的分布式重建 (distributed reconstruction) 并最大化可扩展性 (scalability)。

## 深入理解 PeerDAS (Understanding PeerDAS)

### 数据分区与托管分配 (Data Partitioning and Custody Assignment)

PeerDAS 将数据 Blob (blobs) 划分为更小的单元，称为*列 (columns)*，这些列作为数据采样的原子组件。网络将节点分配给负责特定列集合的托管组 (custody groups)。一个以节点 ID (node IDs) 等公开可验证输入为参数的确定性函数 (deterministic function) 支配着这种分配，以确保分发的透明和可复现。节点必须维持一个最低的托管阈值 (custody threshold)，以保证基线数据可用性。那些存储额外列的节点将成为**超级节点 (super-nodes)**，持有所有列并增加系统冗余以增强容错性 (fault tolerance)。

### 数据编码与分发 (Data Encoding and Distribution)

数据 Blob (blobs) 使用里德-所罗门纠删码 (Reed-Solomon erasure codes) 进行编码。这一过程将数据划分为多个列并添加校验列 (parity columns)，使得即使部分列丢失，也能够重建 (reconstruct) 原始数据集。一旦编码完成，每一列都将使用 Gossip 协议 (gossip protocol) 在网络中传播。提议者 (Proposers) 最初将列分发给一部分节点子集，这些节点随后将数据转发 (relay) 给它们的对等方 (peers)。节点订阅与其托管组 (custody groups) 对齐的子网 (subnets)，从而优化数据流并减少网络拥堵 (network congestion)。如果节点未能通过 Gossip 接收到某一列，它可以使用与其他节点的请求/响应协议 (request/response protocol) 来检索丢失的数据。

### 数据可用性采样 (Data Availability Sampling, DAS)

PeerDAS 采用数据可用性采样 (Data Availability Sampling, DAS) 来验证数据，而无需进行完整下载。节点从其对等方 (peers) 请求随机的列样本。通过使用概率性方法 (probabilistic methods)，当节点获得足够数量的样本时，便可推断出完整数据集的可用性。如果检测到丢失的列，节点可以发起直接请求以恢复数据。

### 基于 KZG 承诺的密码学验证 (Cryptographic Verification with KZG Commitments)

为了确保数据完整性与真实性 (data integrity and authenticity)，PeerDAS 使用了 KZG (Kate-Zaverucha-Goldberg) 承诺。这些密码学承诺使节点能够验证采样的列是否与原始数据集匹配，而无需下载完整的数据。这种高效的验证过程可以防止篡改和数据损坏。

### 数据重建与冗余管理 (Data Reconstruction and Redundancy Management)

节点持续对其托管下的列进行采样。当节点为一个数据 Blob (blob) 累积了超过 50% 的总列数时，它便可以使用里德-所罗门解码 (Reed-Solomon decoding) 完全重建原始数据集。一旦重建完成，节点将把恢复的列重新分发回网络中。这种重新分发增强了整体数据的可用性，以及对抗子网故障 (subnet failures) 或临时数据不可用 (temporary data unavailability) 的弹性 (resilience)。

### 验证者协议与分叉选择规则 (Validator Protocols and Fork-Choice Rules)

验证者 (Validators) 遵循修改后的分叉选择规则 (fork-choice rules)，以在共识过程 (consensus process) 中强制执行数据可用性。对于新提议的区块 (proposed blocks)，验证者根据通过 Gossip 子网 (gossip subnets) 接收到的列来评估数据可用性。对于较旧的区块，它们依靠对等方采样 (peer sampling) 结果来验证持续的数据可用性。这种双重方法 (dual approach) 减轻了与临时数据扣留 (data withholding) 相关的风险，确保验证者仅对具有可验证数据的区块进行投票，从而维护区块链的完整性与安全性。

总体而言，PeerDAS 整合了确定性托管分配、概率性数据可用性采样、强健的纠删码以及密码学承诺，为去中心化网络中的数据可用性提供了一个可扩展、容错且安全的框架。其设计确保了即使在敌对条件下，数据依然是可验证和可恢复的，使其成为分布式数据管理的一套富有韧性的架构。

## 参考文献 (References)

- [EIP-7594: PeerDAS](https://eips.ethereum.org/EIPS/eip-7594)
- [包括 PeerDAS 的 Fulu 规范 (Fulu specifications including PeerDAS)](https://github.com/ethereum/consensus-specs/tree/dev/specs/fulu)
- [使用 PeerDAS 扩展以太坊 L1 - dapplion 的演示 (Scaling Ethereum L1 with PeerDAS - dapplion, presentation)](https://www.youtube.com/watch?v=_PW6jFTWLPc)
- [Pectra 及以后的 PeerDAS - Francesco D'Amato 的演示 (PeerDAS in Pectra and beyond - Francesco D'Amato, presentation)](https://www.youtube.com/watch?v=WOdpO1tH_Us)
- [Manu Nalepa 的 PeerDAS 手册 (PeerDAS Book by Manu Nalepa)](https://hackmd.io/@manunalepa/peerDAS/https%3A%2F%2Fhackmd.io%2F%40manunalepa%2FB1idHCOfke)
- [以太坊协议未来可能的走向之第二部分：Surge——Vitalik Buterin (Possible futures of the Ethereum protocol, part 2: The Surge by Vitalik Buterin)](https://vitalik.eth.limo/general/2024/10/17/futures2.html)
- [从 4844 到 Danksharding：一条扩展以太坊 DA 的路径 (From 4844 to Danksharding: a path to scaling Ethereum DA (ethresear.ch))](https://ethresear.ch/t/from-4844-to-danksharding-a-path-to-scaling-ethereum-da/18046)
- [PeerDAS——一种使用经过实战检验的 P2P 组件的更简单 DAS 方法 (PeerDAS – a simpler DAS approach using battle-tested p2p components (ethresear.ch))](https://ethresear.ch/t/peerdas-a-simpler-das-approach-using-battle-tested-p2p-components/16541)
- [DevCon Sea：使用 DAS 扩展以太坊——Francesco (DevCon Sea: Scaling Ethereum with DAS by Francesco)](https://www.youtube.com/watch?v=toR2UKzE_zA)
- [DevCon Sea：从 PeerDAS 到 FullDAS (DevCon Sea: From PeerDAS to FullDAS)](https://www.youtube.com/watch?v=Y8VKmyJMAUk&t=9s)
- [EthPrague：PeerDAS——Dapplion (EthPrague: PeerDAS by Dapplion)](https://www.youtube.com/watch?v=fCIPNxGXmmE&t=43s)
- [Pectra 及以后的 PeerDAS——Francesco (PeerDAS in Pectra and beyond by Francesco)](https://www.youtube.com/watch?v=WOdpO1tH_Us&t=334s)
