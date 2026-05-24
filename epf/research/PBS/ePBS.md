# 封装式提议者-构建者分离 (Enshrined Proposer-Builder Separation, ePBS)

## 路线图跟踪器 (Roadmap tracker)

| 升级 (Upgrade) | 阶段 (URGE) | 轨道 (Track) | 主题 (Topic) | 交叉引用 (Cross-references) |
|:-------:|:-----------:|:---------:|:---------------------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|  ePBS   | the Scourge | MEV 轨道 (MEV track) | 终局区块生产流水线 (Endgame block production pipeline) | 与以下内容的交集：[ET](https://ethresear.ch/t/execution-tickets/17944), [PEPC](https://efdn.notion.site/PEPC-FAQ-0787ba2f77e14efba771ff2d903d67e4), [IL](https://eips.ethereum.org/EIPS/eip-7547) |

## 简述 (TLDR;)

封装式提议者-构建者分离 (Enshrined Proposer-Builder Separation, ePBS) 是指将 提议者-构建者分离 (Proposer-Builder Separation, PBS) 机制直接集成到以太坊的核心协议中。这在核心协议内正式确立了 区块提议者 (block proposers) 和 区块构建者 (block builders) 之间的分离，旨在提高效率、安全性和去中心化 (decentralization)。

## 什么是 PBS (What is PBS)

提议者-构建者分离 (Proposer-Builder Separation, PBS) 是一种设计哲学[^1]，它将提议区块（提议者 (proposers)）和构建区块（构建者 (builders)）的角色分离开来。这种分离有助于解决区块生产中的挑战和低效问题，特别是关于 最大可提取价值 (Maximum Extractable Value, MEV) 的问题。本文假设读者对 [MEV](/wiki/research/PBS/mev.md)、[PBS](/wiki/research/PBS/pbs.md) 和 [MEV-Boost](/wiki/research/PBS/mev-boost.md) 有一定的了解。

## ePBS 概述 (Overview of ePBS)

与目前像 [MEV-Boost](/wiki/research/PBS/mev-boost.md) 这样的协议外外部解决方案不同，ePBS 将 PBS 直接集成到以太坊的核心协议中。这种集成旨在通过消除对外部中继 (relays) 的需求来简化流程并增强安全性。

### 关键区别 (Key Differences)

- **提议者 (Proposers)**：提议新区块的 验证者 (validators)，专注于选择区块而无需亲自构建它。
- **构建者 (Builders)**：组装区块以优化交易顺序，并通过透明拍卖 (transparent auction) 向提议者提供区块的实体或算法[^2][^3]。
- **中继 (Relays)**：在传统的 MEV-Boost 中，中继介导提议者与构建者之间的通信。在 ePBS 下，协议本身促进了这种交互，从而减少或重新定义了对外部中继的需求。

### 从 mev-boost 向 ePBS 的过渡 (Transition from mev-boost to ePBS)

向 ePBS 的过渡代表了一个重大转变，旨在消除对第三方软件的依赖。下一节将深入探讨为什么需要 ePBS 以及这种协议内方法的益处。

## 需要 ePBS 的原因 (The Case for ePBS)

从 MEV-Boost 过渡到 ePBS 解决了与以太坊核心价值一致的几个关键问题[^4]：

### 主要原因 (Main Reasons)

- **去中心化 (Decentralization)**：减少对中心化中继的依赖，更广泛地分配区块构建职责。
- **安全性 (Security)**：确保区块构建遵循与网络其他部分相同的安全规则，这与目前的链下中继不同。
- **效率和公平性 (Efficiency and Fairness)**：为区块空间创造一个更加透明和竞争的市场，减少与 MEV 提取相关的低效和不公平现象。

### 当前系统面临的挑战 (Challenges with Current System)

- **中心化风险 (Centralization Risks)**：依赖极少数中继可能会引入中心化风险和单点故障 (single points of failure)。
- **安全漏洞 (Security Vulnerabilities)**：在以太坊共识规则之外运行的外部中继会带来安全风险。
- **与 MEV 相关的操纵 (MEV-Related Manipulations)**：当前的系统可能无法针对 MEV 操纵提供足够的保护。
- **运营可持续性 (Operational Sustainability)**：中继的长期可行性是不确定的，这引发了对其持续有效性和财务可持续性的担忧。

### 经济与安全考量 (Economic and Security Considerations)

- **MEV 分配 (MEV Distribution)**：ePBS 旨在创建一个更加透明和公平的 MEV 市场，改变目前的动态。
- **市场效率 (Market Efficiency)**：通过将 PBS 集成到协议中，ePBS 培育了一个更具竞争力的市场，潜在地减少了 Gas 价格拍卖和网络拥堵。
- **中心化风险 (Centralization Risks)**：当前对极少数中继的依赖引入了中心化风险，而 ePBS 寻求减轻这些风险。
- **操纵行为 (Manipulative Practices)**：外部中继可能导致操纵行为，而 ePBS 旨在对其进行更好的监管。

### MEV 销毁 (MEV Burn)

MEV 销毁 (MEV-burn) 涉及将一部分 MEV 利润重定向用于销毁 Ether，从而减少供应量 (supply) 并潜在地增加其价值。该机制旨在使验证者、构建者和社区的利益保持一致。

## 对 ePBS 的反对意见 (Counterarguments to ePBS)

ePBS 的批评者提出了一些担忧，而支持者则给出了有力的回应。讨论还探讨了解决 MEV 的其他替代方法[^3]。

### 主要反对意见 (Primary Counterarguments)

- **复杂性和技术风险 (Complexity and Technical Risk)**：ePBS 向协议引入了新元素，这可能会增加复杂性和技术风险。
- **网络性能 (Network Performance)**：担忧 ePBS 影响网络性能，特别是延迟 (latency) 和区块传播时间。
- **降低验证者的灵活性 (Reduced Flexibility for Validators)**：担心 ePBS 可能会限制验证者在区块提案中的选择。
- **过早优化 (Premature Optimization)**：表明 ePBS 可能会转移对更紧迫问题的注意力。
- **可绕过性 (Bypassability)**：验证者和构建者可能会继续依赖外部解决方案，从而绕过 ePBS 机制[^8]。

### 支持者的回应 (Proponents' Responses)

- **减轻复杂性 (Mitigating Complexity)**：倡导者认为 ePBS 的益处超过了复杂性和风险。
- **网络效率 (Network Efficiency)**：支持者认为，ePBS 的设计可以维持网络效率。
- **增强的决策制定 (Enhanced Decision-Making)**：ePBS 提供了更透明、更具竞争力的构建者市场，促进了去中心化。
- **战略重要性 (Strategic Importance)**：ePBS 解决了关于 MEV 提取的根本性隐忧，并符合以太坊的长期目标。

围绕 ePBS 的辩论体现了关于以太坊未来方向的更广泛讨论，在创新与谨慎之间取得了平衡。虽然反对意见强调了风险和替代方案，但 ePBS 的倡导者强调了它与以太坊核心原则的一致性，以及它对于确保公平、安全和高效的区块链生态系统的必要性[^5]。

## 设计 ePBS (Designing ePBS)

### ePBS 机制的理想属性 (Desirable properties of ePBS mechanisms)

ePBS 机制应确保去中心化、安全性、效率、透明度、降低信任要求以及公平的 MEV 分配[^6]。

**最小可行 ePBS (MVePBS) 的核心属性**

- **去中心化与公平准入 (Decentralization and Fair Access)**：确保没有单一实体控制区块提议和构建，使每个人都保持公平。
- **安全与完整性 (Security and Integrity)**：通过防范审查和双重支出 (double-spending) 等攻击，维护以太坊的安全性。
- **效率与可扩展性 (Efficiency and Scalability)**：在不增加显著开销的情况下，提高网络的效率和可扩展性。
- **透明度与可预测性 (Transparency and Predictability)**：使交易排序和区块生产对所有用户清晰且易于理解。
- **降低信任要求 (Reduced Trust Requirements)**：通过在协议中嵌入 PBS，最小化不同参与方之间对信任的需求。
- **MEV 分配的公平性 (Fairness in MEV Distribution)**：确保 MEV 机会得到公平分配，防止垄断控制 (monopolistic control)。
- **最小信任假设 (Minimal Trust Assumptions)**：通过使用密码学和博弈论原理，减少对信任的需求。
- **经济可行性 (Economic Viability)**：确保参与 ePBS 对提议者、构建者和验证者在财务上都是有利的。
- **灵活性与兼容性 (Flexibility and Compatibility)**：设计 ePBS 以容纳未来的创新并能与现有基础设施协同工作。
- **降低用户的复杂度 (Reduced Complexity for Users)**：尽管增加了协议复杂度，但系统要保持用户友好和简单。
- **激励一致性 (Incentive Alignment)**：使所有网络参与者的利益一致，以确保长期的稳定和健康。

**ePBS 的最小无信任系统 (Minimal trustless system for ePBS)**

一个最小无信任 ePBS 系统应该包括以下属性：诚实构建者发布安全性 (honest builder publication safety)、构建者揭示安全性 (builder reveal safety)、强制性诚实重组 (mandatory honest reorgs)、构建者隐匿安全性 (builder withholding safety)、无条件付款 (unconditional payment)、提议者安全性 (proposer safety) 以及抗审查性[^6]。

- **诚实构建者发布安全性 (Honest Builder Publication Safety)**：确保按时发布区块的构建者被包含在内，防范审查并认可其工作。
- **构建者揭示安全性 (Builder Reveal Safety)**：保护构建者在揭示区块内容和竞价时免受提议者双签的影响。
- **强制性诚实重组 (Mandatory Honest Reorgs)**：强制执行重组规则，以阻止审查或剥削 MEV 的恶意重组。
- **构建者隐匿安全性 (Builder Withholding Safety)**：保证对在指南范围内策略性隐匿区块的构建者不进行不公平惩罚。
- **无条件付款 (Unconditional Payment)**：确保构建者为其工作获得报酬，无论区块是否被包含，从而最小化信任。
- **提议者安全性 (Proposer Safety)**：保护提议者免于失去补偿，鼓励参与区块提案。
- **诚实构建者付款安全性 (Honest Builder Payment Safety)**：保证在区块生产中满足协议期望的构建者获得付款。
- **无许可性 (Permissionlessness)**：允许任何人成为构建者或提议者而无需中央授权，促进竞争和去中心化。
- **抗审查性 (Censorship Resistance)**：防止区块和交易遭到审查，维护以太坊的去中心化完整性。
- **路线图兼容性 (Roadmap Compatibility)**：使 ePBS 与以太坊的发展路线图保持一致，以便与未来的升级顺利集成。

此外，在最小无信任 ePBS 系统中，提议者不应具有在协议外出售区块的优势。

**辩论最佳机制 (Debating the Optimal Mechanism)**

围绕最佳 ePBS 机制的讨论需要仔细权衡设计选择，这通常涉及取舍[^9][^3]。

- **拍卖机制 (Auction Mechanisms)**：决定是使用次价拍卖 (second-price auctions) 还是像频繁批量拍卖 (frequent batch auctions) 这样更复杂的拍卖，会影响公平性、效率和对操纵的敏感性。
- **构建者选择与质押 (Builder Selection and Staking)**：选择如何选择构建者（通过质押、声誉系统 (reputation systems) 还是随机选择）会影响去中心化和安全性，而质押可能会将机会集中在富裕者中。
- **包含与抗审查 (Inclusion and Censorship Resistance)**：确保系统抵抗审查并公平地包含交易可能涉及匿名提交或强制交易列表，同时要与防范垃圾邮件或恶意交易进行平衡。
- **兼容性与适应性 (Compatibility and Adaptability)**：ePBS 必须与当前的以太坊基础设施兼容，并且对未来的更新（如分片 (sharding) 或新的 Layer-2 解决方案）保持灵活。

寻找最小可行 ePBS 涉及平衡这些维度，以促进去中心化，确保安全和效率，保持透明和公平，并最小化信任要求。持续的社区讨论和研究对于完善这些概念并达成对最佳 ePBS 机制的共识至关重要。

## ePBS 解决方案 (ePBS Solutions)

### 双区块头锁 (TBHL) 提案 (The Two-Block HeadLock (TBHL) proposal)

双区块头锁 (TBHL) 设计代表了在以太坊协议中实现提议者-构建者分离 (PBS) 的一种创新方法，旨在解决由 [MEV 带来的运营和战略问题](https://ethresear.ch/t/why-enshrine-proposer-builder-separation-a-viable-path-to-epbs/)。该设计是对先前提案的细致迭代，融合了 Vitalik Buterin 的双时隙设计 (two-slot design) 的元素，并通过头锁机制 (headlock mechanism) 对其进行了增强，以保护构建者免受提议者双签的影响。在此，我们根据提供的详细解释，深入探讨 TBHL 的关键组件及其运行机制[^4]。

[TBHL 提案](/docs/wiki/research/PBS/TBHL.md) 包含了有关该设计和流程的更多细节。

### 负载时效性委员会 (PTC) 以支持 ePBS (Payload-Timeliness Committee (PTC) for ePBS)

负载时效性委员会 (PTC) 提案是一种在以太坊协议中封装 PBS (ePBS) 的设计。它代表了确定区块有效性机制的演变，并包含了一个验证者子集，他们对区块负载的时效性进行投票[^7][^12]。

[PTC 提案](/docs/wiki/research/PBS/PTC.md) 包含了有关该设计和流程的更多细节。

### ePBS PTC 规范概述 (ePBS PTC Specifications Overview)

[当前的 ePBS 规范](https://hackmd.io/@potuz/rJ9GCnT1C) 和 [GitHub 仓库](https://github.com/potuz/consensus-specs/tree/epbs_stripped_out/specs/_features/epbs) 被划分为独立的组件，以构建在现有的以太坊组件规范之上[^15][^16][^23]。
- `Beacon-chain.md`：此文件指定了 ePBS 分叉的信标链规范[^18]。
- `Validator.md`：此文件指定了 ePBS 分叉的诚实验证者行为规范[^19]。
- `Builder.md`：此文件指定了 ePBS 分叉的诚实构建者规范[^20]。
- `Engine.md`：此文件指定了因 ePBS 分叉而引起的引擎 API (Engine API) 更改[^21]。
- `fork-choice.md`：此文件指定了由于 ePBS 分叉对分叉选择 (fork-choice) 带来的修改[^22]。

[ePBS 设计规范 (ePBS design specs)](/docs/wiki/research/PBS/ePBS-Specs.md) 包含了更多关于实现规范和流程的细节。

### 协议强制提议者承诺 (Protocol-Enforced Proposer Commitments, PEPC)

协议强制提议者承诺 (Protocol-Enforced Proposer Commitments, PEPC) 是 PBS 的一种概念性扩展和泛化，它为提议者（验证者）承诺区块构建提供了一种更灵活、更安全的方法。与现有的 MEV-Boost 机制不同（该机制依赖于提议者与构建者/中继之间的协议外协定），PEPC 旨在将这些承诺封装在以太坊协议本身中，为这些交互提供无信任且无许可的 协议内基础设施[^10][^11]。

[PEPC 提案](/wiki/research/PBS/PEPC.md) 包含了有关该设计和流程的更多细节。

## ePBS 的未决问题 (Open Questions in ePBS)

Mike Neuder 强调了关于在以太坊协议中封装 PBS (ePBS) 的几个引人入胜且颇具挑战性的问题[^5]。

### 可绕过性意味着什么？ (What does bypassability imply?)

可绕过性是指验证者和构建者可能会继续使用外部中继或解决方案，而不是封装的协议。如果许多参与者选择退出，这会引发对 ePBS 有效性的担忧，并且在不限制验证者自治或以太坊去中心化性质的情况下，创建一个无法被绕过的系统是一项挑战。

### 封装旨在实现什么？ (What does enshrining aim to achieve?)

封装 PBS 旨在在以太坊协议中创建一个中立的、无信任的中继，以保护提议者-构建者关系。这一努力寻求减少像 MEV-Boost 这样的外部解决方案的中心化风险，提高抗审查性，并更系统地解决与 MEV 相关的问题。即使具有可绕过性，封装也能提供可靠的退路，鼓励直接的协议参与，并支持诸如 MEV 销毁机制（例如 MEV-burn）之类的长期目标。

### 不封装的确切影响是什么？ (What are the exact implications of not enshrining?)

不封装 PBS 会使外部系统对区块构建和 MEV 分配保持相当大的控制，可能增加中心化和安全漏洞。这一选择可能需要优先支持中立中继作为关键基础设施，以维持公平并防止 MEV 市场中的垄断行为。

### 对 ePBS 的真实需求是什么？ (What is the real demand for ePBS?)

以太坊内部对 ePBS 的需求是由解决当前局限性以及为未来可扩展性、安全性和去中心化需求做准备等因素驱动的。这种需求源于区块生产的不断演变以及 MEV 机会的复杂性。量化这种需求具有挑战性，但它反映了以太坊社区解决当前问题并预测未来要求的更广泛需求。

### 我们在多大程度上可以依赖利他主义和社会层？ (How much can we rely on altruism and the social layer?)

社会层 (social layer) 由以太坊的社区规范和价值观组成，它在那些仅靠协议机制无法强制执行的行为中起着至关重要作用。虽然利他主义或长期的自我利益可能会促使一些大型 ETH 持有者支持封装解决方案，但仅仅依赖这些动机是不可靠的。以太坊的完整性和去中心化应该由强大、可强制执行的机制来支持，而不是靠对理想的自愿遵守。

### 在拥有 L2 和 OFA 的未来中，L1 ePBS 有多重要？ (How important is L1 ePBS in a future with L2s and OFAs?)

随着以太坊路线图的推进，随着 L2 解决方案的更多活动以及 OFA 的开发，L1 MEV 的直接影响可能会减少。然而，由 ePBS 建立的原则和基础设施对于跨层的安全和去中心化 MEV 管理仍然可能是至关重要的，从而维护了 ePBS 在多层生态系统中的相关性。

### 鉴于其他协议升级，ePBS 应该具有什么优先级？ (What priority should ePBS have in light of other protocol upgrades?)

鉴于复杂的格局以及以太坊 MEV 动态的潜在转变，ePBS 相对于其他升级（例如抗审查性、单时隙最终性）的优先级需要采取战略性方法。社区可能会专注于能够带来即时益处且风险较低的升级，同时继续开发可以根据需要快速部署的 ePBS 框架。

## 资源 (Resources)
- [关于提议者-构建者分离 (PBS) 的笔记 (Notes on Proposer-Builder Separation (PBS))](https://barnabe.substack.com/p/pbs)
- [Mike Neuder - 迈向封装式提议者-构建者分离 (Towards Enshrined Proposer-Builder Separation)](https://www.youtube.com/watch?v=Ub8V7lILb_Q)
- [PBS 不完全指南 - Mike Neuder 和 Chris Hager (An Incomplete Guide to PBS - with Mike Neuder and Chris Hager)](https://www.youtube.com/watch?v=mEbK9AX7X7o)
- [为什么封装提议者-构建者分离？通往 ePBS 的可行路径 (Why enshrine Proposer-Builder Separation? A viable path to ePBS)](https://ethresear.ch/t/why-enshrine-proposer-builder-separation-a-viable-path-to-epbs/15710/1)
- [ePBS – 无穷自助餐 (ePBS – the infinite buffet)](https://notes.ethereum.org/@mikeneuder/infinite-buffet)
- [ePBS 设计限制 (ePBS design constraints)](https://hackmd.io/ZNPG7xPFRnmMOf0j95Hl3w)
- [负载时效性委员会 (PTC) – 一种 ePBS 设计 (Payload-timeliness committee (PTC) – an ePBS design )](https://ethresear.ch/t/payload-timeliness-committee-ptc-an-epbs-design/16054)
- [考虑 ePBS (Consider the ePBS)](https://notes.ethereum.org/@mikeneuder/consider-the-epbs)
- [ePBS 分组讨论室 (ePBS Breakout Room)](https://www.youtube.com/watch?v=63juNVzd1P4)
- [解耦 PBS：迈向协议强制提议者承诺 (PEPC) (Unbundling PBS: Towards protocol-enforced proposer commitments (PEPC))](https://ethresear.ch/t/unbundling-pbs-towards-protocol-enforced-proposer-commitments-pepc/13879/1)
- [PEPC 常见问题解答 (PEPC FAQ)](https://efdn.notion.site/PEPC-FAQ-0787ba2f77e14efba771ff2d903d67e4)
- [没有 Max EB 和 7002 的最小 ePBS (Minimal ePBS without Max EB and 7002)](https://github.com/potuz/consensus-specs/pull/2)
- [EigenLayer 协议 (EigenLayer protocol)](https://docs.eigenlayer.xyz/eigenlayer/overview/whitepaper)
- [ePBS 规范笔记 (ePBS specification notes)](https://hackmd.io/@potuz/rJ9GCnT1C)
- [ePBS 设计规范 PR (ePBS design specs PR)](https://github.com/potuz/consensus-specs/pull/2)
- [ePBS 设计规范 GitHub 仓库 (ePBS design specs GitHub repo)](https://github.com/potuz/consensus-specs/tree/epbs_stripped_out/specs/_features/epbs)
- [epbs - 信标链规范 (epbs - beacon-chain specs)](https://github.com/potuz/consensus-specs/blob/epbs_stripped_out/specs/_features/epbs/beacon-chain.md)
- [epbs - 诚实验证者规范 (epbs - honest validator specs)](https://github.com/potuz/consensus-specs/blob/epbs_stripped_out/specs/_features/epbs/validator.md)
- [epbs - 诚实构建者规范 (epbs - honest builder specs)](https://github.com/potuz/consensus-specs/blob/epbs_stripped_out/specs/_features/epbs/builder.md)
- [epbs - 引擎 API 规范 (epbs - Engine API specs)](https://github.com/potuz/consensus-specs/blob/epbs_stripped_out/specs/_features/epbs/engine.md)
- [epbs - 分叉选择规范 (epbs - fork-choice specs)](https://github.com/potuz/consensus-specs/blob/epbs_stripped_out/specs/_features/epbs/fork-choice.md)
- [EIP-7547 包含列表 (EIP-7547 Inclusion Lists)](https://eips.ethereum.org/EIPS/eip-7547)
- [更多关于提议者和构建者的图片 (图表) (More pictures about proposers and builders (Diagrams))](https://mirror.xyz/barnabe.eth/QJ6W0mmyOwjec-2zuH6lZb0iEI2aYFB9gE-LHWIMzjQ)

## 参考文献 (References)
[^1]: https://barnabe.substack.com/p/pbs
[^2]: https://www.youtube.com/watch?v=Ub8V7lILb_Q
[^3]: https://www.youtube.com/watch?v=mEbK9AX7X7o
[^4]: https://ethresear.ch/t/why-enshrine-proposer-builder-separation-a-viable-path-to-epbs/15710/1
[^5]: https://notes.ethereum.org/@mikeneuder/infinite-buffet
[^6]: https://hackmd.io/ZNPG7xPFRnmMOf0j95Hl3w
[^7]: https://ethresear.ch/t/payload-timeliness-committee-ptc-an-epbs-design/16054
[^8]: https://notes.ethereum.org/@mikeneuder/consider-the-epbs
[^9]: https://www.youtube.com/watch?v=63juNVzd1P4
[^10]: https://ethresear.ch/t/unbundling-pbs-towards-protocol-enforced-proposer-commitments-pepc/13879/1
[^11]: https://efdn.notion.site/PEPC-FAQ-0787ba2f77e14efba771ff2d903d67e4
[^12]: https://hackmd.io/@potuz/rJ9GCnT1C
[^13]: https://github.com/potuz/consensus-specs/pull/2
[^14]: https://docs.eigenlayer.xyz/eigenlayer/overview/whitepaper
[^15]: https://hackmd.io/@potuz/rJ9GCnT1C
[^16]: https://github.com/potuz/consensus-specs/pull/2
[^17]: https://eips.ethereum.org/EIPS/eip-7547
[^18]: https://github.com/potuz/consensus-specs/blob/epbs_stripped_out/specs/_features/epbs/beacon-chain.md
[^19]: https://github.com/potuz/consensus-specs/blob/epbs_stripped_out/specs/_features/epbs/validator.md
[^20]: https://github.com/potuz/consensus-specs/blob/epbs_stripped_out/specs/_features/epbs/builder.md
[^21]: https://github.com/potuz/consensus-specs/blob/epbs_stripped_out/specs/_features/epbs/engine.md
[^22]: https://github.com/potuz/consensus-specs/blob/epbs_stripped_out/specs/_features/epbs/fork-choice.md 
[^23]: https://github.com/potuz/consensus-specs/tree/epbs_stripped_out/specs/_features/epbs
