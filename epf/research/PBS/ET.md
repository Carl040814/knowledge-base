# 执行票 (Execution Tickets, ET)
> [!WARNING]
> 本文档涵盖了活跃的研究领域，在阅读时可能会过时，并可能会随着围绕 ePBS、执行票 (ET) 和 包含列表 (Inclusion lists, IL) 的设计空间的演变而在未来进行更新。

## 路线图跟踪器 (Roadmap tracker)

| 升级 (Upgrade) | 阶段 (URGE) | 轨道 (Track) | 主题 (Topic) | 交叉引用 (Cross-references) |
| :-----: | :---------: | :-------: | :-------------------------------: | :--------------------------------------------------------------------------------------------------------------------------------: |
|   ET    | the Scourge | MEV 轨道 (MEV track) | 终局区块生产流水线 (Endgame block production pipeline) | 与以下内容的交集：[IL](/wiki/research/inclusion-lists.md), [ePBS](/wiki/research/PBS/ePBS.md), [PEPC](/wiki/research/PBS/PEPC.md) |

## 相关背景 (Relevant context)

最近关于封装 提议者-构建者分离 ([PBS](/wiki/research/PBS/pbs.md)) 的提案和开发工作，旨在减少执行区块构建对少数中心化的、协议外实体（中继 (relays)）的依赖。在当前的 MEV 背景下，中继充当了作为负载提议者 (payload proposers) 的验证者 (Validators) 与专业的、更复杂的区块构建者之间的中介。今天，在 [MEV-boost](/wiki/research/PBS/mev-boost.md) 中，验证者作为提议者，在开放竞价的无许可拍卖 (open-bids permissionless auction) 中放弃了构建 执行负载 (execution payloads) 的权利，而 构建者 (Builders) 则通过向 提议者 (Proposers) 支付费用来换取提供一个良好排序的负载。

![MEV-boost](/docs/wiki/research/img/MEV-boost.webp)

## 见证者-提议者分离 (Attester-Proposer Separation, APS)（又称 验证者-提议者分离）

MEV-boost 是一种 协议外边车 (out-of-protocol side-car)，超出了协议的管辖和控制范围。在 PBS 的背景下，[见证者-提议者分离 (Attester-Proposer Separation, APS)](#见证者-提议者分离-attester-proposer-separation-aps又称-验证者-提议者分离) 的设计哲学解决了这一限制，其目标是将这些基础设施中的一部分重新纳入协议的管辖范围。

APS 的设计逻辑与 **ePBS（封装式提议者-构建者分离）** 的设计空间密切相关。

根据关于 ePBS 的 EPS 维基页面 [[1]](#资源-resources) 和 Barnabé 关于 PBS 的笔记 [[2]](#资源-resources), 我们可以确定 ePBS 的两个主要组成部分：
1. 市场结构 (market structure) - 参与市场的各方及其参与条件。
2. 分配机制 (allocation mechanism) - 参与方之间可以订立的合同空间。

今天提出的 ePBS 既封装了特定的市场结构，又封装了主要继承自 MEV-Boost 的分配机制：

验证者作为提议者将其负载生产权拍卖给构建者。
与当前的 MEV-boost 相比，这将是一项改进，因为这将在提议者和构建者之间拆分验证者角色，为构建者提供公平的竞争环境，并且会有一个无信任的、由协议定义的市场结构来确保市场各方之间的 *公平交换*。

然而，这种设置仍然需要 MEV-boost 内部复杂的公平交换机制，只不过这种复杂性被移到了协议内部。
Barnabé 认为 [[3]](#资源-resources)，当前提议版本的 ePBS 并没有回答一些根本性的问题：
- 我们认为是 市场结构 *还是* 分配机制 在 MEV-boost 下显得过于中心化？
- 如果答案是前者，那么将一个我们既视为技术债（一个 Bug (technical debt (a bug))）又视为有缺陷的设计哲学（该设计目前仅存在于协议之外）的机制封装进协议，是否值得？
- **对于协议来说**，不分裂成两个世界，只关注分配 *提议权 (proposing rights)* 而不去操心分配 *构建权 (building rights)*，不是更好吗？

![](/docs/wiki/research/img/ET_2worlds.webp)

**APS 市场结构的分配机制**：

一个允许买方购买提议执行负载权利的无许可市场。
这些权利赋予持票人（可以是与验证者不同的第三方）随机分配的权利，以提议执行负载。

!(/docs/wiki/research/img/ETvsPBS.webp)

**区块拍卖 ePBS (block-auction ePBS) vs 时隙拍卖 ePBS (slot-auction ePBS)**

!(/docs/wiki/research/img/ba-ePBS.webp)
!(/docs/wiki/research/img/sa-ePBS.webp)

时隙拍卖 ePBS (sa-ePBS) 带来的改进，然而也伴随着处理双签和链头分裂视图的技术成本，而这[并不是一个容易解决的琐碎问题](#未决问题-open-issues)。

[执行票 (ET)](#执行票-execution-tickets-et-1) 和 [APS-Burn](/wiki/research/APS/aps.md) 是实现协议内见证者-提议者分离的两种可能的分配机制。信标提议者 (Beacon proposers) 可以仅对执行提议者做出承诺，因此不会对执行负载的内容做出承诺，这解决了区块拍卖 ePBS 版本面临的 时间游戏 (timing games) 问题。

## 执行票 (Execution Tickets, ET)

执行票 (ET) 与 ePBS 是正交的，从某种意义上说，它提出了一种新的范式：
验证者/见证者-提议者分离，其中提议者由协议招聘，并可以再次将构建角色委托给另一个实体，即构建者。
然而，递交信标区块仍将是验证者的特权。

ET 允许执行提议者从封装的无许可市场中购买“票 (tickets)”，在未来的某个不确定时间赎回执行提议权。

执行提议者从 ET 市场购买提议负载的权利，该市场在撰写本文时是一个抽象模块，[仍处于研发阶段](#未决问题-open-issues)。

### ET 如何遏制时间游戏？ (How do ET contain Timing games?)

当提议者尽可能延迟提议其区块以最大化其能够获得的价值时，就会发生 时间游戏 (timing games)。在区块拍卖 ePBS 中，由于执行负载是在信标区块发布时确定的，时间游戏将由验证者玩，他们可能会试图尽可能延迟递交信标区块，以便对尽可能高的竞价做出承诺。

ET 提出了比 ePBS 更进一步的共识-执行分离 (consensus-execution separation)，验证者作为提议者不对执行负载做出承诺，从而将时间游戏限制在执行提议阶段，这样共识关键功能就不会受到验证者理性行事动机的影响：

!(/docs/wiki/research/img/ET.webp)

## ET 高层级视图 (ET high-level view)

执行票机制可以简要地描述为 [[4]](#资源-resources) ：

* 一个用于买卖票证的协议内市场。
票证赋予所有者未来的区块提议权。票证将是一次性使用的，且仅在随机分配的时隙中有效。

* 用于选择信标区块提议者和见证者的抽奖。这是现有的权益证明 (Proof-of-Stake) 随机选择，验证者被随机选中以提议信标区块。在执行票下，信标区块将不再包含执行负载（区块的最终已执行交易列表），而是包含一个 包含列表 (inclusion list)，指定了一组必须存在于执行区块中的交易。

* 用于选择执行区块提议者和见证者的抽奖：确定中奖票证的第二次随机选择。票证所有者有权提议该时隙的执行区块并将交易包含在链上。然后状态被更新。执行区块提议者为他们拥有的每张票提供抵押品 (collateral)，以保证他们将在其票证分配时隙的执行轮次中生产单个执行区块。如果他们双签 (equivocate) 或离线，保证金 (bond) 将被追溯性 惩罚 (slashed)。

## ET 流程： (ET flow)

一个时隙的执行票流程为 [[4]](#资源-resources)：

1. 在信标轮 (beacon round) 期间，随机选出的信标提议者有权提议一个信标区块。
2. 该提议者提议包含包含列表的信标区块。
3. 信标见证者 (beacon attesters) 对信标区块的有效性和时效性进行投票。
4. 在执行轮 (execution round) 期间，一张随机选出的执行票有权提议一个执行区块。
5. 票证所有者是执行提议者 (execution proposer) 并提议一个执行区块。
6. 执行见证者 (execution attesters) 对执行区块的时效性和有效性进行投票。

!(/docs/wiki/research/img/ETflow.jpeg)

## 对 ET 所体现的 APS 市场结构的反对意见 (Counterarguments to the APS market structure that ET embodies)

- **C1**：从抗审查性的角度来看，我们可能希望让验证者在执行负载的构建中保持活跃。
  - *对反对意见 C1 的反驳*：
    **CC1**：今天，当验证者使用 MEV-Boost 时，他们就完全放弃了这种能力，而在使用 ePBS 时，他们也极有可能完全放弃这种能力。诸如 包含列表 [[6]](#资源-resources) 和 多元化小部件 [[7]](#资源-resources) 这样的改进提案，可以通过在拥有执行票的任何人身上施加绑定但非效率降低的负载构建限制来补救这一点。

- **C2**：ET 将不再保证验证者在他们认为公平时能够选择构建执行负载。
  - *对反对意见 C2 的反驳*：
    **CC2**：验证者之间对公平的定义可能非常不同，而且市场在实践中已经表明，大多数验证者倾向于自己不行使这种构建权，而是将构建委托给其他方。

- **C3**：将验证者和提议者角色分离，创造了一种持票人可能成为 *主动垄断者 (active monopolist)* 的市场结构，从而强加 [垄断定价 (monopoly pricing)](https://arxiv.org/abs/2311.12731) 并可能抢跑时间敏感的资金流。这与 MEV-Boost 下的状态截然不同，后者的特征是验证者作为提议者角色中的 *被动垄断者 (passive monopolist)* [[5]](#资源-resources)。
  - *对反对意见 C3 的反驳*：
    **CC3**：在他们的著作 *何时出售您的区块* [[5]](#资源-resources) 中，Quintus 和 Conor 指出，如果验证者在长期内趋于理性，那么验证者作为提议者和持票人作为提议者之间的差异可能会大大缩小，这意味着验证者-提议者角色分离的改进可能来自该分离本身可以带给 ET 以及随后带给协议的好处，而且这种角色分离极有可能不会造成比现状更糟糕的局面。

## 未决问题 (Open issues)

执行票正处于活跃的研究阶段，已知的一些未决问题包括：

* ET 市场的具体设计以及机票定价机制背后的机制会是什么？
* 执行票对分叉选择 (fork-choice) 有什么影响？
  * 在同一时隙内的分叉选择中可能存在多个有效的条目
  * 负载双签会导致链头分裂视图
* 执行票如何改变预确认 (preconfirmations) 的设计？
* 我们如何为执行票持有人构建“第二次”质押机制？

## 资源 (Resources)

[[1] 关于 ePBS 的 Wiki 页面](/docs/wiki/research/PBS/ePBS.md)

[[2] Barnabé 关于 PBS 的笔记](https://barnabe.substack.com/p/pbs)

[[3] Barnabé 重新审视 PBS 市场结构的文章](https://mirror.xyz/barnabe.eth/LJUb_TpANS0VWi3TOwGx_fgomBvqPaQ39anVj3mnCOg#heading-the-pains-of-being-proposer-as-validator)

[[4] 关于执行票的 ethresearch 帖子](https://ethresear.ch/t/execution-tickets/17944)

[[5] 何时出售您的区块 flashbots 帖子](https://collective.flashbots.net/t/when-to-sell-your-blocks/2814)

[[6] EIP-7547 包含列表](https://eips.ethereum.org/EIPS/eip-7547)

[[7] ROP-9: 旨在抗审查的多元化小部件](https://efdn.notion.site/ROP-9-Multiplicity-gadgets-for-censorship-resistance-7def9d354f8a4ed5a0722f4eb04ca73b)

[[8] PEPC 常见问题解答](https://efdn.notion.site/PEPC-FAQ-0787ba2f77e14efba771ff2d903d67e4#a2d2d17abe90414e88d667ad10d91afe)

[[9] Potuz 的 ePBS 规范笔记](https://hackmd.io/@potuz/rJ9GCnT1C)