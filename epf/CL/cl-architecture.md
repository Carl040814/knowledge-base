# 共识层 (Consensus Layer, CL) 架构

> 许多区块链共识协议是_有分叉的 (forkful)_。有分叉的链使用分叉选择规则 (fork choice rule)，有时会发生重组 (reorganisation)。
>
> 以太坊的共识协议结合了两个独立的共识协议。_LMD GHOST_ 基本上提供活性 (liveness)。_Casper FFG_ 提供最终性 (finality)。它们一起被称为 _Gasper_。
>
> 在一个_活跃的 (live)_ 协议中，好的事情总是会发生。在一个_安全的 (safe)_ 协议中，坏事永远不会发生。没有任何实际协议能够始终安全且始终活跃。

## 分叉选择机制 (Fork-choice Mechanism)

如 [BFT](/wiki/CL/overview.md?id=byzantine-fault-tolerance-bft-and-byzantine-generals39-problem) 中所述，由于各种原因——如网络延迟、宕机、消息乱序或恶意行为——网络中的节点可能对网络状态有不同的视图。最终，我们希望每个诚实节点都就一个完全相同的线性历史和系统状态的共同视图达成一致。协议的分叉选择规则正是帮助实现这一一致性的手段。

#### 区块树 (Block Tree)
给定一个区块树和基于节点对网络的本地视图的决策标准，分叉选择规则旨在选择最有可能成为最终的、线性的、规范的链的分支。它选择最不可能在节点收敛到共同视图时被修剪掉的分支。

<a id="img_blocktree"></a>

![Diagram for Block Tree](https://epf.wiki/images/cl/blocktree.svg)
*分叉选择规则从候选者中挑选一个头部区块 (head block)。头部区块标识了一条唯一的、回到创世区块的线性区块链。*

#### 分叉选择规则 (Fork choice rules)

分叉选择规则通过选择分支顶端的区块（称为头部区块, head block）来隐式地选择一个分支。对于任何正确的节点，任何分叉选择的第一条规则是，所选区块必须根据协议规则是有效的，并且其所有祖先也必须有效。任何无效区块都会被忽略，建立在无效区块之上的区块也是无效的。

以下是几种不同分叉选择规则的示例：

- **工作量证明 (Proof-of-Work)**：在以太坊和比特币中，使用"最重链规则 (heaviest chain rule)"（有时称为"最长链"，尽管严格来说并不准确）。头部区块是累积完成最多"工作量"的链的顶端。
> 请注意，与普遍看法相反，以太坊的工作量证明协议在其分叉选择中[并未使用](https://ethereum.stackexchange.com/questions/38121/why-did-ethereum-abandon-the-ghost-protocol/50693#50693)任何形式的 GHOST。这个误解非常顽固，可能源于[以太坊白皮书](https://ethereum.org/en/whitepaper/#modified-ghost-implementation)。最终当 Vitalik 被问及此事时，他确认虽然 GHOST 在 PoW 下曾被计划，但由于对某些未指明的攻击的担忧，从未被实现。最重链规则更简单且经过充分测试，运行良好。
- **Casper FFG (权益证明, Proof-of-Stake)**：在以太坊的 PoS Casper FFG 协议中，分叉选择规则是"跟随包含**最大高度**的已证明合理检查点 (justified checkpoint) 的链"，并且永不回滚已最终确定的区块。
- **LMD GHOST (权益证明, Proof-of-Stake)**：在以太坊的 PoS LMD GHOST 协议中，分叉选择规则是取"最贪婪最重观察子树 (Greediest Heaviest Observed SubTree)"。它涉及计算验证者对区块及其后代区块的累积投票。它还应用与 Casper FFG 相同的规则。

这些分叉选择规则中的每一个都为一个区块分配一个数值分数。获胜的区块，即头部区块，具有最高的分数。目标是所有正确节点在看到某个区块时，都会同意它是头部并跟随其分支。这样，所有正确节点最终将就一条回到创世的单一规范链达成一致。

#### 重组 (Reorgs) 与回滚 (Reversion)

当节点收到新的投票（以及在权益证明中对区块的新投票）时，它会用这些新信息重新评估分叉选择规则。通常，新区块将是当前头部区块的子区块，并成为新的头部区块。

然而，有时新区块可能是区块树中不同区块的后代。如果节点没有新区块的父区块，它会向对等节点请求该父区块以及任何其他缺失的区块。

在更新后的区块树上运行分叉选择规则可能会显示新的头部区块位于与前一个头部区块不同的分支上。当这种情况发生时，节点必须执行重组 (reorg, reorganisation)。这意味着它将移除（回滚, revert）之前包含的区块，并采用新头部分支上的区块。

例如，如果一个节点的链中有区块 $A, B, D, E,$ 和 $F$，并将 $F$ 视为头部区块，它知道区块 $C$，但 $C$ 不出现在其对链的视图中；它在侧分支上。

<a id="img_reorg0"></a>

![Diagram for Reorg-0](https://epf.wiki/images/cl/reorg-0.svg)
*此时，节点认为区块 $F$ 是最佳头部，因此其链是区块 $[A \leftarrow B \leftarrow D \leftarrow E \leftarrow F]$*

当节点稍后收到区块 $G$（它建立在区块 $C$ 上而非当前头部区块 $F$ 上）时，它必须决定 $G$ 是否应该成为新的头部。仅作为示例，如果分叉选择规则说 $G$ 是更好的头部区块，节点将回滚区块 $D, E,$ 和 $F$。它将它们从链中移除，就像它们从未被收到过一样，并回到区块 $B$ 之后的状态。

然后，节点将区块 $C$ 和 $G$ 添加到其链中并处理它们。经过这次重组后，节点的链将是 $A, B, C,$ 和 $G$。

<a id="img_reorg1"></a>

![Diagram for Reorg-1](https://epf.wiki/images/cl/reorg-1.svg)
*现在节点认为区块 $G$ 是最佳头部，因此其链必须变为区块 $[A \leftarrow B \leftarrow C \leftarrow G]$*

稍后，也许会出现一个建立在 $F$ 上的区块 $H$，而分叉选择规则说 $H$ 应该是新的头部，节点将再次重组，回滚到区块 $B$ 并重放 $H$ 分支上的区块。

由于网络延迟，一两个区块的短期重组是常见的。较长的重组应该很少见，除非链受到攻击或分叉选择规则或其实现中存在 Bug。

### 安全性 (Safety) 与活性 (Liveness)

在共识机制中，两个关键概念是安全性和活性。

**安全性 (Safety)** 意味着"坏事永远不会发生"，例如防止双重支付 (double-spending) 或最终确定冲突的检查点。它确保一致性 (consistency)，意味着所有诚实节点应始终就区块链的状态达成一致。

**活性 (Liveness)** 意味着"好事最终会发生"，确保区块链始终能够添加新区块，永远不会陷入死锁 (deadlock)。

**CAP 定理 (CAP Theorem)** 指出，没有任何分布式系统能够同时提供一致性 (Consistency)、可用性 (Availability) 和分区容忍性 (Partition tolerance)。这意味着我们无法设计一个在通信不可靠时在所有情况下既安全又活跃的系统。

#### 以太坊优先考虑活性

以太坊的共识协议旨在良好的网络条件下同时提供安全性和活性。然而，在网络问题期间它优先考虑活性。在网络分区 (network partition) 中，两侧的节点将继续产生区块，但不会达成最终性（一个安全属性）。如果分区持续存在，每一侧可能最终确定不同的历史，导致两条不可调和、独立的链。

因此，虽然以太坊努力同时实现安全性和活性，但它倾向于确保网络保持活跃并继续处理交易，即使以在严重网络中断期间潜在的安全问题为代价。

## 机器中的幽灵

以太坊的权益证明共识协议结合了两个独立的协议：[LMD GHOST](/wiki/CL/gasper?id=lmd-ghost.md) 和 [Casper FFG](/wiki/CL/gasper?id=casper-ffg.md)。它们一起构成了名为 "Gasper" 的共识协议。关于这两个协议及其如何组合工作的详细信息将在下一节 [Gasper](/wiki/CL/gasper) 中涵盖。

Gasper 旨在结合 LMD GHOST 和 Casper FFG 的优势。LMD GHOST 提供活性，通过定期产生新区块确保链持续运行。然而，它容易出现分叉，且并非正式安全。另一方面，Casper FFG 通过定期最终确定链来提供安全性，保护其免受长回滚的影响。

本质上，LMD GHOST 保持链向前移动，而 Casper FFG 通过最终确定区块来确保稳定性。这种组合使以太坊能够优先考虑活性，意味着即使 Casper FFG 无法最终确定区块，链也继续增长。尽管这种组合机制并非总是完美且存在一些复杂性，但它是一个在实际中对以太坊运行良好的实用工程解决方案。

## 架构

以太坊是一个去中心化的节点网络，节点通过对等 (peer-to-peer) 连接进行通信。这些连接由运行以太坊客户端软件的计算机形成。

![Diagram for Network](https://epf.wiki/images/cl/network.png)
*节点不需要运行验证者客户端（绿色部分）就能成为网络的一部分，但是要参与共识，需要质押 32 ETH 并运行验证者客户端。*

### 共识层的组件

- **信标节点 (Beacon Node)**：信标节点使用客户端软件来协调以太坊的权益证明共识。示例包括 Prysm、Teku、Lighthouse 和 Nimbus。信标节点与其他信标节点、本地执行节点以及（可选）本地验证者进行通信。

- **验证者 (Validator)**：验证者客户端是允许人们在以太坊共识层中质押 32 ETH 的软件。验证者在权益证明系统中提议区块，取代了工作量证明的矿工。验证者仅与本地信标节点通信，信标节点向其发出指令并将其工作广播到网络。

托管现实世界应用的主要以太坊网络称为以太坊主网 (Ethereum Mainnet)。以太坊主网是以太坊的实时生产实例，铸造和管理真实的以太坊 (ETH) 并持有真实的货币价值。

还有测试网络，为开发者、节点运行者和验证者铸造和管理测试以太坊，以便在在主网上使用真实 ETH 之前测试新功能。每个以太坊网络都有两层：执行层 (Execution Layer, EL) 和共识层 (Consensus Layer, CL)。每个以太坊节点都包含两层的软件：执行层客户端软件（如 Nethermind、Besu、Geth 和 Erigon）和共识层客户端软件（如 Prysm、Teku、Lighthouse、Nimbus 和 Lodestar）。

<a id="img_node-layers"></a>

![Diagram for CL](https://epf.wiki/images/cl/cl.png)

**共识层**负责维护共识链（信标链, beacon chain）并处理从其他对等节点接收的共识区块（信标区块, beacon block）和证明 (attestation)。**共识客户端**参与一个单独的[对等网络](/wiki/CL/cl-networking.md)，该网络具有与执行客户端不同的规范。它们需要参与区块 Gossip 协议 (block gossip)，以从对等节点接收新区块，并在轮到它们提议时广播区块。

EL 和 CL 客户端并行运行，需要连接以进行通信。共识客户端向执行客户端提供指令，执行客户端将交易包传递给共识客户端以包含在信标区块中。通信是通过 **Engine-API** 使用本地 RPC 连接实现的。它们共享一个 [ENR](/wiki/CL/cl-networking?id=enr-ethereum-node-records)，每个客户端有单独的密钥（eth1 密钥和 eth2 密钥）。

### 控制流

**当共识客户端不是区块生产者时：**
1. 通过区块 Gossip 协议接收一个区块。
2. 预验证该区块。
3. 将区块中的交易作为执行负载 (execution payload) 发送到执行层。
4. 执行层执行交易并验证区块状态。
5. 执行层将验证数据发送回共识层。
6. 共识层将区块添加到其区块链中并对其进行证明，在网络上广播证明。

**当共识客户端是区块生产者时：**
1. 收到成为下一个区块生产者的通知。
2. 调用执行客户端中的创建区块方法。
3. 执行层访问交易内存池 (mempool)。
4. 执行客户端将交易打包成区块，执行它们，并生成区块哈希。
5. 共识客户端将交易和区块哈希添加到信标区块中。
6. 共识客户端通过区块 Gossip 协议广播该区块。
7. 其他客户端验证该区块并对其进行证明。
8. 一旦被足够多的验证者证明，该区块就被添加到链的头部，变为已证明合理 (justified) 并最终确定 (finalized)。


### 状态转换 (State Transitions)

状态转换函数 (state transition function) 在区块链中至关重要。每个节点维护一个反映其对世界视图的状态。

节点通过按顺序使用"状态转换函数"应用区块来更新其状态。此函数是"纯 (pure)"的，意味着其输出仅取决于输入且没有副作用。因此，如果每个节点从相同的状态（创世状态, Genesis state）开始并应用相同的区块，它们最终都会得到相同的状态。如果它们没有，那就是共识失败。

如果 $S$ 是一个信标状态 (beacon state)，$B$ 是一个信标区块 (beacon block)，则状态转换函数 $f$ 为：

$$S' \\equiv f(S, B)$$

这里，$S$ 是前状态 (pre-state)，$S'$ 是后状态 (post-state)。函数 $f$ 随着每个新区块迭代更新状态。

### 信标链状态转换

与区块驱动的工作量证明不同，信标链是时隙驱动的。状态更新取决于时隙的推进，无论是否存在区块。

信标链的状态转换函数包括：

1. **每时隙转换 (Per-slot transition)**：$S' \\equiv f_s(S)$
2. **每区块转换 (Per-block transition)**：$S' \\equiv f_b(S, B)$
3. **每纪元转换 (Per-epoch transition)**：$S' \\equiv f_e(S)$

每个函数在特定时间更新链，如信标链规范中所定义。

### 有效性条件 (Validity Conditions)

从前状态和一个签名区块得到的后状态是 `state_transition(state, signed_block)`。触发未处理异常（例如失败的 assert 或越界列表访问）或 uint64 溢出/下溢的转换被视为无效。

### 信标链状态转换函数

与前状态 `state` 和签名区块 `signed_block` 对应的后状态定义为 `state_transition(state, signed_block)`。触发未处理异常的状态转换（例如失败的 `assert` 或越界列表访问）被视为无效。导致 `uint64` 溢出或下溢的状态转换也被视为无效。

```python
def state_transition(state: BeaconState, signed_block: SignedBeaconBlock, validate_result: bool=True) -> None:
    block = signed_block.message
    # Process slots (including those with no blocks) since block
    process_slots(state, block.slot)
    # Verify signature
    if validate_result:
        assert verify_block_signature(state, signed_block)
    # Process block
    process_block(state, block)
    # Verify state root
    if validate_result:
        assert block.state_root == hash_tree_root(state)
```

```python
def verify_block_signature(state: BeaconState, signed_block: SignedBeaconBlock) -> bool:
    proposer = state.validators[signed_block.message.proposer_index]
    signing_root = compute_signing_root(signed_block.message, get_domain(state, DOMAIN_BEACON_PROPOSER))
    return bls.Verify(proposer.pubkey, signing_root, signed_block.signature)
```

```python
def process_slots(state: BeaconState, slot: Slot) -> None:
    assert state.slot < slot
    while state.slot < slot:
        process_slot(state)
        # Process epoch on the start slot of the next epoch
        if (state.slot + 1) % SLOTS_PER_EPOCH == 0:
            process_epoch(state)
        state.slot = Slot(state.slot + 1)
```


## 资源

- Vitalik Buterin, ["Parametrizing Casper: the decentralization/finality time/overhead tradeoff"](https://medium.com/@VitalikButerin/parametrizing-casper-the-decentralization-finality-time-overhead-tradeoff-3f2011672735)
- [Engine API spec](https://hackmd.io/@n0ble/consensus_api_design_space)
- [Vitalik's Annotated Ethereum 2.0 Spec](https://notes.ethereum.org/@vbuterin/SkeyEI3xv)
- Ethereum, ["Eth2: Annotated Spec"](https://github.com/ethereum/annotated-spec)
- Martin Kleppmann, [Distributed Systems.](https://www.youtube.com/playlist?list=PLeKd45zvjcDFUEv_ohr_HdUFe97RItdiB)
- Leslie Lamport et al., [The Byzantine Generals Problem.](https://lamport.azurewebsites.net/pubs/byz.pdf)
- Austin Griffith, [Byzantine Generals - ETH.BUILD.](https://www.youtube.com/watch?v=c7yvOlwBPoQ)
- Michael Sproul, ["Inside Ethereum"](https://www.youtube.com/watch?v=LviEOQD9e8c) 
- [Eth2 Handbook by Ben Edgington](https://eth2book.info/capella/part2/consensus/)
- [Lighthouse Consensus Client architecture](https://www.youtube.com/watch?v=pLHhTh_vGZ0)
