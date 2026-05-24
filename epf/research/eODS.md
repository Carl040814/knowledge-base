# 协议内嵌的运营商-委托者分离 (Enshrined Operator-delegator separation, eODS)

> [!WARNING]
> 本文档涵盖了一个活跃的研究领域，在阅读时可能已过时并会在未来进行更新，因为围绕验证者角色 (Validator roles) 解耦的设计空间正在不断演变。

## 路线图追踪 (Roadmap tracker)

| 升级 (Upgrade) | 阶段 (URGE) | 轨道 (Track) | 条目 (Item) | 交叉引用 (Cross-references) |
|:-------:|:-----------:|:-----------------:|:-------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------:|
|  eODS   | the Scourge (洗劫) | 质押经济学 (Staking economics) | 抗审查性 (Censorship resistances)、流动性质押去中心化 (liquid staking decentralization) | [SSF](/docs/wiki/research/SSF.md), [IL](/docs/wiki/research/cl-upgrades.md), [ePBS](/wiki/research/PBS/ePBS.md), [ET](/wiki/research/PBS/ET.md) |

## 相关背景 (Relevant context)

流动性质押 (liquid staking) 的委托人-代理人问题 (Principal–Agent problem)（即代理人 (Agent) 的利益与委托人 (Principal) 的利益不一致）是任何资本委托的固有部分，在当今的质押生态系统 (staking ecosystem) 中更是如此[^1]。

自信标链 (Beacon Chain) 早期开始，在以太坊中就已经出现了允许在不运行实际验证者软件 (validator software) 的情况下为质押池 (staking pools) 提供流动性的市场结构。
因此，质押在协议层之外 (outside protocol level) 自然地分裂为两类参与者[^2]：

| 层级 (Tier) | 当前的自然分离 (Current natural separation) | 罚没风险 (Slashing risk) |
|:----------:|:----------------------------------------------------------------------------------------------------------------------------------------:|:-------------:|
| 委托者 (Delegators) | 以太坊质押者 (ETH stakers)，没有最低资金承诺，除了带来本金 (principal) 之外，没有以任何其他方式参与的严格要求 | 可被罚没 (Slashable) |
| 运营商 (Operators) | 节点运营商 (Node operators)，包括家庭质押 (home staking) 或提供验证者服务 (validator services)，其声誉或自身固定数额的资本面临风险 | 可被罚没 (Slashable) |

这两个层级密切相关，因为如果流动性质押代币 (Liquid Staking Token, LST) 的持有者不相信持有其本金 (principal) 的运营商是*优秀的代理人 (good agents)*，那么流动性质押协议 (liquid staking protocols) 就无法获得信任。

罚没 (slashing) 的风险，加上通过增发 (issuance) 向 Gasper 服务提供商 (Gasper service providers) 支付的总额巨大的质押收益，导致了**委托人-代理人关系 (principal-agent relationships)**的中间链条的产生[^3]。委托者 (Delegators) 向代表他们参与 Gasper 的运营商 (Operators) 提供资金（委托人, Principal），以便为委托人 (Principal) 和他们自己赚取质押奖励 (staking rewards)。
然而，如果验证者 (validator) 表现不佳，委托者 (delegator) 的资金也会被罚没 (slashed)，因为验证者（代理人, Agent）的行为会影响委托者（委托人, Principal）的资产。

为了被视为优秀的代理人 (good agents)，诚实的运营商 (Operators) 必须致力于委托人 (Principal) 的利益：
- 妥善履行验证职责 (validation duties)（即运行和维护良好的质押基础设施，努力保持良好的在线声誉以表现出高参与率 (participation rate)，并运行规定的协议，因此绝不通过签署冲突区块进行双签 (equivocate)）
- 代理人 (agents) 绝不能具有对抗性（恶意），任意偏离协议，例如为了获得被低估的质押份额以攻击网络而进行双签并被罚没 (slashed)。

## 什么是 eODS？ (What is eODS?)

eODS 是以太坊内部活跃讨论的设计空间[^4]，建议在协议层 (protocol level) 实现**验证者 (Validator)**角色的进一步分离（解耦, unbundling），从而在“洗劫 (the Scourge)”的质押经济学轨道中完成协议内嵌提议者-构建者分离 (ePBS) 与执行票 (Execution tickets) 之间的闭环。

### 解耦验证者角色 (Unbundling the Validator role)

| 升级 (Upgrade) | 分离类型 (Separation type) |
| ------- | ------------------ |
| ePBS    | 提议者-构建者分离 (proposer-builder) |
| ET      | 验证者-提议者分离 (validator-proposer) |
| eODS    | 运营商-委托者分离 (operator-delegator) |

这种分离解决了在以太坊 (ETH) 质押背景下，与协议所见范围的局限性[^5]以及其利用自动防御系统做出反应的能力相关的各种低效问题。

## 走向单时隙最终性的双层质押方法 (The two-tier staking approach to SSF)

在 [单时隙最终性 (SSF)](/docs/wiki/research/SSF.md) 的研究讨论中，与每个时隙的 BLS 签名聚合 (BLS signatures aggregation) 相关的技术限制产生了以下范式：
> 我们需要摆脱每个参与者在每个时隙都签名的概念 —— Vitalik Buterin

在上述背景下，将**质押拆分**为两层参与者的提议具有重要价值：[^23]
  * **高复杂度层级 (high-complexity tier)**：被称为提供**重节点服务 (Heavy node services)**，每个时隙都需要参与，但只有大约 10,000 个参与者，以及
  * **低复杂度层级 (lower-complexity tier)**：提供**小（轻）节点服务 (Small(light) node services)**，仅偶尔被召集参与，具有较低的计算开销、硬件或技术知识要求。

重节点服务的提供者将面临罚没，但参与协议的最终性组件 (Finality gadget) 也会获得高额回报；而小节点服务的提供者将享有较低的奖励，但可以完全免于罚没（不积极参与每个时隙），或者可以选择在短期内（如几个时隙内）接受罚没约束。

## 委托者的角色 (The role of Delegators)

Vitalik 在其 2023 年 10 月的论文《可能改善去中心化并减少共识开销的协议与质押池改变》中提出了一个关键问题：**从协议的角度来看，设立委托者 (delegators) 的意义究竟何在？**[^6]

Vitalik 进行了一个思想实验，假设将最大罚没惩罚限制在 2 ETH 成为现实[^7]。

在此前提下运行并对比了两种场景：
1. 因为罚没 (slashing) 和非活跃惩罚 (leaking penalties) 被封顶，Rocket Pool 相应地将运营商保证金 (operator bond) 降至 2 ETH，所有 ETH 都被质押，Rocket Pool 作为流动性质押协议 (liquid staking protocol, LSP) 吸收了 100% 的市场份额（不仅在质押者中，在 ETH 持有者中也是如此。随着 rETH 变得无风险，几乎所有的 ETH 持有者都成为了 rETH 持有者或节点运营商）。
2. 在第二种场景中，Rocket Pool 作为 LSP 并不存在。最低存款额降至 2 ETH，总质押 ETH 限制在 625 万。同时，节点运营商的收益率降至 1%。

这两种场景分别从质押经济学和攻击成本的角度进行了运行。
在计算完毕后，两种情况的最终结果完全相同，因此，理论上协议若能直接去除中间商、大幅降低质押奖励并将总质押 ETH 限制在 625 万，其表现会更好。

上述论点的目的并不是主张降低质押奖励，也不是为了限制总质押 ETH，而是指出一个运行良好的质押系统应该具备的关键属性：

>**委托者应该做一些真正重要的事情** —— *Vitalik Buterin*

### eODS 下委托者的角色 (Delegators role under eODS)

如果委托者 (Delegators) 要拥有**有意义的角色**，那会是什么？协议又该如何**激励**该角色的选择？

在运营商-委托者分离 (Operator-Delegator separation) 架构下，有两种可能的解决方案[^8]：
1. **运营商集合的策展 (curation of operator set)**：有主见的委托者可以决定根据费用或可靠性等因素在不同的运营商 (operators) 之间做出选择。
2. **轻服务的提供 (provision of light services)**：委托者可能会被要求提供非惩罚性（非罚没）但至关重要的服务，例如：
    - 将他们的观点输入到抗审查组件 (censorship-resistance gadgets) 中，如纳入列表 (inclusion lists) 或多元性组件 (multiplicity gadgets)。
    - 对他们所看到的当前规范链头进行签名，作为 Gasper 运营商质押的替代信号。

**激励委托者角色 (Incentivizing the Delegator role)**：

在此模型下，委托者 (Delegators) 不会对 FFG 的经济安全性 (economic security) 做出贡献（即委托者不参与最终性 (finality)（非罚没质押）），但他们能够揭示组件运行中的差异。他们的服务可以通过重新分配聚合发行量 (issuance) 来获得补偿。

## The layers of Operator-Delegator Separation

如果单独来看，eODS 确立了这种分离，但并未给现有的市场结构带来改善：

| 一维-eODS (1D-eODS) |           |
| --------- | --------- |
| 运营商 (OPERATOR)  | 委托者 (DELEGATOR) |
| 可被罚没 (slashable) | 可被罚没 (slashable) |

为了使 eODS 具有现实意义，必须将其与其它相关的、提议的协议更改进行整体考量：

* **惩罚上限 (Capping penalties)**：
    | 1D-eODS + 惩罚上限 |               |
    | --------------------------- | ------------- |
    | 运营商 (OPERATOR)                    | 委托者 (DELEGATOR)     |
    | 可被罚没 (slashable)                   | 免于罚没 (non-slashable) |

    在此模型中，通过将罚没 (slashing) 和非活跃惩罚 (leaking penalties) 限制在仅针对运营商的质押，委托者 (delegators) 的资产不再面临风险。然而，Barnabé 认为[^4]在 1D-eODS 下，委托者的角色并不是很清晰，原因如下：
    - 在双层质押模型中的委托者与当前承担罚没风险的 LSP 委托者不同。
    - 某些代理人会希望将资产委托给“双层运营商”并使自己处于罚没约束下，以寻求更高收益。
    - 某些代理人自己不想运行小节点服务 (small node services)，但希望通过委托运营来参与轻服务的提供。

    此外，在最小可行发行量 (Minimum Viable Issuance, MVI) 的背景下，以此种方式实施惩罚罚没，在经济上等同于将质押收益降至 1%，并使质押明确成为一种利他活动[^9]。

* 上述问题可以通过**引入两种不同类型的协议服务**来解决，这基于[走向 SSF 的双层质押方法](#the-two-tier-staking-approach-to-ssf)：
    - ***重服务 (Heavy Services)***
    - ***轻服务 (Light Services)***

    这种*设计哲学*在 2 个维度上对验证者 (Validator) 的角色进行了分离（二维运营商-委托者分离, 2D Operator-Delegator Separation），每个维度自身都诱导出了委托者和运营商的市场结构：

    | 二维-eODS (2D-eODS / 彩虹质押, rainbow staking) |                 |               |
    | ------------------------- | --------------- | ------------- |
    | 轻度运营商 (Light OPERATOR)            | 轻度委托者 (Light DELEGATOR) | 免于罚没 (non-slashable) |
    | 重度运营商 (Heavy OPERATOR)            | 重度委托者 (Heavy DELEGATOR) | 可被罚没 (slashable)     |

    ### 轻服务的经济学 (Economics of light services)
    轻服务 (Light services) 使用质押资产作为*女巫控制 (sybil-control)*机制和*权重函数 (weight functions)*[^10]。

    协议将提供轻服务生态系统（例如抗审查组件），这些服务具有较低的硬件和经济要求。
    在协议组件中，抗审查机制通过纳入列表 (inclusion lists) 等机制来奖励那些帮助协议“看到”被审查输入内容的参与者。

    这些轻服务通过重新分配用于其提供的聚合发行量 (issuance) 来获得补偿，这种模式在今天的同步委员会 (sync committees) 中已经得到应用[^11]。

    轻度资产持有者（委托者）可以将资产委托给轻度服务运营商，由运营商代表委托者执行服务并收取费用，这使运营商的激励与最大化委托者的奖励相一致。在一个充满竞争的轻度运营商市场中，再加上允许从表现不佳的运营商处立即退出的重新委托机制，轻度运营商有望以微薄的利润率和高成本效率来提供服务。

    **部分可罚没的轻服务 (Partially slashable light services)**

    某些轻服务可能需要可罚没的质押。可以实施罚没上限方法的一种变体，其中仅有运营商的质押是可被罚没的。

    | 2D-eODS (彩虹质押) | +   惩罚上限 (penalties capping)         |
    | ------------------------- | ----------------------------- |
    | 轻度运营商 (Light OPERATOR) 可被罚没 | 轻度委托者 (Light DELEGATOR) 免于罚没 |
    | 重度运营商 (Heavy OPERATOR) 可被罚没  | 重度委托者 (Heavy DELEGATOR) 可被罚没    |

    ### 重服务的经济学 (Economics of heavy services)
    重服务 (Heavy services) 使用质押资产作为*经济安全性 (economic security)*[^12]。

    如果他们的质押参与了 FFG 安全故障（冲突的已最终化检查点），他们所有的质押都将被损失。

    重服务（例如以太坊的最终性组件 FFG）的要求将被强化，以实现单时隙最终性 (Single-Slot Finality, SSF)。

    有助于培育安全质押环境的潜在协议内置组件：
    - 流动性质押模块（LSM）风格的原语，用于在其之上构建流动性质押协议
    - 协议内置的部分资金池 (enshrined partial pools) 或分布式验证者技术 (DVT) 网络

    所有这些都允许快速重新委托以及其他特性。

    ### 机制设计：重服务 vs. 轻服务 (Mechanism design (Heavy vs. Light))
    |                               | 重服务 (Heavy services)                                                               | 轻服务 (Light services)                                                             |
    | ----------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
    | 服务雏形 (Service archetype)             | Gasper                                                                       | 抗审查组件 (Censorship-resistance gadgets)                                              |
    | 奖励动态 (Reward dynamics)               | 关联性通常会产生奖励，而在故障期间抗关联性是件好事 | 抗关联性产生奖励（呈现不同的信号） |
    | 罚没风险 (Slashing risk)                 | 运营商和委托者 | 无或仅针对运营商 |
    | 运营商角色 (Role of operators)             | 运行全节点以提供 Gasper 验证服务 | 运行小节点以提供轻服务 |
    | 委托者角色 (Role of delegates)             | 为 Gasper 贡献经济安全性 | 向服务质量良好的轻度运营商赋予权重 |
    | 运营商资金需求 (Operator capital requirements) | 高资本效率（高单运营商质押量）+ 高额资本投资 | 并非主要限制（运营商接收权重）+ 小节点固定成本 |
    | 独行质押者接入 (Solo staker access)            | 主要作为 LSP 的一部分（例如作为 DVT 节点） | 对所有轻服务均有很高的接入性 |

## 释放再质押？ (Unlocking re-staking?)

eODS（二维模型）可以被视为再质押 (re-staking) 的部分协议内嵌 (partial enshrinement)，这要归功于再质押者能够致力于这种新型协议主动验证服务 (Actively Validated Service, AVS)[^13]，其奖励来自新铸造的 ETH 发行（即 Gasper 是一种具有众所周知的发行时间表的协议 AVS）[^14]。

由此，ETH 持有者被允许直接作为运营商，或间接作为委托者参与到这些 AVS 的提供中。

## 独行质押者在这一新世界中处于何种位置？ (Where do Solo Stakers fit in this new world?)

Gasper（以及单时隙最终性 (SSF) 场景下的变体）是以太坊协议中最重的 AVS。它获得了最大份额的聚合发行，因此在委托质押计划中，许多方面都有兴趣提供所需的质押[^3]，委托者将质押提供给代表他们参与 Gasper 的运营商。

鉴于质押的中间链条的形成，有必要防止出现由大部分 ETH 供应提供担保的单一主导流动性质押提供商。为了针对协议提供足够的安全性，最小可行发行量 (Minimum Viable Issuance, MVI)[^15] 措施至关重要，它能产生足够的压力，将 Gasper 的经济权重保持在合理的比例。

由于独行质押者 (solo stakers) 无法从其抵押品中发行可信的流动性质押代币，因此其资本效率较低，MVI 强加的竞争压力并不非常适合他们。在 eODS 的背景下，这一劣势可以得到极大的改善。

### eODS 下独行质押者的角色 (The role of solo stakers, under eODS)

在其“解耦质押：走向彩虹质押”的研究帖子[^16]中，Barnabé 提出了独行质押者所理想体现的两个核心价值主张：

* **支持网络韧性 (Bolster network resilience)**：独行质押者可以增强网络应对大型运营商故障的韧性，例如在大型运营商下线时推进（动态可用的）链。由于资本和成本效率的限制，这不会是他们的主要运营线路，但在最坏的情况下，这可能是一个强有力的后备选择。

* **偏好熵的产生者 (Generators of preference entropy)**：独行质押者可以作为抗审查代理人做出贡献，使协议能够看到更多并服务于更广泛的用户。对于广泛的低算力参与者来说，执行此类轻服务是触手起及的。多元性组件 (Multiplicity gadgets) 可以奖励那些增加偏好熵 (preference entropy) 的运营商的贡献[^16]。

偏好熵 (Preference entropy) 表示协议代理人向协议显露的信息。进行审查的代理人具有较低的偏好熵，因为他们决定限制某些偏好的表达，例如可能违反其自身管辖偏好的活动。
运行节点以提供服务的独行质押者群体是高度去中心化的，因此能够表达高偏好熵。这种经济价值会转化为该群体成员的收益。

## 为什么要分离？ (Why separate?)

### 社会层杠杆 (Social layer leverage)
在对抗质押领域的中心化和防止出现主导的 LST 及其相关危害方面，过度依赖社会层和道德来保护协议是存在风险的。

### 已是既定的模式 (Already an established model)
eODS 意味着在协议中内嵌一种已经确立的、包含两类参与者（委托者/质押者和节点运营商）的质押模式变体。

### 为委托者赋能更强大的共识参与形式 (Enabling stronger forms of consensus participation for Delegators)
为了提高委托者的选择权[^17], 我们可以：

* **改善质押池内的投票工具**：
  在当前范式下，质押池内的投票仅限于治理代币持有者（而非 ETH 持有者）。目前有一些乐观治理 (Optimistic governance) 的尝试，即 ETH 持有者可以否决 LSP 的治理投票，但（转述 Vitalik 的话）代币投票还不够强大，归根结底，任何形式的无激励委托者选择都只是代币投票的一种形式。

* **改善质押池之间的竞争**：
  小型流动性质押协议面临的挑战是，它们的市值较小、流动性较差、较难获得信任，且较少受到应用的支持。限制惩罚和 1D-eODS 理论上可以帮助解决这些挑战，但在实际中，由于[上述原因](#the-layers-of-operator-delegator-separation)，限制惩罚的实施是不可行的。

* **内嵌委托 (enshrine delegation)**：
  eODS 为委托者提供了更强大的共识参与形式，无论是：
    - 作为资本提供者，向代表他们参与链最终性 (finality) 的运营商提供资金（重服务，可被罚没），或者
    - 作为抗审查组件（如纳入列表 (Inclusion lists) 或多元性组件 (Multiplicity gadgets)）的参与者（轻服务，免于罚没），或者
    - 作为替代最终性参与者的信号进行签名（轻服务，免于罚没）。

### 减少 BLS 签名的数量（SSF 场景） (Reduce the number of BLS signatures (SSF scenario))
以太坊在成为理想的全球规模网络的道路上不断改进和成长。

在这种情况下，单时隙最终性 (SSF) 不仅是令人向往的，而且很可能是强制性的，在走向 SSF 的道路上需要做好权衡。

在上述背景下，我们可以假设每个时隙可以处理的 BLS 签名限制在 10 万到 180 万之间。
eODS 提出了数量更少的验证者作为重服务提供商（小于 10,000 个），从而即使在单时隙最终性的假设下，也能减少每个时隙需要处理的 BLS 签名数量。

### 与活跃的研发空间的协同作用 (Synergies with active R&D space)

**执行票 (Execution tickets)**

eODS 在一个验证者角色[已被消除歧义](#unbundling-the-validator-role)的世界中非常契合。执行票 (Execution Tickets)[^18] 的本质是将执行与共识分离，这有助于实现重度运营商的最小可行发行量 (MVI)[^15]。如果执行服务在 ET（或类似的证明者-提议者分离）下与共识服务分离，重度运营商在最终确定执行服务输出（有效载荷, payload）时，将较少受到奖励可变性和参与时机博弈 (Timing games)[^19] 的困扰。

**纳入列表与多元性组件 (Inclusion lists & Multiplicity gadgets)**

轻服务通过将有效载荷服务纳入纳入列表 (IL) 或多元性组件 (MG) 等抗审查组件中，对其构成约束。

由诚实验证者准备的纳入列表 (Inclusion lists)[^20] 可作为审查信号，并防御来自可识别验证者的系统性审查。证明者通过分叉选择来维护纳入列表的有效性和满意度。

多元性组件 (Multiplicity gadgets)[^21] 与 IL 相关，但它提议将构建纳入集的责任分配给一个委员会，而不是单个领导者（如 IL），从而避免了对单个参与者的诚实性或理性程度的依赖。在要纳入的项目集上获得共识也可以增加问责制并允许激励方案。

虽然轻服务对执行有效载荷生产者的确切约束机制还需要进一步研究和开发，但根据 eODS 将*生产者*与*执行者*分离，将对当今的 IL 设计带来改进，在当今设计中验证者同时充当生产者和执行者。

通过分离服务，轻度运营商成为列表的生产者，但执行有效载荷相对于 Gasper 的有效性由重度运营商执行，重度运营商证明并最终确定链的有效历史，忽略无效的有效载荷。

**最大有效余额 (MAX_EFFECTIVE_BALANCE)**

EIP-7251[^22] 将允许单个消息携带更多质押。内嵌运营商与委托者分离将为协议提供一种在功能上区分此类消息上“运营商质押”与“委托者质押”的方法，从而进一步在重服务提供商和轻服务提供商中分离每个层级。

### 未来的协议服务 (Future protocol services)
eODS 提供了一个接口，以即插即用的方式集成进一步的“协议服务”。

## 前方的道路 (The road ahead)

从高层视角来看，eODS 的两个维度可以按如下方式实施：
- **分离的垂直维度**（即轻服务 - 重服务的分离）在实践中可以通过提高 MAX_EFFECTIVE_BALANCE[^22]，随后实施例如 2048 ETH 的余额阈值来确定哪些验证者进入哪个复杂度层级。
- **分离的水平维度**（即运营商-委托者分离）可以通过[本节](#the-layers-of-operator-delegator-separation)中介绍的方法来实现。

对于每种（重度和轻度）类型的协议服务需要靶向的具体 MVI，以及 eODS 的实际应用和实现，还需要进一步的研发。

关于流动性质押的协议内嵌，Vitalik 指出：
> 直接内嵌某些东西可能仍然更好，但值得注意的是，这种“内嵌某些东西，把其他东西留给用户”的设计空间确实存在。[^7]

## [参考文献] (References)

[^1]: https://eprint.iacr.org/2023/605

[^2]: https://notes.ethereum.org/@vbuterin/staking_2023_10#Protocol-and-staking-pool-changes-that-could-improve-decentralization-and-reduce-consensus-overhead

[^3]: https://mirror.xyz/barnabe.eth/v7W2CsSVYW6I_9bbHFDqvqShQ6gTX3weAtwkaVAzAL4

[^4]: https://ethresear.ch/t/unbundling-staking-towards-rainbow-staking/18683#operatordelegator-separation-2

[^5]: https://barnabe.substack.com/i/95811604/case-studies-in-upgrading-the-fence

[^6]: https://notes.ethereum.org/@vbuterin/staking_2023_10#The-role-of-delegators

[^7]: https://vitalik.eth.limo/general/2023/09/30/enshrinement.html#enshrining-liquid-staking

[^8]: https://notes.ethereum.org/@vbuterin/staking_2023_10#If-delegators-can-have-a-meaningful-role-what-might-that-role-be

[^9]: https://youtu.be/ZBvV88jEkiA?t=2561

[^10]: https://ethresear.ch/t/unbundling-staking-towards-rainbow-staking/18683#economics-of-light-services-5

[^11]: https://eth2book.info/capella/part2/incentives/rewards/#proposer-rewards-for-sync-committees

[^12]: https://ethresear.ch/t/unbundling-staking-towards-rainbow-staking/18683#economics-of-heavy-services-4

[^13]: https://docs.eigenlayer.xyz/eigenlayer/avs-guides/avs-developer-guide#what-is-an-avs

[^14]: https://eth2book.info/capella/part2/incentives/issuance/

[^15]: https://notes.ethereum.org/@anderselowsson/MinimumViableIssuance

[^16]: https://ethresear.ch/t/unbundling-staking-towards-rainbow-staking/18683#staking-economics-in-the-rainbow-world-6

[^17]: https://notes.ethereum.org/@vbuterin/staking_2023_10#Expanding-delegate-selection-powers

[^18]: https://ethresear.ch/t/execution-tickets/17944

[^19]: https://ethresear.ch/t/timing-games-implications-and-possible-mitigations/17612

[^20]: https://eips.ethereum.org/EIPS/eip-7547

[^21]: https://efdn.notion.site/ROP-9-Multiplicity-gadgets-for-censorship-resistance-7def9d354f8a4ed5a0722f4eb04ca73b

[^22]: https://eips.ethereum.org/EIPS/eip-7251

[^23]: https://ethresear.ch/t/sticking-to-8192-signatures-per-slot-post-ssf-how-and-why/17989#approach-2-two-tiered-staking-4
