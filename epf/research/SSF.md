# 单时隙最终性 (Single Slot Finality, SSF)

## 路线图跟踪器 (Roadmap tracker)

| 升级 (Upgrade) | 紧迫性 (URGE) | 轨道 (Track) | 主题 (Topic) | 交叉引用 (Cross-references) |
| :-----: | :-------: | :---: | :----------------------: | :----------------------------------------------------------------------------------------------------------: |
| 单时隙最终性 (Single Slot Finality, SSF) | 合并 (the Merge) | - | 权益证明 (Proof of Stake, PoS) 升级、最终性 (finality) 和安全 (security) | 与以下内容交叉：[最大有效余额 (Maximum Effective Balance, MAX_EB)](/docs/wiki/research/cl-upgrades.md), [集成式提议者-构建者分离 (Enshrined Proposer-Builder Separation, ePBS)](/docs/wiki/research/PBS/ePBS.md) |

[单时隙最终性 (Single Slot Finality)](/docs/eps/week10-research.md) 是信标链 (Beacon Chain) 共识机制 (consensus mechanism) 的一项研究性改进概念，旨在解决区块最终确认所需时间所导致的高效性问题。它提出要显著提高区块验证效率 (blocks validation efficiency) 并大幅缩短达到最终性 (time-to-finality) 的时间。
在单时隙最终性下，无需等待 2 个纪元 (epochs)，区块就可以在同一个时隙 (slot) 内被提议 (proposed) 并最终确认 (finalized)。该主题在 [EPS 第 10 周 (EPS week 10)](https://github.com/eth-protocol-fellows/protocol-studies/blob/ssf/docs/eps/week10-research.md) 中也有所涵盖。

## 背景与动机 (Context and Motivation)
以太坊 (Ethereum) 共识层 (consensus layer) 实现了 Gasper 协议 (Gasper protocol)，其中包含 Casper 友好最终性小工具 (Casper Friendly Finality Gadget, Casper FFG)。Casper FFG 确保网络在每个纪元 (epoch) 持续产出区块 (blocks) 并累积验证者关注度 (validator attentions)。最终性 (Finality) 是权益证明 (Proof of Stake, PoS) 经济安全性 (economic security) 的终极状态，其改变需要 2/3 的验证者集合 (validator set) 被罚没 (slashed)。
在撰写本文时，信标链 (Beacon Chain) 每 2 个纪元 (epochs) 或每 64 个时隙 (slots) 实现一次最终确认 (finality)。每个时隙 (slot) 长度为 12 秒，因此最终确认大约需要 12.8 分钟。
对于大多数用户而言，目前的最终性耗时 (time to finality) 显得过长，并且对于那些不希望等待这么长时间来确认交易 (transactions) 永久性 (permanent) 的应用 (apps) 和交易所 (exchanges) 来说也很不方便。
区块提议 (proposal) 与最终确认 (finalization) 之间的延迟，也为短暂的重组 (reorganizations, reorgs) 创造了机会，攻击者 (attacker) 可以借此审查 (censor) 某些区块 (blocks) 或提取最大可提取价值 (Maximal Extractable Value, MEV)。

## 单时隙最终性 (SSF) 的优势
* 对应用更方便 - 交易 (transactions) 最终确认时间提升了数量级，即 12 秒而不是 12 分钟，这为所有以太坊 (Ethereum) 用户带来了更好的用户体验 (User Experience, UX)。
* 攻击难度大幅提升 - 多区块最大可提取价值重组 (multi block MEV re-orgs) 得以消除，因为链只需要 1 个区块即可最终确认，而不再是 64 个区块。
* 未来的共识机制（在单时隙最终性场景下）与目前 LMD-GHOST 和 Casper-FFG 的组合相比，复杂性将大幅度降低，而目前的组合可能会导致某些攻击（如平衡攻击 (balancing attacks)、扣留与攒存攻击 (withholding and savings attacks)）。

## 单时隙最终性 (SSF) 中的分叉选择规则 (fork-choice rule)
当今的共识机制依赖于 Casper-FFG（决定 2/3 的验证者是否对某条链进行了见证的算法）与分叉选择规则 (fork-choice rule)（LMD-GHOST，即在存在多个选项时决定哪条链是正确链的算法）之间的耦合[^1]。
分叉选择算法仅考虑自上一个最终确认区块 (finalized block) 以来的区块。在单时隙最终性 (SSF) 下，将不会有任何区块供分叉选择规则去考虑，因为最终确认 (finality) 发生在区块被提议的同一个时隙 (slot) 中。这意味着在单时隙最终性下，**要么**分叉选择算法活跃，**要么**最终性小工具 (finality gadget) 活跃。
最终性小工具将最终确认那些有 2/3 验证者在线 (online) 并且进行诚实验证 (attesting honestly) 的区块。如果区块无法超过 2/3 的阈值 (threshold)，则分叉选择规则将介入以确定遵循哪条链。这也为维持非活跃泄漏机制 (inactivity leak mechanism) 创造了机会，该机制可以恢复超过 1/3 验证者下线的链。如果区块无法超过 2/3 的阈值，则分叉选择规则将介入以确定遵循哪条链。

在任何此类设计中，分叉选择与共识之间的某些交互问题仍然存在，对其进行深入研究依然非常重要。对现有分叉选择的短期改进（例如视图合并 (view-merge)）也可能为单时隙最终性 (SSF) 分叉选择的工作提供参考。[^2]

## 实现单时隙最终性 (SSF) 需要解决的关键问题是什么？
Vitalik 概述的三个开放性问题[^4]：

* 具体的共识算法 (consensus algorithm) 将会是什么？

* 签名 (signatures) 的聚合策略 (aggregation strategy) 将会是什么？

* 验证者经济学 (validator economics) 的设计将会是什么？

## 最大有效余额 (MAX_EB) 和齐普夫 ETH 分布 (Zipfian ETH distribution)

## 参考文献 (References)

[^1]: 结合 GHOST 和 Casper https://arxiv.org/pdf/2003.03052.pdf, [[已存档]](https://arxiv.org/pdf/2003.03052.pdf)

[^2]: Ethereum.org 上的单时隙最终性 (SSF) 页面 https://ethereum.org/en/roadmap/single-slot-finality/#role-of-the-fork-choice-rule, [[已存档]](https://web.archive.org/web/20240309234119/https://ethereum.org/en/roadmap/single-slot-finality/#role-of-the-fork-choice-rule)

[^3]: EIP-7251：增加最大有效余额 (MAX_EFFECTIVE_BALANCE) https://eips.ethereum.org/EIPS/eip-7251, [[已存档]](https://web.archive.org/web/20240324072459/https://eips.ethereum.org/EIPS/eip-7251)

[^4]: VB 的单时隙最终性 (SSF) 笔记 https://notes.ethereum.org/@vbuterin/single_slot_finality, [[已存档]](https://web.archive.org/web/20240330010706/https://notes.ethereum.org/@vbuterin/single_slot_finality)

[^5]: 单时隙最终性 (SSF) 后每个时隙保持 8192 个签名 https://ethresear.ch/t/sticking-to-8192-signatures-per-slot-post-ssf-how-and-why/17989. [[已存档]](https://web.archive.org/web/20240105131126/https://ethresear.ch/t/sticking-to-8192-signatures-per-slot-post-ssf-how-and-why/17989)

[^6]: 一个简单的单时隙最终性 (Single Slot Finality) 协议 https://ethresear.ch/t/a-simple-single-slot-finality-protocol/14920, [[已存档]](https://web.archive.org/web/20231214080806/https://ethresear.ch/t/a-simple-single-slot-finality-protocol/14920)

[^7]: 单时隙最终性 (SSF) 笔记 Lincoln Murr https://publish.obsidian.md/single-slot-finality/Welcome+to+My+Research!,
[[已存档]](https://web.archive.org/save/https://publish.obsidian.md/single-slot-finality/Welcome+to+My+Research!)
