# 快速确认规则 (Fast Confirmation Rule, FCR)

# 概述 (Overview)

**快速确认规则 (Fast Confirmation Rule, FCR)** 是一种算法 (algorithm)，允许以太坊 (Ethereum) 节点 (nodes) 在良好的网络条件 (network conditions) 假设下，确定一个区块 (block) 是否**永远不会离开规范链 (canonical chain)**。

FCR 输出一个简单的决策：
- 区块已**确认 (confirmed)**
- 区块**未确认 (not confirmed)**

FCR 旨在提供**快速、安全的区块确认 (block confirmation)**，这远早于以太坊 (Ethereum) 的最终化 (finalization)，并有助于填补在[单时隙最终性 (Single Slot Finality, SSF)](https://ethereum.org/roadmap/single-slot-finality/) 实现之前的空白。

> 规范链 (canonical chain) 是诚实验证者 (honest validators) 所遵循的链。
---

# 动机 (Motivation)

## 当前的确认机制 (Current Confirmation Mechanism)

以太坊共识协议 (Ethereum protocol) 中目前唯一可用的**确认规则 (Confirmation Rule)**是 [Gasper](https://ethereum.org/developers/docs/consensus-mechanisms/pos/gasper/)，即 **FFG最终化规则 (FFG Finalization Rule)**。虽然这种确认规则 (Confirmation Rule) 极其安全，并且可以在**异步 (asynchronous)**网络条件 (network conditions) 下工作，但最终化 (finalization) 对许多使用场景来说**太慢了**。

- 最佳情况 (Best case)：约 12 分钟 (minutes)  
- 平均 (Average)：约 16 分钟 (minutes)  

一个已最终化 (finalized) 的区块：
- 永远不会与任何诚实验证者 (honest validator) 的规范链 (canonical chain) 冲突
- 只有在**超过 1/3 的验证者 (validators) 被罚没 (slashed)**时才能被回滚 (reverted)

### 用户体验限制 (UX Limitations)

- 购买日常用品（例如咖啡）需要等待约 16 分钟 (minutes)
- 充值 (Deposits) 到中心化交易所 (Centralized Exchanges, CEXs) 在最终化 (finalization) 之前是不可用的
- 钱包 (Wallets) 通常在交易 (transactions) 进入区块 (block) 时就立即将其标记为“已确认”，这**并不安全 (not safe)**
- ...

---

><details>
><summary><strong>如果将来有了 SSF，为什么还需要 FCR (Why FCR Exists if we will have SSF anyway)</strong></summary>
>
> 单时隙最终性 (Single Slot Finality, SSF)：
> - 依然距离我们很遥远
> - 只有在以下技术实现*之后*才会被考虑：
>   - 无状态以太坊 (Stateless Ethereum)
>   - Verkle 树 (Verkle trees)
>
> 在那之前，以太坊 (Ethereum) 缺乏一个**安全且快速的确认规则 (confirmation rule)**。
>
> 这就是为什么**中心化交易所 (Centralized Exchanges, CEXs)**和基础设施提供商 (infrastructure providers) 对 FCR 感兴趣的原因。
></details>

---

### 用户**不应该**使用的危险替代方案（启发式方法） (Dangerous alternatives (heuristics) which users should NOT use)

<details>
<summary><strong>依赖于区块深度 (Block Depth) 的启发式方法 (Heuristics that rely on Block Depth)</strong></summary>

- 在拥有 `N` 个后代 (descendants) 区块后确认区块 (blocks)

</details>
<details>
<summary><strong>依赖于基于合理化启发式 (Justification-Based Heuristics) 的方法 (Heuristics that rely on Justification-Based Heuristics)</strong></summary>

- 略好于区块深度 (block depth)
- 仍然缺乏关于确认 (confirmation) 的信息

</details>

这两种方法都不满足[安全属性 (safety property)](/wiki/research/FCR/FCR.md#properties-of-the-confirmation-rule)。

---

## FCR 保证什么 (What FCR Guarantees)

快速确认规则 (Fast Confirmation Rule) 依赖于**同步 (synchrony)**条件，但它能提供仅 **12 秒 (seconds)** 的**最佳情况确认时间 (best-case confirmation time)**，极大地改善了 FFG最终化规则 (FFG Finalization Rule) 的延迟 (latency)。

**FCR 绝不会确认非规范的区块。**

它提供了一个具有形式化保证 (formal guarantees) 的确认规则 (confirmation rule)。

### 确认规则的属性 (Properties of the Confirmation Rule)

- **安全性 (Safety)**：一旦确认，已确认的区块 (confirmed blocks) 在诚实视角 (honest view) 下不会被重组 (reorged)
- **单调性 (Monotonicity)**：除非触发“重置为最终化 (reset-to-finalized)”（假设失败信号 (assumption failure signal)），否则已确认的区块 (confirmed block) 绝不会后退

### FCR 的实际意义 (Practical Implications of FCR)

<details>
<summary><strong>提升钱包可靠性 (Improved Wallet Reliability)</strong></summary>

- FCR 提供了一个**可证明安全的确认信号 (provably safe confirmation signal)**

</details>

<details>
<summary><strong>降低 CEX 交易回滚的风险 (Reduce CEX Risk Trade Reversals)</strong></summary>

- 针对允许在充值后立即进行乐观交易 (optimistically trade) 的交易所 (exchanges)

</details>

<details>
<summary><strong>PBS 使用场景 (PBS Use Case)</strong></summary>

- 区块构建者 (Block builders) 可以获得关于其区块是否不太可能被重组 (reorged) 出去的指示

</details>


> 用户可以根据他们对网络条件 (network conditions) 的看法以及对快速响应的需求，选择最适合其需求的确认规则 (Confirmation Rule)。

---

# Gasper 概述 (Gasper Overview)

以太坊权益证明 (Proof of Stake, PoS) 共识 (consensus) 由 [Gasper](https://ethereum.org/developers/docs/consensus-mechanisms/pos/gasper/) 定义，其中：
- 时间单位是**时隙 (Slot)**：当前为 12 秒 (seconds)
- **纪元 (Epoch)**：32 个时隙 (slots)
- 每个纪元 (epoch) 将验证者 (validators) 集合划分为 32 个委员会 (committees)

Gasper 由两个子协议 (sub-protocols) 组成：

### LMD-GHOST
- 分叉选择算法 (Fork-choice algorithm)
- 确定规范链头 (canonical chain head)

### FFG-Casper
- 最终化由 LMD-GHOST 选择的检查点 (checkpoints)

![alt text](../img/FCR/Gasper.png)

---
### 区块产出流程 (Block Production Flow)

1. 在每个时隙 (slot) 开始时：
   - 从委员会 (committee) 中随机选择一名提议者 (proposer)
2. 提议者 (proposer) 在规范链头 (canonical head) 之上提议一个区块 (block)
3. 委员会 (committee) 中的其他验证者 (validators)：
   - 对提议的区块进行见证 (attest)
4. 分叉选择规则 (Fork-choice rule)：
   - 确定链的规范链头 (canonical head)

> 在正常情况下，分叉选择规则 (fork choice rule) 是不需要的 —— 每个时隙 (slot) 只有一个区块提议者 (block proposer)，而且诚实验证者 (honest validators) 会对其进行见证 (attest)。只有在网络出现严重异步 (asynchronicity) 或不诚实的区块提议者 (block proposer) 进行了双签 (equivocated) 时，才需要分叉选择算法 (fork choice algorithm)。然而，当这些情况确实发生时，分叉选择算法 (fork choice algorithm) 是确保正确链安全的至关重要的防御手段。


---

## LMD-GHOST 的确认规则 (Confirmation Rule for LMD-GHOST)

确认算法 (confirmation algorithm) 需要满足以下**假设 (assumptions)**：

1. 同步性 (Synchronous)：假设网络健康。诚实验证者 (honest validator) 在一个时隙 (slot) 内发送的见证 (attestations) 会在时隙结束前被任何其他诚实验证者 (honest validator) 接收到
2. 任何委员会 (committees) 集合中，最多有 **β** 比例的质押份额 (stake) 是不诚实的
    - **β** = 20-25%（可配置 (configurable)）

目标是提供一个在上述假设下安全的**快速确认规则 (fast confirmation rule)**。

这些假设在绝大多数时间里都是成立的，使该协议能够为用户提供一种比最终化 (finalization) 快得多的区块确认方式。

---

### 规范概述 (Specification overview)

⚙️ 当前规范 (specification) 可以在 [consensus-specs PR](https://github.com/ethereum/consensus-specs/pull/4747) 中找到

---

确认规则 (confirmation rule) 的**输入 (input)**是先前已确认的区块 (confirmed block)（存储在 `store.confirmed_block` 中），算法试图沿着当前的规范链 (canonical chain) 推进确认。

`get_latest_confirmed` 函数是确认规则 (confirmation rule) 的入口点，包含两个阶段：

1. **假设检查 (Assumption checking)**
    - 验证所需的假设 (assumptions) 是否仍然成立
    
2. **推进确认 (Confirmation advancement)**
   - 沿着规范链 (canonical chain) 寻找最新的已确认后代 (confirmed descendant)，从而允许算法相对于先前确认的区块快速确认 (fast-confirm) 新区块。

根据该函数执行的时间，会应用特定版本的逻辑。

---

`isOneConfirmed` 谓词 (predicate)

- 该算法在规范链 (canonical chain) 中搜索使 `isOneConfirmed` 评估为 `true` 的最长前缀 (prefix)。

- 如果一个区块通过了此项检查，则表明它具有足够的 **LMD-GHOST** 支持来抵御任何潜在的兄弟区块接管 (sibling takeover)。
    - 该谓词考虑了所有导致分叉选择 (fork choice) 转向冲突链的因素，包括：
        - 提议者提升 (proposer boost)
        - 对抗性预算 (adversarial budget) β
        - 双签 (equivocation)
        - 空时隙折价 (empty slot discounting)

该检查沿着链的后缀 (suffix) 迭代应用，从最新确认的区块 (confirmed block) 开始，并在 `isOneConfirmed` 不再成立时停止。


---

🧩 **确认规则的复杂性源于需要正确处理所有的边界情况 (edge cases)。**

---

# 总结 (Summary)

FCR 提供：
- 具有强安全保证 (strong safety guarantees) 的快速确认 (最佳情况延迟为 1-2 个时隙 (slots))
- 更好的用户体验 (UX)，无需等待最终化 (finalization)
- 为钱包 (wallets) 和交易所 (exchanges) 带来更高的可靠性

在单时隙最终性 (Single Slot Finality) 可用之前，**FCR 填补了以太坊确认模型中的一个关键空白**。

# 文档 (Documentation)

- [html 论文: 以太坊共识协议的快速确认规则（又称快速同步最终性） (html Paper: A Fast Confirmation Rule (aka Fast Synchronous Finality) for the Ethereum Consensus Protocol)](https://arxiv.org/html/2405.00549v3)
- [快速确认规则 (FCR) 分组会议播放列表 (Fast Confirmation Rule (FCR) breakout room playlist)](https://www.youtube.com/watch?v=y5_75Y_09No&list=PLJqWcTqh_zKH4c_vgCCPZ33Ilb9SdEIcq)
- [快速确认规则规范 (Fast Confirmation Rule specs)](https://github.com/ethereum/consensus-specs/pull/4747)
- FCR 简要说明草案 - 完成后将添加链接
- [快速确认规则 - Roberto Saltini (2025 视频) (Fast Confirmation Rule - Roberto Saltini (2025 video))](https://www.youtube.com/watch?v=dZU-Ch22MKY&list=PLCGgAwcxXAWyHcMm0X57jVuHtJ6e8gwIP&index=25)
- [快速确认规则 - Devcon SEA - Roberto & Luca (2024 视频) (Fast Confirmation Rule - Devcon SEA - Roberto & Luca (2024 video))](https://www.youtube.com/watch?v=p7JPRTELnJc&list=PLCGgAwcxXAWyHcMm0X57jVuHtJ6e8gwIP&index=25)

# 实现 (Implementation)

- [Prysm PR #15164](https://github.com/OffchainLabs/prysm/pull/15164)
- [Lighthouse 概念验证 EPF 工作（作者 Harsh Pratap Singh） (Lighthouse PoC EPF work by Harsh Pratap Singh)](https://hackmd.io/6H_e-WqaQFyBENsifLiH6g)
