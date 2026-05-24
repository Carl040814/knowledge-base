> [主观的 (Subjective)](https://dictionary.cambridge.org/dictionary/english/subjective)：受个人信念或情感影响或基于个人信念或情感，而不是基于事实  
> [客观的 (Objective)](https://dictionary.cambridge.org/dictionary/english/objective)：基于真实事实，且不受个人信念或情感的影响（在链接页面中向下滚动查看）

## 背景 (Background)

让我们在信息论和区块链的语境下，对“客观性 (objectivity)”和“主观性 (subjectivity)”做一下区分。如果一条信息对其“正确性 (correctness)”是**完全可验证的**，那么它就是“客观的 (objective)”。如果其正确性不是**完全可验证的**（需要融入一定程度的信念），那么它就是“主观的 (subjective)”。在现实中，大多数信息都处于客观性与主观性之间的光谱之中。

例如，在合并 (merge) 之前，如果客户端从创世区块 (genesis block) 开始，它就可以**客观地**验证其后的每一个区块。为此，客户端会向其对等节点请求区块链最新的“最终 (final)”区块头（稍后会谈到），然后通过查询父区块（又称历史记录 (history)）一直回溯 (backtrack) 到创世 (genesis) 阶段。由于父区块哈希被编码在区块中，因此验证所有查询到的父区块非常简单。当客户端到达创世区块时，它可以认定整个历史记录是**正确的 (correct)**，并且自身已处于***已同步 (synced)*** 状态。

> 如今存在不同的同步机制。上面描述的是“完全 (full)”同步的总体概述。

修改链的历史记录需要极其庞大的计算能力（工作量证明 (Proof-of-Work, PoW)），这使得合并前已验证的历史记录具有***客观性 (objective)***。然而，对于创世区块的正确性仍然需要给予一定程度的信任。但可以认为它是***弱客观的 (weakly objective)***，而不是***主观的 (subjective)***，因为大多数客户端都预装了创世区块及其校验和 (checksum)。

> 一个“最终 (final)”区块过去被认为是距离当前区块链头部第 7 个区块，因为撤销价值 6 个区块的 PoW 被认为是极其困难的。

随着合并 (merge) 的进行，以太坊为其新的共识机制 (consensus mechanism) 引入了成员资格（验证者 (validators)）和投票。此外，共识机制在逻辑上与历史记录和执行机制隔离，分别被称为共识层 (Consensus Layer, CL) 和执行层 (Execution Layer, EL)。虽然历史记录机制基本保持不变，但同步所需的最新最终区块是从共识层获取的。作为回报，EL 向 CL 验证区块执行（智能合约 (smart contracts) 等）的正确性，以便 CL 对区块进行投票并凝聚***共识 (consensus)***。然而，共识机制的改变使得从创世同步变得[“不安全 (unsafe)”](https://docs.teku.consensys.io/concepts/weak-subjectivity#sync-outside-the-weak-subjectivity-period)。**弱主观性 (weak-subjectivity) 登场**。

> 新的共识机制也带来了几个非同寻常的细微差别。理解这些细微差别对于理解弱主观性（下一节）至关重要。这是一个很好的入门材料：https://ethos.dev/beacon-chain

## 弱主观性下的同步 (Syncing in Weak Subjectivity)

当验证者退出 (validator exits) 链时，其参与共识过程的权力将被撤销。该验证者不能再对任何未来的区块进行投票/见证 (vote/attest)。然而，*该验证者可以对任何**过去**的区块进行重新投票/重新见证 (re-vote/re-attest)*。如果有足够多已退出的验证者对过去的区块（过去的某个分叉区块）重新进行见证，他们就会创建一条链的替代历史 (alternate history)。一个已经同步在规范链头 (canonical head) 处的节点不会关心这种双重投票 (equivocation)（替代历史），因为它已经处理了该验证者的退出。然而，一个正在***同步中 (syncing)*** 的节点无法知道验证者的未来状态，可能会跟随错误的链。为了避免这种情况，信标链的同步方向 (direction of syncing) 被反转了。这是信标区块同步与执行区块同步之间的第一个主要区别。

第二个区别在于信息信任的锚点 (anchor of trust in information)。由于在某些条件下链的历史记录可能会被篡改，因此同步目标 (sync target) 无法被客观地验证——即我们永远无法具体证明它存在于规范链 (canonical chain) 中。因此，对同步目标抱有很大程度的信任，它仍然是主观的 (subjective)。由于涉及这种信任，同步目标（称为弱主观性检查点 (weak subjectivity checkpoints)）是通过带外 (out-of-band) 方式共享的，因为以太坊 P2P 层中的随机对等节点连接是不可信的。需要注意的是，由于同步目标是一个“已终局的 (finalized)”检查点，它仍然是**弱**主观的 (weakly subjective)。换句话说，它允许对其正确性进行一定程度的验证（只是可能无法验证其存在性）。

第三个区别在于链的时效性 (timing of the chain)。失步时间足够长的节点也可能会受到相同的攻击。理论上，这段时间应该足够让足够多的验证者退出并发起攻击。这段时间被称为***弱主观性时期 (weak subjectivity period)***。该时间段也适用于同步目标。如果后向填充过程 (backfilling process)（信标链）或执行层同步耗费了太多时间，那么目标就会变得***过期 (stale)***。因此，在执行层同步的同时（后向填充通常足够快），新的信标区块会被持续导入，而无需检查新头部区块的执行负载 (execution payload)。这被称为***乐观地 (optimistically)*** 导入区块。

因此，弱主观性同步 (weak subjectivity sync) 遵循以下定义的步骤：
1. **通过带外方式获取弱主观性检查点 (Obtain a weak subjectivity checkpoint out-of-band)**
2. **从弱主观性检查点开始，一路向后填充区块直到创世区块 (Backfill blocks all the way back to genesis from the weak subjectivity checkpoint)**
3. **更新执行链的目标区块头 (Update the target header for the execution chain)**
4. **乐观地跟随链头，并持续更新执行链的目标区块头 (Optimistically follow the head of the chain and continuously update the target header for the execution chain)**
5. **一旦执行层已同步，则在验证后将共识层插槽标记为已验证。此时可认为该节点已完全同步 (Once the EL is synced, then mark the CL slots as verified post-verification. The node may now be considered fully synced)**

## 常用链接 (Useful Links)

1. https://docs.prylabs.network/docs/how-prysm-works/optimistic-sync
2. https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/weak-subjectivity/
3. https://www.symphonious.net/2019/11/27/exploring-ethereum-2-weak-subjectivity-period/
4. https://blog.ethereum.org/2014/11/25/proof-stake-learned-love-weak-subjectivity
5. https://notes.ethereum.org/@adiasg/weak-subjectvity-eth2
6. https://blog.ethereum.org/2016/05/09/on-settlement-finality
