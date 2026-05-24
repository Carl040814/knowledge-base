# 封装式 PBS 的双区块头锁 (Two-Block HeadLock, TBHL) 提案

双区块头锁 (Two-Block HeadLock, TBHL) 设计代表了在以太坊协议中实现 [提议者-构建者分离 (Proposer-Builder Separation, PBS)](/docs/wiki/research/PBS/pbs.md) 的一种创新方法，旨在解决由 [MEV 带来的运营和战略问题](https://ethresear.ch/t/why-enshrine-proposer-builder-separation-a-viable-path-to-epbs/)。该设计是对先前提案的细致迭代，融合了 Vitalik Buterin 的双时隙设计 (two-slot design) 的元素，并通过头锁机制 (headlock mechanism) 对其进行了增强，以保护构建者 (builders) 免受提议者 (proposer) 双签行为 (equivocations) 的影响。在此，我们根据提供的详细解释，深入探讨 TBHL 的关键组件及其运行机制[^1]。

## TBHL 设计概述 (TBHL Design Overview)

TBHL 修改了以太坊中传统的 时隙 (slot) 结构，在单个时隙时间框架内引入了双区块系统，从而有效地产生了一个 提议者区块 (proposer block) 和一个 构建者区块 (builder block)。该系统保留了每个时隙单个 执行负载 (execution payload) 的实质，但增加了一轮 见证 (attestations)，这可能会延长时隙持续时间。TBHL 与以太坊现有的 LMD-GHOST 机制高度契合，并遵循六个特定的设计属性，确保了以太坊生态系统内的兼容性和完整性。

## 时隙结构和运行阶段 (Slot Anatomy and Operational Phases)

![TBHL 的时隙结构](../img/scaling/Slot-Anatomy-of-TBHL-Mike.png)


_图 – TBHL 的时隙结构。图片信用：mike neuder 和 justin drake。_

TBHL 的运行框架围绕一个时隙内的四个关键时间戳展开，为提议者、构建者和见证者 划定了特定操作：

1. **t=t0 - 提交获胜竞价 (Proposal of the Winning Bid)**：提议者 (proposer) 首先评估 竞价池 (bidpool) 内的出价，竞价池是一个点对点 (peer-to-peer, P2P) 主题，构建者在该主题中提交其竞价。选定竞价后，提议者在进入下一阶段之前发布提议者区块。

2. **t=t1 - 提议者区块的见证截止时间 (Attestation Deadline for Proposer Block)**：在此阶段，见证委员会 (attesting committee) 评估提议者区块的时效性。他们投票给首次观察到的区块，或者在没有观察到区块时，投票给空时隙。

3. **t=t1.5 - 双签检查 (Equivocation Check)**：负责构建者区块的见证委员会评估提议者区块是否存在 双签行为 (equivocations)。唯一的提议者区块将为相关构建者触发 提议者权重提升 (proposer boost)，从而增强该过程的公平性和完整性。

4. **t=t2 - 构建者验证和区块发布 (Builder's Verification and Block Publication)**：构建者 (builders) 验证自己是否为唯一的获胜者。如果发生双签，他们可以生产一个包含证据的区块以撤销付款。否则，他们将继续发布包含交易内容的构建者区块。

5. **t=t3 - 第二次见证截止时间 (Second Attestation Deadline)**：此阶段涉及另一轮见证，这次是针对构建者区块的，以巩固其在区块链中的地位。

## 满足 ePBS 设计属性 (Satisfying ePBS Design Properties)

TBHL 有效地解决了几项关键的 封装式提议者-构建者分离 (Enshrined Proposer-Builder Separation, ePBS) 设计属性：

- **诚实构建者发布和支付安全 (Honest Builder Publication and Payment Safety)**：保护机制确保构建者可以充满信心率地发布区块，并防范提议者双签行为。

- **诚实提议者安全 (Honest Proposer Safety)**：诚实提议者所做的承诺会得到尊重，他们的区块会获得必要的见证，并且除非提交了双签证明，否则无条件付款将继续进行。

- **无许可性和抗审查性 (Permissionlessness and Censorship Resistance)**：该设计为构建者促进了一个开放和竞争的环境，同时维持了打击审查的措施。

- **路线图兼容性 (Roadmap Compatibility)**：TBHL 准备与以太坊的路线图无缝集成，包括 单时隙最终性 (Single Slot Finality, SSF) 和 MEV 销毁 (MEV-burn) 机制，展示了其在满足未来网络需求方面的适应性和前瞻性。

## 实现 TBHL 的工程挑战 (Engineering Challenges of implementing TBHL)

实现 TBHL 提案引入了多项微妙的挑战和工程问题，这反映了以太坊共识机制 (consensus mechanism)、动态 MEV 格局与协议总体设计哲学之间复杂的相互作用。根据 ePBS 讨论中的深入探讨，以下是与 TBHL 相关的主要实现问题和工程缺陷 (engineering drawbacks)：

- **协议复杂度增加 (Increased Protocol Complexity)**：TBHL 通过在单个时隙内引入双区块机制显着改变了传统的时隙结构，使共识过程复杂化。这种复杂性源于管理两种不同类型的区块（提议者和构建者区块），并需要额外的见证轮次来验证每一个区块。通过检测和管理提议者区块双签行为的需要，复杂性被进一步放大，这需要强大的机制来确保构建者安全和支付安全。

- **时隙定时和网络延迟 (Slot Timing and Network Latency)**：实现 TBHL 需要仔细考虑时隙定时，因为额外的见证轮次可能会延长时隙持续时间。这种调整会影响网络延迟 (network latency)，可能影响区块传播和见证聚合的时效性。确保网络能够高效处理这些过程，而不会引入显着的延迟或脆弱性，是一项重大的工程挑战。此外，增加时隙持续时间在合并后的以太坊中是一项非常庞大的工程项目，因为它需要对 共识层 (Consensus Layer, CL)、执行层 (Execution Layer, EL) 以及其他智能合约 (smart contracts) 层进行修改。

- **双签和构建者安全机制 (Equivocation and Builder Safety Mechanisms)**：TBHL 的关键组件之一是其管理提议者双签以保护构建者的方法。设计和实现强大的机制来检测双签，允许构建者提供此类事件的证明，并据此撤销付款，这引入了显着的复杂性。这些机制必须是万无一失的，以防止被剥削并确保系统的完整性，这需要进行严格的测试，并可能针对发现的漏洞进行迭代。

- **无许可性和抗审查性 (Permissionlessness and Censorship Resistance)**：虽然 TBHL 旨在维护无许可环境并对抗审查，但在新框架内实现这些目标面临挑战。确保任何构建者都可以提交竞价，并且提议者区块能够公平地代表竞争格局，需要对竞价池进行透明且安全的操作。此外，在 TBHL 结构中集成前向包含列表 (forward inclusion lists) 或其他抗审查机制，需要额外的协议考量，以防止操纵或排他性行为。

- **与现有及未来以太坊特性的兼容性 (Compatibility with Existing and Future Ethereum Features)**：TBHL 必须与以太坊当前的协议特性无缝集成，并能适应未来的创新，如单时隙最终性 (SSF) 和 MEV 销毁 (MEV-burn) 机制。确保与以太坊生态系统这些不断演变的部分相兼容，需要一种前瞻性的设计和实现方法，以便在不破坏 TBHL 框架完整性或有效性的情况下容纳调整和增强。

- **资源和计算开销 (Resource and Computational Overheads)**：TBHL 的引入带来了新的计算和资源开销，特别是与处理双区块机制产生的数据量增加以及额外的见证轮次相关的开销。优化协议以高效管理这些需求，而不会显着增加验证者 (validators) 的计算负担或损害网络性能，是一个核心工程关注点。

TBHL 提案证明了在完善以太坊 PBS 机制方面所做的持续努力，力求在运营效率、安全性和去中心化 (decentralization) 的总体理念之间取得平衡。通过解决提议者-构建者动态的细微差别并引入强大的保护措施，TBHL 标志着以太坊协议设计演变中向前迈出的重要一步，为减轻 MEV 带来的挑战同时增强网络的弹性和完整性提供了一条充满希望的途径。

您可以去 [PTC](/docs/wiki/research/PBS/PTC.md) 和 [PEPC](/docs/wiki/research/PBS/PEPC.md) 了解更多关于不同 ePBS 解决方案的信息。

## 资源 (Resources)
- [为什么封装提议者-构建者分离？通往 ePBS 的可行路径 (Why enshrine Proposer-Builder Separation? A viable path to ePBS)](https://ethresear.ch/t/why-enshrine-proposer-builder-separation-a-viable-path-to-epbs/15710/1)
- [PBS](/docs/wiki/research/PBS/pbs.md)
- [ePBS](/docs/wiki/research/PBS/epbs.md)
- [Mike Neuder - 迈向封装式提议者-构建者分离 (Towards Enshrined Proposer-Builder Separation)](https://www.youtube.com/watch?v=Ub8V7lILb_Q)

## 参考文献 (References)
[^1]: https://ethresear.ch/t/why-enshrine-proposer-builder-separation-a-viable-path-to-epbs/15710/1