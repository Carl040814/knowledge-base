# 以太坊协议路线图 (Ethereum Protocol Roadmap)

以太坊的发展哲学是开放地拥抱协议演变，并为了获得值得改变的收益而承担一定的风险规避。随着我们对以太坊的认识和经验的增长，研究人员和开发人员正在构思如何应对网络面临的挑战和限制。在核心协议存在的许多年里，已经发生了[许多变化](/wiki/protocol/history.md)。这些变化中的大多数是我们可以称之为路线图的一些共同目标的一部分。

尽管没有官方的路线图，也没有可以主导它的权威机构，但社区内广泛的讨论正在引导协议向特定方向发展。通过就某些目标达成一致，并对当前的发展状态达成共识，社区、开发和研究团队共同努力，在这一抽象路线图中向前推进。

## 无穷花园 (The Infinite Garden)

> *以太坊并不是一场有清晰终点线的零和游戏，而是一场我们希望能够持续进行下去的游戏。要让这成为现实，无穷花园 (Infinite Garden) 需要定期升级其安全性、可扩展性或可持续性，直到达到固化 (ossification)。在那之后，可能只会有一些修剪 —— 这里修剪一下，那里修剪一下。*

## 核心研发 (Core R&D)

关于核心协议的所有讨论、资源以及所有研究和开发都是完全开放、免费和公开的。任何人都可以学习它（正如您可能在此 wiki 中所做的那样），而且任何人都可以参与。没有哪一组特定的个人能够强行推行核心协议的更改，以太坊社区可以通过发声来帮助引导讨论。要了解更多塑造协议的核心研发信息，请阅读[关于它的 wiki 页面](/wiki/dev/core-development.md)。

## 路线图概览 (Roadmap overview)

虽然以太坊开发没有遵循单一的路线图，但我们可以跟踪当前的研发努力，以勾画出正在发生以及未来可能发生的变化。
Vitalik 绘制的这张图表（2023 年 12 月）是映射当前核心研发中许多领域的热门概览：

![V.B. 于 2023 年 12 月更新的以太坊路线图](../research/img/full_roadmap2024_1600x1596.webp)

在这一概览中，不同的领域与相关的类别耦合，形成了各种“升级方向 (urges)”。其中许多方框在我们的 wiki 上都有自己的页面，您可以进行更深入的学习。

### 合并 (The Merge)

与从工作量证明 (Proof-of-Work, PoW) 切换到权益证明 (Proof-of-Stake, PoS) 相关的升级。合并已于 UTC 时间 2022 年 9 月 15 日星期四 06:42:42 成功实现，减少了网络年化电力消耗的 99.988% 以上。然而，这一类别还追踪随后的升级，这些升级可以用来改进共识机制并抚平合并后我们遇到的问题。

**已实现 (IMPLEMENTED)**
| 升级 (Upgrade) | 描述 (Description) | 效果 (Effect) | 最新进展 (State of the art) |
|:------------------------------------ |:---------------------------------------------------------------------------------------------------:|:-------------------------------------------------------------------------------------------------------------------------------------------------:|:---------------------------------------------------- |
| 启动信标链 (Launch the Beacon Chain) | 以太坊转向权益证明 (Proof-of-Stake, PoS) 共识机制的关键一步 | 信标链作为独立网络启动并连接到以太坊，引导验证者 (bootstrap validators) 以为合并 (Merge) 做准备。 | 已发布 (shipped) </br> EIP-2982<span markdown='1'>[^1]</span> |
| 合并执行层与共识层 (Merge Execution and Consensus Layers) | 以太坊的执行层 (execution layer) 与信标链（共识层 (consensus layer)）合并 | 工作量证明 (Proof-of-Work, PoW) 活动停止，网络共识机制转向权益证明 (Proof-of-Stake, PoS)。验证者承担处理所有交易有效性并提议区块的角色和职责 | 已发布 (shipped) |
| 启用取款 (Enable Withdrawals) | 以太坊向权益证明 (Proof-of-Stake, PoS) 共识机制过渡的最后一步（共三步） | 验证者可以添加其取款凭证 (withdrawal credentials)，信标链自动扫描并提取所有非活跃的 ETH | 已发布 (shipped) </br>EIP-4895[^2] |

**待办 (TODO)** 
| 升级 (Upgrade) | 描述 (Description) | 预期效果 (Expected effect) | 最新进展 (State of the art) |
| :----------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :---------------------------------------------------------------------------------------------------------------- |
| 单时隙最终性 (Single slot finality, SSF) | 区块可以在同一个时隙 (slot) 中被提议并最终化 (finalized) | (i) 对应应用更便捷（交易最终化时间提升了一个数量级，即 12 秒而非 12 分钟，意味着对所有用户都有更好的用户体验 (UX)。在[完全 rollup 扩容](#the-surge)下，随着实时 SNARK 证明的实现，单时隙最终性也将意味着 L2 的跨链桥接更快），(ii) 攻击难度显著增加（可以消除多区块 MEV 重组 (re-orgs)，并降低共识机制的复杂度） | 研究中 (in research) </br>(i) VB 的 SSF 笔记[^3] </br>(ii) SSF 后每时隙 8192 个签名[^4] </br>(iii) 简单的 SSF 协议[^5] |
| 单一秘密领导者选举 (Single Secret Leader Election, SSLE) | 允许当选的区块提议者在区块发布前保持私密，以防止 DoS 攻击 | 只有选定的验证者自己知道其已被选定提议区块。 | 研究中 (in research) </br>EIP-7441[^6] |
| 启用更多验证者 (Enable more Validators) | 高效协调不断增加的验证者数量，以在尽可能最好的权衡下实现 SSF 的技术挑战 | 更高的冗余度、更广泛的提议者范围、更广泛的见证者阵列以及整体提高的韧性 | 研究中 (in research) </br> (i) EIP-7514[^7] </br>(ii) EIP-7251[^8] </br> (iii) 8192 个签名[^5] |
| 量子安全签名 (Quantum-safe signatures) | 前瞻性研究和整合抗量子密码算法 (quantum-resistant cryptographic algorithms) | 量子安全且便于聚合的签名将增强协议抵御量子攻击的安全性 | 研究中 (in research) </br> (i) 基于格 (lattice-based)[^9] </br>(ii) 基于 STARK [^10] 的系统 |

### 激增 (The Surge)
通过 Rollups 和数据分片 (Data Sharding) 提高可扩展性的相关升级。

**已实现 (IMPLEMENTED)**
| 升级 (Upgrade) | 轨道 (Track) | 主题 (Topic) | 描述 (Description) | 效果 (Effect) | 最新进展 (State of the art) |
| :--- | :---: | :---: | :--- | :---: | :--- |
| Proto-danksharding | - | 基础 rollup 扩容 | 我们可以停止在以太坊上永久存储 Rollup 数据，并将其移动到临时“blob”存储中，一旦不再需要，该存储就会从以太坊中删除 | 降低交易成本 | 已发布 (shipped) </br>EIP-4844[^11] |

**待办 (TODO)** 
| 升级 (Upgrade) | 轨道 (Track) | 主题 (Topic) | 描述 (Description) | 预期效果 (Expected effect) | 最新进展 (State of the art) |
| :--- | :---: | :---: | :--- | :--- | :--- |
| Danksharding | - | 完全 rollup 扩容 | Danksharding 是从 Proto-Danksharding 开始的 Rollup 扩容的最终完全实现 | 在以太坊上为 Rollup 提供海量空间以倾倒其压缩交易数据 | 研究中 (in research) </br> |
| 数据可用性采样 (Data Availability Sampling, DAS) | - | 完全 rollup 扩容 | 数据可用性采样是网络在不给任何单个节点施加太大压力的情况下检查数据可用性的一种方式 | (i) 确保 Rollup 运营商在 EIP-4844 之后使交易数据可用 (ii) 确保区块生产者使其所有数据可用以保护轻客户端 (light clients) (iii) 在提议者-构建者分离下，只有区块构建者被要求处理整个区块，其他验证者将使用数据可用性采样进行验证 | 研究中 (in research) </br> EIP-7594[^12] |
| 移除 Rollup 辅助轮 (Removing Rollup Training Wheels) | - | 基础与完全 rollup 扩容 | (i) Optimistic Rollup 欺诈/故障证明者 (Optimistic Rollup Fault Provers) </br> (ii) ZK-EVM </br> (iii) Rollup 互操作性 (Rollup interoperability) | (i) Optimistic Rollup 拥有实时证明系统将解决 L2 的审查风险 </br>(ii) 通过 ZK-EVM（支持零知识证明计算的 EVM 兼容虚拟机），在不牺牲链的安全性与去中心化特性的情况下，实现以太坊扩容和隐私保护的巨大提升 </br> (iii) L1 排序器 (L1 Sequencers)，或被赋予特定 Rollup 排序权的以太坊 L1 提议者，将带来更好的可信中立性 (credible-neutrality) 和安全性，并提供 Rollup L1 兼容性 | 研究中 (in research) </br> (i) Arbitrum BoLD[^13] </br> Optimism Cannon[^14] </br> (ii) ZK-EVM [^15] [^16] [^17] </br> (iii) [ET](/wiki/research/PBS/ET.md), </br> [带预确认的 Based 排序 (Based Sequencing with Preconfirmations)](/wiki/research/Preconfirmations/BasedSequencingPreconfs.md) |
| 量子安全且免于可信设置的承诺 (Quantum-safe and Trusted-Setup-Free Commitments) | - | - | 将 KZG 承诺替换为不需要可信设置且量子安全的承诺 | 量子安全承诺 (Quantum-safe Commitments) | 研究中 (in research) </br> |

### 洗劫 (The Scourge)
与抗审查性、去中心化以及缓解由于 MEV 和流动性质押/资金池造成的协议风险相关的升级。

**已实现 (IMPLEMENTED)**
| 升级 (Upgrade) | 轨道 (Track) | 主题 (Topic) | 描述 (Description) | 效果 (Effect) | 最新进展 (State of the art) |
| :--- | :---: | :---: | :--- | :--- | :--- |
| MEV-Boost | MEV 轨道 | 终局区块生产流水线 (Endgame Block Production Pipeline) | 协议外 MEV 市场 | 以太坊社区通过协议外市场成功将 MEV（部分）商品化。现在大部分 MEV 都流向了验证者。 | [已发布 (shipped)](/wiki/research/PBS/mev-boost.md) </br> |
| 提高最大有效余额 (Increase MAX_EFFECTIVE_BALANCE) | 质押经济学 | 提高验证者上限 | 将以太坊验证者的最大有效余额从 32 ETH 提高到 2048 ETH | 合并验证者，减轻网络负载，并简化大型质押者的操作 | 已发布 (shipped) </br> EIP-7251[^27] |

**待办 (TODO)** 
| 升级 (Upgrade) | 轨道 (Track) | 主题 (Topic) | 描述 (Description) | 预期效果 (Expected effect) | 最新进展 (State of the art) |
| :--- | :---: | :---: | :--- | :--- | :--- |
| 协议内嵌提议者-构建者分离 (ePBS) | MEV 轨道 | 终局区块生产流水线 | 出于反审查和缓解 MEV 风险的原因，在协议层内嵌提议者与构建者分离 (Proposer-Builder Separation, PBS) | (i) 创造在协议层阻止交易审查的机会 </br> (ii) 防止业余验证者被能够更好地优化区块构建盈利能力的机构玩家击败 </br> (iii) 通过启用 Danksharding 升级来帮助以太坊扩容 | [研究中 (in research)](/wiki/research/PBS/ePBS.md)[^18] </br> |
| MEV 销毁 (MEV - Burn) | MEV 轨道 | 终局区块生产流水线 | 一个简单的协议内嵌 PBS 附加组件，用于平滑和重新分配 MEV 峰值 | 提取的 ETH 将被销毁，从而使所有 ETH 持有者受益，而不仅仅是运行验证者的人。 | [研究中 (in research)](/wiki/research/PBS/ePBS.md#mev-burn)[^19] |
| 执行票 (ET) | MEV 轨道 | 终局区块生产流水线 | 一个无许可市场，允许买家购买提议执行有效载荷 (execution payloads) 的权利。 | 见证者-提议者分离：信标提议者不关心执行提议者。执行提议者从无许可的执行票 (execution tickets) 市场中选出，并有权将执行构建权转移给第三方。 </br>由于 ET 市场将是一个协议组件，协议将能够自省谁进入市场以及他们愿意支付多少 | [ET](/wiki/research/PBS/ET.md), </br>APS 销毁 (APS-Burn)[^20] |
| 纳入列表 (IL) | MEV 轨道 | 终局区块生产流水线 | 纳入列表 (Inclusion lists) —— 以太坊最去中心化的群体通过在链的构建中输入其偏好来对抗审查的一种方式 | 防止区块构建者审查区块。允许提议者通过提供强制纳入交易的机制来保留部分权威，避免当前的局面，即在没有任何强制交易纳入机制的情况下，提议者面临两个选择：要么对被纳入的交易说“不”，要么在本地构建区块（对交易拥有最终决定权）并牺牲部分 MEV 奖励 | [研究中 (in research)](/wiki/research/inclusion-lists.md)[^21] </br> 多元性组件 (Multiplicity gadgets) [^22] </br> COMIS [^23] |
| 分布式区块构建 (Distributed Block Building) | MEV 轨道 | 终局区块生产流水线 | 通过分布式方式去中心化区块构建过程 | 去中心化构建者 (Builder) 的不同部分： </br> (i) 选择交易的算法（区块构建交易排序） </br> (ii) 用于区块构建的资源，特别是在完全 Danksharding 下（拆分大区块） </br> (iii) 添加额外的构建者服务（例如预确认 (Preconfirmations)） | 研究中 (in research) </br> [预确认 (Preconfirmations)](/wiki/research/Preconfirmations/Preconfirmations.md),</br> SUAVE[^24] |
| 应用层 MEV 最小化 (Application Layer MEV Minimization) | MEV 轨道 | - | 应用层为减少有害 MEV 所做的努力 | 最小化技术针对： </br>(i) 抢跑 (frontrunning)，和 </br>(ii) 夹心攻击 (sandwich attacks) | 示例[^25] |
| 预确认 (Preconfirmations) | MEV 轨道 | - | 用户对交易执行的预确认，以在以太坊交互中获得具有竞争力的用户体验 (UX) | 区块构建者可以公开同意将包含带有超过一定数额优先费的交易，并向用户发送收据，表明他们有意将该交易包含在特定区块中 | [研究中 (in research)](/wiki/research/Preconfirmations/Preconfirmations.md)[^26] |
| 更便宜的节点 (Cheaper Nodes) | 质押经济学 | 改善节点运营商易用性 | 使用沃克尔树 (Verkle trees) 和 SNARK 降低运行节点的成本并提高其易用性。 | 降低 SSD 要求，加快同步时间，并降低独行质押者 (solo stakers) 的准入门槛。 | 研究/提案： [在 eps 节点研讨会中](/docs/eps/nodes_workshop.md)[^28] |
| 限制验证者集合 / Orbit SSF (Capping Validator Set / Orbit SSF) | 质押经济学 | 验证者管理 | 实施 Orbit SSF 以在实现单时隙最终性 (SSF) 的同时高效管理验证者集合大小。 | 防止验证者过度参与导致网络变慢；为独行质押者维持去中心化。 | 研究/提案： [研究中](/wiki/research/eODS.md)[^29] |
| 对抗 LST 中心化 (Combat LST Centralization) | 质押经济学 | 流动性质押 | 探索诸如彩虹质押 (rainbow staking) 或双层质押 (two-tiered staking) 等解决方案，以减少主导 LST 提供商的影响。 | 防止大型流动性质押池获得“造王者”地位或对网络进行系统性控制。 | 研究/提案： [^30], [^31], [^32], [^33],[^34] |

### 边缘 (The Verge)
与更容易验证区块相关的升级。目标是达到一种任何人都可以以极少的资源（例如，在手机或智能手表上）验证以太坊链的状态，通过使验证成本独立于状态大小来实现。这主要是通过无状态 (statelessness) 和简洁证明 (succinct proofs) 来实现的。

**待办 (TODO)**
| 升级 (Upgrade) | 轨道 (Track) | 主题 (Topic) | 描述 (Description) | 预期效果 (Expected effect) | 最新进展 (State of the art) |
| :--- | :---: | :---: | :--- | :--- | :--- |
| 沃克尔树 (Verkle Trees) | 无状态 | 状态承诺 | 用沃克尔树 (Verkle trees) 替换默克尔帕特里夏树 (Merkle Patricia Trees)，以启用更小的见证数据 (witnesses)。 | 启用无状态客户端 (stateless clients)；节点可以在本地不需要完整状态的情况下验证区块。 | [研究中 (in research)](https://verkle.info/)[^35] |
| 数据可用性采样 (Data Availability Sampling, DAS) | 完全 Rollup | Blob 验证 | 为无需完整下载的轻客户端 (light clients) 提供概率性 blob 采样。 | 以极低开销确保 L2 数据可用性和轻客户端安全。 | [研究中 (in research) / EIP-7594](https://eips.ethereum.org/EIPS/eip-7594) |
| SNARK 化的信标链 (SNARKed Beacon Chain) | 简洁性 | 共识验证 | 使用 ZK-SNARK 证明信标链状态转换的有效性。 | 轻客户端可以以极低的资源获得全节点级别的安全保障。 | [研究中 (in research)](https://ethresear.ch/t/snarking-the-beacon-chain-for-light-clients/14115)[^36] |
| SNARK 化的 L1 EVM (SNARKed L1 EVM) | 简洁性 | 执行验证 | 为整个 L1 执行层创建 SNARK 证明。 | 显著降低运行验证节点的成本。 | 研究中 (in research) |

### 清除 (The Purge)
通过修剪历史和非活跃状态、简化协议以及消除技术债务来应对协议和数据膨胀，从而保持网络长期高效。

**待办 (TODO)**
| 升级 (Upgrade) | 主题 (Topic) | 描述 (Description) | 预期效果 (Expected effect) | 最新进展 (State of the art) |
| :--- | :---: | :--- | :--- | :--- |
| 历史数据过期 (History Expiry, EIP-4444) | 存储 | 节点在 p2p 层停止存储超过一年的执行历史。 | 显著降低运行节点的磁盘空间要求（从太字节到吉字节级）。 | [EIP-4444](https://eips.ethereum.org/EIPS/eip-4444)[^37] |
| 状态过期 (State Expiry) | 存储 | 将长期未被访问的状态修剪到“冷”存储或存档中。 | 无期限地保持活跃状态大小在可管理范围内，防止“状态膨胀”。 | [研究中 (in research)](https://notes.ethereum.org/@vbuterin/state_expiry_paths)[^38] |
| 协议清理 (Protocol Cleanup) | 技术债务 | 移除旧的预编译合约 (precompiles) 并简化 SELFDESTRUCT 等复杂操作码。 | 降低客户端复杂度、安全性风险暴露面和维护负担。 | EIP-6780 (已发布), EIP-7523 (已发布)[^39] |

### 挥霍 (The Splurge)
包含不适合其他类别但对以太坊的长期健康、用户体验和韧性至关重要的杂项改进。

**已实现 (IMPLEMENTED)**
| 升级 (Upgrade) | 类别 (Category) | 主题 (Topic) | 描述 (Description) | 效果 (Effect) | 最新进展 (State of the art) |
| :--- | :---: | :---: | :--- | :--- | :--- |
| EIP-1559 | 经济学 | 费用市场 | 引入了基础费用 (base fee) 和 ETH 销毁机制。 | 更具预测性的交易费用和 ETH 供应管理。 | [已发布 (shipped)](https://eips.ethereum.org/EIPS/eip-1559) |
| ERC-4337 | 用户体验 (UX) | 账户抽象 | 智能合约钱包标准，使用独立的内存池 (mempool) 和打包器 (bundlers)。 | 社交恢复、赞助交易以及批量操作。 | [已发布 (shipped)](https://eips.ethereum.org/EIPS/eip-4337)[^40] |

**待办 (TODO)**
| 升级 (Upgrade) | 类别 (Category) | 描述 (Description) | 预期效果 (Expected effect) | 最新进展 (State of the art) |
| :--- | :---: | :--- | :--- | :--- |
| EOF (EVM 对象格式, EVM Object Format) | EVM | 一种新型的 EVM 字节码容器格式，具有显式版本控制和结构化头部。 | 更安全、更高效的 EVM 执行；更易于静态分析。 | [EIP-7692](https://eips.ethereum.org/EIPS/eip-7692)[^41] |
| 账户抽象 (Account Abstraction, EIP-7702) | 用户体验 (UX) | 允许外部账户 (EOA) 在单次交易中临时作为智能合约。 | 在不强迫进行完全迁移的情况下，为现有钱包带来账户抽象 (AA) 功能。 | [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702)[^42] |
| 多维 Gas (Multidimensional Gas) | 经济学 | 对不同资源（执行、blobs、存储）进行独立定价和限制。 | 更好的资源分配和更高的网络吞吐量。 | [研究中 (in research)](https://vitalik.eth.limo/general/2024/05/09/multidim.html)[^43] |

---

## 参考文献 (Resources)

[^1] : [EIP-2982: Serenity Phase 0](https://eips.ethereum.org/EIPS/eip-2982), [[archived]](https://web.archive.org/web/20230928204358/https://eips.ethereum.org/EIPS/eip-2982)

[^2] : [EIP-4895: Beacon chain push withdrawals](https://eips.ethereum.org/EIPS/eip-4895), [[archived]](https://web.archive.org/web/20240415201815/https://eips.ethereum.org/EIPS/eip-4895)

[^3] : [VB's SSF notes](https://notes.ethereum.org/@vbuterin/single_slot_finality), [[archived]](https://web.archive.org/web/20240330010706/https://notes.ethereum.org/@vbuterin/single_slot_finality)

[^4] : [Sticking to 8192 signatures per slot post-SSF](https://ethresear.ch/t/sticking-to-8192-signatures-per-slot-post-ssf-how-and-why/17989). [[archived]](https://web.archive.org/web/20240105131126/https://ethresear.ch/t/sticking-to-8192-signatures-per-slot-post-ssf-how-and-why/17989)

[^5] : [A simple Single Slot Finality protocol](https://ethresear.ch/t/a-simple-single-slot-finality-protocol/14920), [[archived]](https://web.archive.org/web/20231214080806/https://ethresear.ch/t/a-simple-single-slot-finality-protocol/14920)

[^6] : [EIP-7441: Upgrade BPE to Whisk](https://eips.ethereum.org/EIPS/eip-7441), [[archived]](https://web.archive.org/web/20231001031437/https://eips.ethereum.org/EIPS/eip-7441)

[^7] : [EIP-7514: Add Max Epoch Churn Limit](https://eips.ethereum.org/EIPS/eip-7514), [[archived]](https://web.archive.org/web/20240309191714/https://eips.ethereum.org/EIPS/eip-7514)

[^8] : [EIP-7251:Increase the MAX_EFFECTIVE_BALANCE](https://eips.ethereum.org/EIPS/eip-7251), [[archived]](https://web.archive.org/web/20240324072459/https://eips.ethereum.org/EIPS/eip-7251)

[^9] : [Medium post on lattice encryption](https://medium.com/asecuritysite-when-bob-met-alice/so-what-is-lattice-encryption-326ac66e3175), [[archived]](https://web.archive.org/web/20230623222155/https://medium.com/asecuritysite-when-bob-met-alice/so-what-is-lattice-encryption-326ac66e3175)

[^10] : [VB's hackmd post on STARK signature aggregation](https://hackmd.io/@vbuterin/stark_aggregation), [[archived]](https://web.archive.org/web/20240313124147/https://hackmd.io/@vbuterin/stark_aggregation)

[^11] : [EIP-4844: Shard Blob Transactions](https://eips.ethereum.org/EIPS/eip-4844), [[archived]](https://web.archive.org/web/20240326205709/https://eips.ethereum.org/EIPS/eip-4844)

[^12] : [EIP-7594: PeerDAS](https://github.com/ethereum/EIPs/pull/8105) 

[^13] : [BoLd: dispute resolution protocol](https://github.com/OffchainLabs/bold/blob/e00b1c86124c3ca8c70a2cc50d9296e7a8e818ce/docs/research-specs/BOLDChallengeProtocol.pdf)

[^14] : [Fault proofs bring permissionless validation to the OP Sepolia testnet](https://blog.oplabs.co/open-source-and-feature-complete-fault-proofs-bring-permissionless-validation-to-the-op-sepolia-testnet/)

[^15] : [Parallel Zero-knowledge Virtual Machine](https://eprint.iacr.org/2024/387), [[archived]](https://web.archive.org/web/20240415180222/https://eprint.iacr.org/2024/387)

[^16] : [What is zkEVM](https://www.alchemy.com/overviews/zkevm), [[archived]](https://web.archive.org/web/20240129204732/https://www.alchemy.com/overviews/zkevm)

[^17] : [Types of ZK-EVMs](https://vitalik.eth.limo/general/2022/08/04/zkevm.html), [[archived]](https://web.archive.org/web/20240329112600/https://vitalik.eth.limo/general/2022/08/04/zkevm.html)

[^18] : [Barnabe - More pictures about proposers and builders](https://mirror.xyz/barnabe.eth/QJ6W0mmyOwjec-2zuH6lZb0iEI2aYFB9gE-LHWIMzjQ), [[archived]](https://web.archive.org/web/20240424010902/https://mirror.xyz/barnabe.eth/QJ6W0mmyOwjec-2zuH6lZb0iEI2aYFB9gE-LHWIMzjQ)

[^19] : [MEV burn—a simple design](https://ethresear.ch/t/mev-burn-a-simple-design/15590), [[archived]](https://ethresear.ch/t/mev-burn-a-simple-design/15590)

[^20] : [APS-Burn](https://mirror.xyz/barnabe.eth/QJ6W0mmyOwjec-2zuH6lZb0iEI2aYFB9gE-LHWIMzjQ#heading-aps-burn)

[^21] : [Inclusion lists](https://eips.ethereum.org/EIPS/eip-7547), [[archived]](https://web.archive.org/web/20240309191147/https://eips.ethereum.org/EIPS/eip-7547)

[^22] : [ROP-9: Multiplicity gadgets](https://efdn.notion.site/ROP-9-Multiplicity-gadgets-for-censorship-resistance-7def9d354f8a4ed5a0722f4eb04ca73b)

[^23] : [Committee-enforced inclusion sets (COMIS)](https://ethresear.ch/t/the-more-the-less-censored-introducing-committee-enforced-inclusion-sets-comis-on-ethereum/18835), [[archived]](https://web.archive.org/web/20240310000045/https://ethresear.ch/t/the-more-the-less-censored-introducing-committee-enforced-inclusion-sets-comis-on-ethereum/18835)

[^24] : [SUAVE](https://writings.flashbots.net/the-future-of-mev-is-suave), [[archived]](https://web.archive.org/web/20240310000045/https://writings.flashbots.net/the-future-of-mev-is-suave)

[^25] : [Examples of app layer MEV minimization](https://herccc.substack.com/i/142947825/examples-of-the-defensive-side-of-mev)

[^26] : [Based preconfirmations](https://ethresear.ch/t/based-preconfirmations/17353), [[archived]](https://web.archive.org/web/20240310000045/https://ethresear.ch/t/based-preconfirmations/17353)

[^27] : [EIP-7251: Increase the MAX_EFFECTIVE_BALANCE](https://eips.ethereum.org/EIPS/eip-7251)

[^28] : [Spin Up Your Own Ethereum Node - Ethereum.org](https://ethereum.org/en/developers/docs/nodes-and-clients/run-a-node/)

[^29] : [Paths to SSF](https://ethresear.ch/t/orbit-ssf-solo-staking-friendly-validator-set-management-for-ssf/19928)

[^30] : [Enshrining Liquid Staking/Decentralized Liquid Staking](https://notes.ethereum.org/@vbuterin/H1_5auGQd)

[^31] : [Enshrined LST from Arixon](https://ethresear.ch/t/enshrined-lst-allocating-stake-to-node-operators/11053)

[^32] : [Unbundling staking: towards rainbow staking](https://ethresear.ch/t/unbundling-staking-towards-rainbow-staking/11054)

[^33] : [Liquid Staking Maximalism design by Dankrad](https://ethresear.ch/t/liquid-staking-maximalism/11050)

[^34] : [Two-tiered staking from Mike Neuder](https://ethresear.ch/t/two-tiered-staking/11049)

[^35]: [Verkle Trees](https://verkle.info/)

[^36]: [`SNARKing` the Beacon Chain for Light Clients](https://ethresear.ch/t/snarking-the-beacon-chain-for-light-clients/14115)

[^37]: [EIP-4444: Bound Historical Data in Execution Clients](https://eips.ethereum.org/EIPS/eip-4444)

[^38]: [State expiry paths](https://notes.ethereum.org/@vbuterin/state_expiry_paths)

[^39]: [EIP-7523: Empty accounts deprecation](https://eips.ethereum.org/EIPS/eip-7523)

[^40]: [ERC-4337: Account Abstraction via Entry Point](https://eips.ethereum.org/EIPS/eip-4337)

[^41]: [EIP-7692: EVM Object Format (EOF2) Meta](https://eips.ethereum.org/EIPS/eip-7692)

[^42]: [EIP-7702: Set EOA implementation code](https://eips.ethereum.org/EIPS/eip-7702)

[^43]: [Multidimensional Gas Pricing](https://vitalik.eth.limo/general/2024/05/09/multidim.html)

[ethereum/EIPs github repository](https://github.com/ethereum/EIPs/tree/master#ethereum-improvement-proposals-eips)

[Roadmap on Ethereum.org](https://ethereum.org/en/roadmap/)

[ethroadmap.com](https://ethroadmap.com/)
