# 为实现可扩展性 (Scalability) 对以太坊核心 (Ethereum Core) 进行的修改

随着维塔利克·布特林 (Vitalik Buterin) 于 2020 年发布 [以 Rollup 为中心的路线图 (Rollup-Centric Roadmap)](https://ethereum-magicians.org/t/a-rollup-centric-ethereum-roadmap/4698)，以太坊 (Ethereum) 已迈上了实现可扩展性 (scalability) 的道路。以太坊可扩展性 (scalability) 的终极目标可以总结如下：

- 将执行层 (execution layer)（去中心化应用 (dApps)）完全过渡到二层网络 (Layer 2, L2) Rollups。
- 优化以太坊一层网络 (Layer 1, L1)，使其主要作为结算与数据可用性层 (settlement and data availability layer)。

在[合并 (Merge)](https://github.com/ethereum/consensus-specs/tree/dev/specs/bellatrix)之后（参见[共识规范 (consensus spec)](https://github.com/ethereum/consensus-specs)、[注释规范 (annotated spec)](https://github.com/ethereum/annotated-spec/blob/master/merge/beacon-chain.md) 和 [执行规范 (execution spec)](https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/paris.md)），以太坊链已被划分为两条不同的链：[共识层 (Consensus Layer, CL)](https://github.com/ethereum/consensus-specs) 和 [执行层 (Execution Layer, EL)](https://github.com/ethereum/execution-specs)。共识层 (CL) 负责以太坊的安全性 (security)、去中心化 (decentralization) 和抗审查 (censorship-resistant) 属性，而执行层 (EL) 则负责执行由共识层提议 (proposed) 的每个新区块 (block) 内的交易 (transactions)。这些交易会更新链的全局状态 (global state)，而该状态会被安全地存储在共识层 (CL) 上。

虽然执行层 (EL) 可用于直接的一层网络 (L1) 去中心化应用 (dApp) 交易 (transactions)，但如前所述，其目标是让去中心化应用 (dApp) 交易完全转移到二层网络 (L2) Rollups 上。执行层 (EL) 将主要由 Rollups 用于更新二层网络 (L2) 状态 (state)，或者在二层网络 (L2) 因某些原因[停止工作 (stopping working)](https://docs.arbitrum.io/sequencer#unhappyuncommon-case-sequencer-isnt-doing-its-job) 时用作备份 (backup)。

共识层 (CL) 将被 Rollups 用于数据可用性 (Data Availability, DA) 并存储有效性证明 (proofs of validity)，特别是对于零知识 Rollups (ZK rollups)。为了实现可扩展性 (scalability) 并降低 Gas 成本 (gas costs)，在共识层 (CL) 区块上存储数据必须具有长期可负担性。为了完成这一宏伟路线图，共识层 (CL) 的开发 (development) 阶段可以概述如下：

- 原始 Danksharding (Proto-Danksharding)（[EIP-4844](https://eips.ethereum.org/EIPS/eip-4844)），引入了 Blob (blobs) 的升级（于 2024 年 3 月 14 日上线）。
- 增加 [Blob 数量与 Gas 修改 (blob count & gas modifications)](https://ethresear.ch/t/on-increasing-the-block-gas-limit/18567)（计划于 2024 年底前进行）。
- 引入 [对等方数据可用性采样 (Peer Data Availability Sampling, PeerDAS)](https://ethresear.ch/t/peerdas-a-simpler-das-approach-using-battle-tested-p2p-components/16541)。
- 全面实现 [Danksharding](https://ethresear.ch/t/from-4844-to-danksharding-a-path-to-scaling-ethereum-da/18046)。

在接下来的章节中，我们将讨论 EIP-4844 将如何影响执行层 (EL) 和共识层 (CL)。
