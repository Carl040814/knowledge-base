# 数据可用性 (Data Availability)

任何一层 (layer-1) 区块链的核心职责都是保证_数据可用性 (data availability)_。

数据可用性 (Data Availability) 是指区块数据（即包含交易树的标头与主体 (headers and bodies)）的可用性。在同步 (synchronization) 过程中，节点从其对等方 (peers) 下载每个区块以进行验证并添加到本地数据库。如果接收到的区块包含任何不正确的交易或违反了任何区块有效性规则，该区块将被拒绝并视为无效。同步过程中的区块验证对于节点去信任化地 (trustlessly) 运行并构建需要历史和最新数据的当前状态 (state) 而言是必不可少的。

以太坊 (Ethereum) 通过要求全节点 (full nodes) 下载每个区块（如果部分数据不可用则拒绝该区块）来解决数据可用性问题。每个节点只需通过 P2P 网络 (p2p network) 从其对等方 (peers) 请求特定的区块，并在本地进行验证。

对于旨在处理海量数据的 Rollups 等可扩展系统 (scalable systems) 而言，数据可用性 (data availability) 成为了一个瓶颈。乐观 Rollups (Optimistic Rollups) 需要交易数据可用，以便其欺诈证明机制 (fraud-proof mechanism) 识别并证明不正确的状态过渡 (state transition)。甚至基于有效性证明系统 (validity-proof-based systems) 也需要数据可用性 (data availability) 来确保活性 (liveness)（例如，为了证明 Rollup 的逃生通道 (escape hatch) 或强制交易包含机制 (forced transaction inclusion mechanism) 中的资产所有权 (asset ownership)）。

因此，Rollups（以及其他链下扩展协议 (off-chain scaling protocols)）的数据可用性问题，在于向基础层 (Layer 1, L1) 证明用于重建二层网络状态 (L2 state) 的数据是可用的，而无需 L1 节点 (L1 nodes) 下载区块并存储数据副本。

## 以太坊的方法（4844 之前）(The Ethereum Approach (pre-4844))

以太坊 (Ethereum) 上的 Rollups 由_排序器 (sequencer)_运行：这是一个全节点 (full node)，负责接收来自用户在二层网络 (L2) 上的交易 (transactions)，处理这些交易（使用 Rollup 的虚拟机 (rollup's virtual machine)），并生成用于更新 Rollup 上合约与账户 (contracts and accounts) 状态的 L2 区块 (L2 blocks)。

为了使以太坊 (Ethereum) 能够监控并强制执行 Rollup 的状态过渡 (state transitions)，一个特殊的“共识”合约会存储状态根 (state root)——这是对 Rollup 规范状态 (canonical state) 的密码学承诺 (cryptographic commitment)（类似于 L1 链上的区块标头），它是在处理完一个新批次 (batch) 中的所有交易后生成的。

每隔一段时间，排序器 (sequencers) 会将交易整理进一个新批次 (batch) 中，并通过调用存储 Rollup 批次的 L1 合约，将压缩后的数据作为调用数据 (calldata) 传递给批次提交函数 (batch submission function)，从而将该批次发布到一层网络 (L1)。这实际上将 L2 数据 (L2 data) 存储在 L1 上以确保其可用性 (availability)。

这对于二层网络 (L2s) 而言是一项巨大的成本开销，为此引入了 _Blob 数据块 (blobs)_ 作为替代方案。诸如消除冗余 (redundancy)、使用位图 (bitmaps)、利用常见函数签名 (common function signatures)、对大数采用科学计数法 (scientific notation) 以及对重复数据 (repetitive data) 利用合约存储等压缩技术，能够减小调用数据 (calldata) 的大小，从而大幅节省 Gas 费用 (gas savings)。

## 数据可用性采样 (Data Availability Sampling)

采样的目的是为节点提供数据可用性的概率性保证 (probabilistic guarantee)。轻节点 (Light nodes) 确实需要连接到其他（诚实的）节点来进行采样。对于所有区块链中的全节点也是如此——它们不进行数据可用性采样，但它们需要连接 to 至少一个其他节点，该节点将通过 P2P 网络 (p2p network) 向它们提供历史数据。

在数据可用性采样 (Data Availability Sampling, DAS) 中，每个节点最终仅下载数据的一小部分，但采样是在_客户端侧 (client-side)_完成的，并且是在每个 Blob (blob) 内部进行采样，而不是在 Blob 之间采样。每个节点（包括未参与质押 (staking) 的客户端节点 (client nodes)）都会检查每个 Blob，但它们不下载整个 Blob，而是私下选择 Blob 中的 N 个随机索引 (random indices)（例如，N = 20），并尝试仅下载这些位置的数据。

该任务的目标是验证每个 Blob 中的数据至少有一半是_可用的 (available)_。

![data-availability-sampling](../img/scaling/das.png)

### 纠删码 (Erasure Coding)

纠删码 (Erasure coding) 允许我们对 Blob (blobs) 进行编码，其方式是只要 Blob 中至少有一半的数据被发布，网络中的任何人都可以重建并重新发布其余的数据；一旦重新发布的数据完成传播，最初拒绝该 Blob 的客户端最终将趋向于接受它。

从高层次来看，纠错码 (erasure correcting code) 的工作原理如下：将一个由 `k` 个信息块构成的向量编码 (encoded) 为一个_更长_的由 `n` 个编码块构成的向量。该编码的编码率 (rate) `R = k/n` 衡量了该编码引入的冗余 (redundancy)。随后，从编码块的某些子集 (subsets) 中，我们就可以解码出原始的信息块。

如果该编码是[_最大距离可分 (maximum distance separable)_](https://www.johndcook.com/blog/2020/03/07/mds-codes/) (MDS) 的，那么原始信息块可以从编码块中任何大小为 `k` 的子集中恢复，这是一个非常实用的效率和鲁棒性保证。该概念基于一个简单的原理：如果你知道一条线上的两个点，你就可以唯一确定整条线。换句话说：一旦你已知多项式在 `t` 个不同位置的求值结果，你就可以获得它在任何其他位置的求值结果（通过先恢复该多项式，然后对其进行求值）。

我们要求区块承诺 (commit) 该“扩展 (extended)”数据的默克尔根 (Merkle root)，并让轻客户端 (light clients) 概率性地检查大部分扩展数据是否可用，在检查中以下三种情况之一将为真：

- 整个扩展数据都是可用的，纠删码构造正确，且区块有效。
- 整个扩展数据都是可用的，纠删码构造正确，但区块无效。
- 整个扩展数据都是可用的，但纠删码构造不正确。

在情况 (1) 下，区块有效，轻客户端 (light client) 可以接受它。在情况 (2) 下，预期其他某些节点将迅速构造并转发一个欺诈证明 (fraud proof)。在情况 (3) 下，同样预期其他某些节点将迅速构造并转发一种专门的欺诈证明 (fraud proof)，以显示该纠删码的构造是不正确的。如果轻客户端在一段时间内没有收到任何欺诈证明，它将把这作为该区块实际上有效的证据。

一个值得研究的有趣实现可以在[此处](https://github.com/ethereum/research/tree/master/erasure_code/ec65536)找到。

其他资源 (Other resources)：

- https://github.com/ethereum/research/wiki/A-note-on-data-availability-and-erasure-coding#what-is-the-data-availability-problem
- https://hackmd.io/@alexbeckett/a-brief-data-availability-and-retrievability-faq
- https://arxiv.org/abs/1809.09044
- https://ethereum.org/en/developers/docs/data-availability/
