# 客户端架构 (Client Architecture)

## 概述 (Overview)

除了执行层 (Execution Layer, EL) 的核心任务——交易执行 (Transaction Execution) 之外，执行层客户端 (Execution Layer Client) 还承担着几项关键责任。这些包括对区块链数据 (Blockchain Data) 进行验证 (Verification) 并存储其本地副本，通过传闻协议 (Gossip Protocols) 与其他执行层客户端进行网络通信，维护一个交易池 (Transaction Pool)，并满足推动其功能的共识层 (Consensus Layer, CL) 的要求。这种多方面的运作确保了以太坊网络 (Ethereum Network) 的健壮性 (Robustness) 和完整性 (Integrity)。

客户端的架构是围绕各种特定的标准构建的，每个标准在整体功能中都扮演着独特的角色。执行引擎 (Execution Engine) 位于最顶端，驱动着执行层 (Execution Layer)，而执行层本身又由共识层 (Consensus Layer) 驱动。执行层运行在 DevP2P（网络层 (Networking Layer)）之上，网络层通过提供合法的引导节点 (Boot Nodes) 来初始化，这些引导节点提供了进入网络的初始访问点。当我们调用 Engine API 的其中一个方法（例如分叉选择更新 (Fork Choice Updated)）时，我们可以通过订阅我们偏好的同步模式 (Sync Mode) 的主题从对等方 (Peers) 下载区块。

<img src="https://epf.wiki/images/el-architecture/architecture-overview.png" width="1000"/>

该图展示了该设计的简化表示，排除了一些组件。

**以太坊虚拟机 (Ethereum Virtual Machine, EVM)**

以太坊以虚拟中央处理器 (Central Processing Unit, CPU) 为中心。计算机在硬件级别有自己的中央处理器 (CPUs)，它们可以是许多类型，例如 x86、ARM、RISC-V 或其他类型。这些多样化的处理器架构具有独特的指令集 (Instruction Sets)，使它们能够执行算术、逻辑和数据操作等任务，从而使计算机能够作为通用计算机器运行。因此，当执行以硬件级指令集编写的程序时，结果可能会根据执行它的特定硬件而有所不同。在计算机科学中，我们通过创建虚拟机（如 Java 虚拟机 (Java Virtual Machine, JVM)）对指令集进行虚拟化来解决这个问题。这些虚拟机能够确保无论底层硬件如何，结果都是一致的。EVM 是为以太坊程序设计的虚拟化执行引擎。它确保无论在其上运行的硬件如何，都能获得一致的结果，并促进所有以太坊客户端之间就计算结果达成共识 (Consensus)。

此外，以太坊在其设计哲学中融入了三明治复杂度模型 (Sandwich Complexity Model)。这意味着外层应该保持简单，而所有的复杂性都应该集中在中间层。在此背景下，EVM 的代码可以被视为最外层，而像 Solidity 这样的高级语言 (High-level Language) 可以被视为顶层。在这两者之间，有一个复杂的编译器 (Compiler)，它将 Solidity 代码翻译成 EVM 的字节码 (Bytecode)。

**状态 (State)**

以太坊是一个通用的计算系统，它作为状态机 (State Machine) 运行，这意味着它可能会根据接收到的输入在几种状态之间进行转换。此外，以太坊与比特币等其他区块链显着不同，因为它维护着一个全局状态 (Global State)，而比特币仅保留全局未花费的交易输出 (Unspent Transaction Outputs, UTXOs)。“状态”一词是指存储各种信息的综合数据收集、数据结构（如默克尔·帕特里夏树 (Merkle Patricia Tries)）以及数据库。这包括地址、余额、合约的代码和数据，以及当前状态和网络状态。

**交易 (Transactions)**

EVM 通过称为状态过渡 (State Transition) 的过程产生数据并修改以太坊网络的状态。这种状态过渡是由交易触发的，这些交易在 EVM 内部进行处理。如果交易被认为是合法的，它会导致以太坊网络的状态发生改变。

**DevP2P**

用于与其他执行层客户端通信的接口。交易最初存储在内存池 (Mempool) 中，内存池充当所有传入交易的仓库，并通过执行层客户端使用点对点 (P2P) 通信传播到网络中的其他执行层客户端。通过网络发送的交易的每个接收方在将其广播到网络之前，都会确认其有效性。

**JSON-RPC API**

在使用钱包 (Wallet) 或去中心化应用 (Decentralized Application, DApp) 时，我们与执行层的通信是通过标准化的 JSON-RPC API 进行的。这使我们能够向外部查询以太坊状态，或向其发送由钱包签名的交易，该交易随后由执行层客户端验证并传播到整个网络。

**Engine API (引擎 API)**

**Engine API** 驻留在**执行层 (Execution Layer)** 中，专门用于**共识层 (Consensus Layer)** 与**执行层 (Execution Layer, EL)** 之间的内部通信。该引擎向共识层公开了两大类端点 (Endpoints)：**fork choice updated (分叉选择更新)** 和 **new payload (新有效载荷)**，并以它们公开的三个版本为后缀（V1-V3）。

1. **New Payload (V1/V2/V3) (新有效载荷):**  
   处理有效载荷的验证和插入。当 CL 收到一个新的信标区块 (Beacon Block) 时，它会提取执行有效载荷 (Execution Payload) 并调用 EL 上的 `engine_newPayload`。EL 通过以下方式验证有效载荷：
   - 检查有效载荷头中的父区块哈希 (Parent Block Hash) 是否存在，且与本地链中的预期父区块匹配。
   - 验证任何额外的执行承诺（例如，Cancun 升级后的数据）。
   - 执行交易并更新状态。
   
   响应包含一个状态：
   - **VALID (有效):** 完全执行且正确。
   - **INVALID (无效):** 有效载荷或祖先区块验证失败。
   - **SYNCING (同步中):** EL 仍在追赶进度（例如，缺失区块）。
   - **ACCEPTED (已接受):** 基础检查通过，但完整执行挂起（在浅层状态客户端中很常见）。

2. **Fork Choice Updated (V1/V2/V3) (分叉选择更新):**  
   管理状态同步并触发区块构建 (Block Building)。CL 发送分叉选择更新（带有区块头、安全区块和最终确定区块的哈希），如果它被选中提议区块，则可能包括有效载荷属性 (Payload Attributes)。EL 将：
   - 更新其规范链头 (Canonical Head)。
   - 如果提供了有效载荷属性，则启动有效载荷构建。
   - 返回包含状态的响应，如果是构建区块，还会返回一个 `payloadId`。该状态指示 EL 当前处理分叉选择更新并（如果适用）开始构建区块的能力。

可能返回给 CL 的状态：
- **VALID (有效):** 分叉选择更新已成功处理，EL 的链视图已是最新的。
- **INVALID (无效):** 提供的分叉选择引用了无效的区块或链片段。
- **SYNCING (同步中):** EL 仍在追赶进度（例如，评估分叉选择所需的缺失区块或状态）。
- **ACCEPTED (已接受):** 分叉选择更新被临时接受，但完整验证挂起。当 EL 具有浅层状态 (Shallow State) 或非规范分叉的完整历史不完整时，可能会发生这种情况。

**同步 (Sync)**

为了在以太坊上准确处理交易，我们必须在全局状态上达成共识，而不仅仅依赖于我们的本地视角。执行层客户端的全局状态同步 (Global State Synchronization) 是由共识层中受 LMD-GHOST 算法支配的分叉选择规则触发的。然后，它通过 Engine API 的分叉选择更新 (Fork Choice Updated) 端点中继到执行层。同步需要两个可能的流程：从对等方下载远程区块并在 EVM 中进行验证。状态响应（VALID、INVALID、SYNCING、ACCEPTED）指示了 EL 当前的同步水平。

### 能力交换 (Exchange of Capabilities)

在常规运行开始之前，CL 和 EL 通过 `engine_exchangeCapabilities` 方法进行能力交换。此步骤在客户端之间协商支持的 Engine API 方法版本，确保双方使用通用的协议版本（例如，V1、V2、V3）进行操作。这种交换对于确保兼容性并启用新功能，同时保持向后兼容性至关重要。

**正常路径流程 – 节点启动和验证者操作 (Happy Path Flow – Node Startup and Validator Operation):**

1. **节点启动 (Node Startup):**  
   - CL 调用 `engine_exchangeCapabilities` 与 EL 共享支持的 Engine API 方法和版本的列表。
   - EL 回应其自身支持的方法列表。
   - 接下来，CL 发送一个初始的 `engine_forkchoiceUpdated` 调用（不带有效载荷属性）以通知 EL 当前的分叉选择。
   - 如果 EL 仍在追赶进度，它将返回 SYNCING 状态。一旦追上，它将响应 VALID。

2. **验证者操作 (Validator Operation):**  
   - 在每个时隙 (Slot) 中，CL 发送 `engine_forkchoiceUpdated` 调用以更新 EL 的状态。
   - 当验证者被指派提议区块时，CL 会在分叉选择更新中包含有效载荷属性，以触发区块构建。
   - EL 返回有效载荷状态以及一个 `payloadId`，CL 稍后将其与 `engine_getPayload` 配合使用，以检索构建的执行有效载荷。
   - 另外，当验证者从网络中接收到信标区块时（由另一个验证者提议），CL 会提取执行有效载荷并调用 EL 上的 `engine_newPayload` 来验证有效载荷。

## 架构的组件 (Components of the Architecture)

### 引擎 (Engine)

执行层客户端充当*执行引擎 (Execution Engine)*，并公开 Engine API，这是一个经过身份验证的端点，它连接到共识层客户端。该引擎也被执行层客户端称为外部共识引擎。执行层客户端只能由单个共识层驱动，但共识层客户端实现可以连接到多个执行层客户端以实现冗余。Engine API 在 HTTP 上使用 JSON-RPC 接口，并需要通过 [JWT](https://jwt.io/introduction) 令牌进行身份验证。此外，Engine JSON-RPC 不向共识层以外的任何人公开。然而，值得注意的是，JWT 主要用于验证有效载荷（即发送方是共识层客户端），它并不对流量进行加密。

这种设计强制执行了明确的职责分离：CL 处理共识和分叉选择，而 EL 验证并执行交易。

#### 例程 (Routines)

##### 有效载荷验证 (Payload validation)

有效载荷是根据区块头 (Block Header) 和执行环境规则集进行验证的：

<img src="https://epf.wiki/images/el-architecture/payload-validation-routine.png" width="1000"/>

随着合并 (The Merge) 的进行，以太坊网络中执行层的功能发生了改变。此前，它承担着管理区块链共识、确保区块正确顺序以及处理重组 (Reorganizations) 的责任。然而，在合并之后，这些任务已被委托给共识层，从而显着简化了执行层。现在，我们可以将执行层主要设想为执行状态过渡函数 (State Transition Function)。

为了更好地理解上述概念，从共识层与执行层的关系来考察是有益的。共识规范在 deneb 信标链规范中定义了*处理执行有效载荷 (Process Execution Payload)*，这是由信标链在进行验证区块和推进共识层所需的几项验证时执行的。这里的执行层由 `execution_engine` 函数表示，该函数充当共识层和执行层之间的通信手段，并具有许多不同的复杂性。

在*处理执行有效载荷*期间，我们首先进行几项高级别检查，包括验证父哈希的准确性和验证时间戳。此外，我们执行各种轻量级验证。随后，我们将有效载荷传输到执行层，并在那里进行区块验证。通知有效载荷函数，是充当共识层和执行引擎之间接口的最低级别函数。它仅包含函数的签名，没有任何实现细节。其唯一目的是将执行有效载荷传输到执行引擎，执行引擎充当执行层的客户端。然后，执行引擎执行实际的状态过渡函数，这涉及验证区块头的准确性并确保交易被正确地应用于状态。执行引擎最终将返回一个布尔值，指示状态过渡是否成功。从共识层的角度来看，这仅仅是区块的验证。

这是以 Go 编写的区块级状态过渡函数 (State Transition Function, STF) 的简化描述。STF 是区块验证和插入管道的关键组件。虽然该示例特定于 Geth，但它也代表了其他客户端中 STF 的运行情况。值得提及的是，在不同客户端的代码中，除了 EELS Python 规范客户端外，状态过渡函数很少直接以其名称命名。这是因为其实际操作分散在客户端架构的许多组件中。

```go
func stf(parent types.Block, block types.Block, state state.StateDB) (state.StateDB, error) { //1
    if err := core.VerifyHeaders(parent, block); err != nil { //2
            // 检测到区块头错误
            return nil, err
  }
  for _, tx := range block.Transactions() { //3
      res, err := vm.Run(block.header(), tx, state)
      if err != nil {
              // 交易无效，区块无效
              return nil, err
      }
      state = res
  }
  return state, nil
}
```

1. 状态过渡函数的参数和返回值
   - 在此上下文中，我们检查父区块和当前区块，以验证从父区块到当前区块的某些过渡逻辑。
   - 我们将状态数据库 (State DB) 作为参数传入，它包含与父区块相关的所有状态数据。这代表了最新的有效状态。
   - 我们返回代表状态过渡后更新状态的状态数据库。
   - 如果状态过渡失败，我们不更新状态数据库并返回错误。
2. 在状态过渡函数过程中，我们首先验证区块头 (Headers)
   - 作为区块头验证失败的例证，让我们考虑 Gas 限制 (Gas Limit) 字段，这在历史上也很重要。目前，Gas 限制约为 3000 万。需要注意的是，Gas 限制在执行层内不是固定的。区块生产者有能力使用一种允许他们将 Gas 限制增加或减少前一个区块 Gas 限制的 1/1024 的技术来修改 Gas 限制。因此，如果您在单个区块中将 Gas 限制从 3000 万提高到 4000 万，区块头验证将失败，因为它超出了 3000 万加上 3000 万的千分之一的阈值。
   - 区块头验证失败的其他实例可能在区块编号不按顺序排列时出现。通常，信标链负责检测此类差异，尽管在某些情况下在此阶段也会检测到。当 1559 基础费用 (Base Fee) 未根据上一次已使用 Gas 与 Gas 限制之间的比较进行准确更新时，也可能会发生失败。
3. 一旦区块头验证完成，我们将区块头中的环境视为应该在其中执行交易的环境，并应用交易。我们遍历区块中的交易，并在 EVM 中执行每笔交易。
   - 区块头被传递给 EVM，以便为处理交易提供必要的上下文。此上下文包括 coinbase、Gas 限制和时间戳等指令，这些都是正确执行所需的。
   - 此外，我们传入交易和状态。
   - 如果执行失败，我们只需返回错误，这表示区块中存在无效交易，从而使该区块无效。在执行层中，区块中存在的任何错误都会使整个区块无效，因为它污染了整个区块。
   - 一旦我们确认交易的有效性，我们就继续用结果更新我们的状态。该状态现在代表应用了新区块中所有交易后的累积状态。

从信标链的角度来看，上面提到的状态过渡函数包含在对 "new payload" 函数的调用中。

```go
func newPayload(execPayload engine.ExecutionPayload) bool {
    if _, err := stf(..); err != nil {
        return false
  }
  return true
}
```

信标链调用 new payload 函数并转移执行有效载荷作为参数。在执行层上，我们使用来自执行有效载荷的信息调用状态过渡函数。如果状态过渡函数没有产生错误，我们返回 true。否则，我们返回 false 以指示区块无效。

##### Geth

###### Geth 中的交易执行 (Transaction Execution in Geth)
Geth 与其他以太坊执行客户端一样，通过验证签名、检查随机数 (Nonces)、扣除 Gas 费用和更新状态来处理交易。交易首先进入内存池 (Mempool)，在那里等待被包含在区块中。一旦被拾取，Geth 就会执行它们，修改账户余额、合约存储和其他状态数据。

🔗 [交易执行规范代码 (Transaction Execution Specs Code)](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork.py#L542)

###### 区块处理与状态更新 (Block Processing & State Updates)
每个新区块都包含多笔交易，Geth 会按顺序处理这些交易。一旦所有交易执行完毕，最终状态就会被提交，并存储状态根哈希以确保一致性。此过程遵循一套定义的规则来维护网络完整性。

🔗 [状态过渡代码 (State Transition Code)](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork.py#L145)

###### 网络与点对点通信 (Networking & Peer-to-Peer Communication)
以太坊节点使用 DevP2P 进行通信，这是一种允许执行客户端交换交易和区块的协议。当发送新交易时，它会通过点对点连接在网络中传播，确保所有节点保持同步。每个接收方在转发交易之前都会对其进行验证，从而防止垃圾邮件和无效的状态过渡。

🔗 [DevP2P 规范 (DevP2P Specification)](https://github.com/ethereum/devp2p/blob/master/caps/eth.md)

###### EVM 执行 (EVM Execution)
在其核心，Geth 运行以太坊虚拟机 (Ethereum Virtual Machine, EVM)，它处理智能合约逻辑。与合约交互的每笔交易都在 EVM 内部执行，确保所有节点之间的一致性和确定性。

🔗 [Geth 中的 EVM 代码 (EVM Code in Geth)](https://github.com/ethereum/go-ethereum/blob/master/core/vm/evm.go#L90)

##### 状态同步 (State Sync)

所有执行客户端都需要最新的世界状态 (World State) 来验证和构建区块。为了引导新节点的当前状态，客户端利用以太坊的 DevP2P 子协议：`eth/*`（有线协议）用于获取区块头、区块体和收据，以及 `snap/1` 用于创建状态快照。使用这些子协议，客户端可以在两种同步策略之间进行选择：**完全同步 (Full Sync)** 或 **快照同步 (Snap Sync)**。快照同步节点与完全逐块同步节点之间的区别在于，快照同步节点是从比创世区块更近的初始检查点 (Checkpoint) 开始的。

让我们看看完全同步和快照同步的流程是如何工作的。

### 完全同步 (Full Sync)

完全同步为了时间和资源而权衡了绝对的无信任性。客户端从创世区块 (Genesis) 到链尖 (Tip) 重新播放每个区块和交易，一步步重建状态树：
1. 通过 `eth/*` 协议使用 `GetBlockHeaders` 下载自创世区块以来的每个区块头。
2. 使用 `GetBlockBodies` 检索每个区块的交易和叔区块 (Uncles)。
3. 按照 EVM 顺序依次执行每笔交易，在每个区块更新本地树。
4. 确认本地状态树的根与链尖的根匹配。每次状态过渡都已通过验证。

这种方法保证了最大的安全性，但在主网 (Mainnet) 上可能需要几天时间，并消耗大量的 CPU、磁盘和网络资源。完全同步是一种幼稚的默认策略，因为它从创世区块开始，随着区块高度的不断增长，所需时间越来越长。此外，在 [EIP-4444](https://eips.ethereum.org/EIPS/eip-4444) 完全实施后，将不再支持从创世区块开始的完全同步。EIP-4444 之后的同步将改为**检查点同步 (Checkpoint Syncs)**，这意味着同步将从弱主观性检查点 (Weak Subjectivity Checkpoint) 而不是从创世区块开始。

### 快照同步 (Snap Sync)

快照同步通过仅获取树叶子节点（账户和存储插槽）以及默克尔证明来重构数据透视区块 (Pivot Block) 的状态，然后单独下载任何所需的合约字节码，并在本地重建树：
1. 选择一个最近最终确定的区块作为数据透视区块。
2. 在 `eth/*` 上，通过 `GetBlockHeaders` 请求该区块的头部，以了解其状态根 (stateRoot)。
3. 通过 `eth/*` 获取直至数据透视区块的区块头，以便知道要针对哪个状态根。
4. 分块下载数据透视区块的世界状态树和账户存储树的叶子节点。
   - **Accounts (账户)**：使用 `GetAccountRange` 拉取连续的世界状态树叶子节点值。
   - **Storage (存储)**：使用 `GetStorageRanges` 拉取每个账户的连续存储插槽叶子节点。
5. 针对账户体中发现的每个代码哈希发送 `GetByteCodes`，以获取该账户的合约代码。
6. 在本地将每个获取的叶子节点插入到一个新的快照数据库 (Snapshot DB) 中，通过默克尔范围证明针对数据透视区块的状态根进行验证。
> 注意：在实践中，客户端将这些获取的叶子节点存储在特定于快照的数据库格式中，该格式不同于正常执行期间使用的默克尔树。此格式针对范围查询和快速重建进行了优化。完整的 MPT 结构是在治愈阶段创建和验证的。

### 治愈阶段 (Healing Phase)

在步骤 6 之后，我们拥有了存储在扁平快照数据库中的数据透视区块状态的快照。然而，由于链在持续向前推进，数据可能会陈旧、不完整或不一致。因此，需要一个治愈阶段 (Healing Phase)。在治愈期间，客户端遍历快照数据库，并验证状态数据对于数据透视区块的状态根是完整且一致的。任何缺失的树节点、存储插槽或合约字节码都使用针对性的快照请求来获取。治愈确保最终状态完整、一致，并且可以完全重构成有效的 MPT。

此时，我们已经有了数据透视区块状态的快照，因此我们将数据透视区块之后的区块交易应用于下载的状态，以到达链尖。

快照同步将主网引导时间从几天缩短到几小时。其权衡在于，治愈阶段对于硬件要求很高，以便能够超越新区块产生带来的树变化。

当区块链数据通过验证且客户端追上链尖（从而能够构建最新状态）时，完全同步和快照同步都会结束。

##### 有效载荷构建 (Payload building)

更多细节在 [区块生产 (Block Production)](/wiki/EL/block-production.md) 页面。

#### 方法 (Methods)

##### 新有效载荷 (New payload)

验证先前由有效载荷构建例程构建的有效载荷。

<img src="https://epf.wiki/images/el-architecture/new-payload.png" width="1000"/>

##### 分叉选择更新 (Fork choice updated)

权益证明 LMD-GHOST 分叉选择规则和有效载荷构建例程的实例化。

<img src="https://epf.wiki/images/el-architecture/forkchoice-updated.png" width="1000"/>

### 内部共识引擎 (Internal Consensus engines)

执行层有自己的共识引擎，以配合其自身拥有的信标链副本。执行层共识引擎被称为 Ethone（执行层专用共识引擎），其功能大约是共识层完整共识引擎的一半。

| 功能 (Function) | 信标（权益证明） (Beacon (Proof-of-Stake)) | Clique（权威证明） (Clique (Proof-of-Authority)) | Ethash（工作量证明） (Ethash (Proof-of-Work)) |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------- |
| **Author (作者)**: 区块铸造者的以太坊地址 | 如果区块头被归类为权益证明 (PoS) 区块头，且区块头难度设置为 0，我们将检索区块头的 coinbase。否则，我们会将区块头转发给信标的 Ethone 引擎（Clique 或 Ethash）以进行进一步处理。 | 获取创建区块的账户地址。从区块头的 extraData 中恢复公钥的过程由 ecrecover 执行。 | |
| **Verify Header(s) (验证区块头)**: 处理一批区块头并根据当前共识引擎的规则对其进行验证。 | 根据 [终端总难度 (Terminal Total Difficulty, TTD)](https://eips.ethereum.org/EIPS/eip-3675#definitions) 将区块头分为 TTD 前和 TTD 后批次。使用 Ethone 引擎验证 TTD 前的批次，并使用信标的“验证区块头”验证 TTD 后的批次。 | 验证区块头的时间不大于系统时间。 | |
| | 在这里，我们执行类似于 [执行层规范 (Execution Layer Specs)](wiki/EL/el-specs?id=block-header-validation) 维基页面中介绍且在下面客户端代码表中涵盖的区块头验证。 | 如果它是检查点区块（纪元 (Epoch) 的第 1 个时隙），则确保它没有受益人。 | |
| | | 区块头 nonce 可以取两个值：0x00..0，表示投票添加签名者，或 0xff..f，表示投票删除签名者。然而，在检查点，我们只能投票删除签名者。 | |
| | | extraData 长度必须能够容纳 vanity（虚荣数据） + signature（签名）。在检查点，extraData 包含签名者列表 + 签名。 | |
| | | 区块头 Gas 检查。 | |
| | | 检索快照 (Snapshot) | |
| | | 在检查点区块上，根据 extraData 验证快照中的签名者 | |
| | | 调用 verify Seal 函数以确定区块头中包含的签名的所有标准是否都已满足。恢复过程涉及从区块头和 Clique 对象中的近期签名者列表中提取信息。然后我们验证签名者是否包含在快照中。 | |
| **Verify Uncles (验证叔区块)** | 如果区块头是权益证明区块头，检查叔区块的长度是否为 0。如果区块头不是权益证明，验证叔区块的过程通过 Ethone 引擎完成。 | 在 Clique 中，不应存在叔区块 | |
| **Prepare (准备)**: 初始化区块头的共识字段。 | 如果达到 TTD，我们将区块头的难度设置为信标的难度 0，否则我们调用 Ethone 的 prepare | 通过提供父区块哈希和编号来创建投票快照。在反向迭代过程中，我们从区块编号开始向后进行。如果我们到达创世区块，如果我们使用的是不存储父区块的轻客户端，如果我们在反向遍历中到达了一个纪元，或者如果遍历的区块头超过了软最终性 (soft Finality) 值（表明该段被认为是不可变的），我们就停止迭代。在停止迭代的检查点处，我们创建一个快照。 | |
| | | 如果我们处于纪元的结束位置，我们将遍历快照对象的 proposals 字段中的地址，并随机选择一个作为 coinbase。如果提议被授权，我们将投出授权票 (auth-vote)；否则，我们将投出删除票 (drop vote)。 | |
| | | 根据签名者的轮次设置区块头难度（如果签名者轮到则为 2，否则为 1） | |
| | | 验证 extraData 包含所有必要元素，包括 extraVanity 和区块发生在纪元结束时签名者的列表。这被添加到区块头的 extraData 字段中。 | |
| **Finalize (最终确定)**: 在对状态进行更改后，状态数据库可能会更新，但此操作不涉及区块的组装。 | 如果区块头不是权益证明区块头，我们执行 Ethone 的 finalize 函数。否则，我们遍历区块中的提现，将其金额从 wei 转换为 gwei。然后我们通过将转换后的金额添加到与当前提现相关联的地址来修改状态。 | Clique 没有交易后共识规则，在权威证明中没有区块奖励 | |
| **FinalizeAndAssemble (最终确定并组装)**: 最终确定并组装最终区块 | 如果区块头不是权益证明区块头，我们调用 Ethone 的 FinalizeAndAssemble。如果不存在提现且区块在上海升级 (Shanghai Fork) 之后，我们会包含一个空的提现对象。接下来，我们调用 finalize 函数来计算状态根。然后我们将此值分配给区块头对象的 root 属性。最后，我们通过结合区块头、交易、叔区块、收据和提现来构建一个新区块。 | 验证没有提现，调用 finalize 函数，计算状态数据库的状态根，并将其分配给区块头。使用区块头、交易和收据构建一个新区块。 | |
| **Seal (密封)**: 为区块生成密封请求，并将请求推送到给定的通道中 | 如果区块头不是权益证明区块头，我们调用 Ethone 的 seal。否则，我们不采取任何行动并返回 nil。密封的验证由共识层执行。 | 确保该区块不是初始区块，获取快照，并确认我们拥有签名的权限且未包含在近期签名者列表中。协调我们各自轮次的定时，应用 sign 函数进行签名，并通过指定通道传输安全密封的区块。 | |
| **SealHash**: 密封前的区块哈希 | | | |
| **CalcDifficulty**: 难度调整算法，返回新区块的难度 | | | |

#### 客户端代码 (Client code)

| | EELS(cancun) (以太坊执行层规范(cancun)) | Geth | Reth | Erigon | Nethermind | Besu |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---- | ---- | ------ | ---------- | ---- |
| $V(H) \equiv H_{gasUsed} \leq H_{gasLimit}$ | [validate_header](https://github.com/ethereum/execution-specs/blob/9fc7925c80ff2f3949e1cc340a4a0d36fcd4161c/src/ethereum/cancun/fork.py#L294) | | | | | |
| $H_{gasLimit} < P(H)_{H_{gasLimit'}} + floor(\frac{P(H)_{H_{gasLimit'}}}{1024} ) $ | validate_header -> calculate_base_fee_per_gas -> [ensure](https://github.com/ethereum/execution-specs/blob/9fc7925c80ff2f3949e1cc340a4a0d36fcd4161c/src/ethereum/cancun/fork.py#L256) | | | | | |
| $H_{gasLimit} > P(H)_{H_{gasLimit'}} - floor(\frac{P(H)_{H_{gasLimit'}}}{1024} ) $ | '' | | | | | |
| $H_{gasLimit} > 5000$ | calculate_base_fee_per_gas -> [check_gas_limit](https://github.com/ethereum/execution-specs/blob/9fc7925c80ff2f3949e1cc340a4a0d36fcd4161c/src/ethereum/cancun/fork.py#L1132) | | | | | |
| $H_{timeStamp} > PH)_{H_{timeStamp'}} $ | validate_header-> [ensure](https://github.com/ethereum/execution-specs/blob/9fc7925c80ff2f3949e1cc340a4a0d36fcd4161c/src/ethereum/cancun/fork.py#L323) | | | | | |
| $H_{numberOfAncestors} = PH)_{H_{numberOfAncestors'}} + 1 )$ | validate_header-> [ensure](https://github.com/ethereum/execution-specs/blob/9fc7925c80ff2f3949e1cc340a4a0d36fcd4161c/src/ethereum/cancun/fork.py#L324) | | | | | |
| $length(H_{extraData}) \leq 32_{bytes} $ | validate_header-> [ensure](https://github.com/ethereum/execution-specs/blob/9fc7925c80ff2f3949e1cc340a4a0d36fcd4161c/src/ethereum/cancun/fork.py#L325) | | | | | |
| $H_{baseFeePerGas} = F(H) $ | | | | | | |
| $H_{parentHash} = KEC(RLP( P(H)_H )) $ | validate_header-> [ensure](https://github.com/ethereum/execution-specs/blob/9fc7925c80ff2f3949e1cc340a4a0d36fcd4161c/src/ethereum/cancun/fork.py#L332) | | | | | |
| $H_{ommersHash} = KEC(RLP(())) $ | validate_header-> [ensure](https://github.com/ethereum/execution-specs/blob/9fc7925c80ff2f3949e1cc340a4a0d36fcd4161c/src/ethereum/cancun/fork.py#L329) | | | | | |
| $H_{difficulty} = 0 $ | validate_header-> [ensure](https://github.com/ethereum/execution-specs/blob/9fc7925c80ff2f3949e1cc340a4a0d36fcd4161c/src/ethereum/cancun/fork.py#L327) | | | | | |
| $H_{nonce} = 0x0000000000000000 $ | validate_header-> [ensure](https://github.com/ethereum/execution-specs/blob/9fc7925c80ff2f3949e1cc340a4a0d36fcd4161c/src/ethereum/cancun/fork.py#L328) | | | | | |
| $H_{prevRandao} = PREVRANDAO() $ (这已过时，信标链现在提供此值) | | | | | | |
| $H_{withdrawalHash} \neq nil $ | | | | | | |
| $H_{blobGasUsed} \neq nil $ | | | | | | |
| $H_{blobGasUsed} \leq MaxBlobGasPerBlock_{=786432} $ | | | | | | |
| $H_{blobGasUsed} \% GasPerBlob_{=2^{17}} = 0 $ | | | | | | |
| $H_{excessBlobGas} = CalcExcessBlobGas(P(H)_H) $ | [ensure](https://github.com/ethereum/execution-specs/blob/9fc7925c80ff2f3949e1cc340a4a0d36fcd4161c/src/ethereum/cancun/fork.py#L187) | | | | | |

### 下载器 (Downloader)

### 交易池 (Transaction Pools)

在以太坊中，主要识别两类交易池：

1. **遗留交易池 (Legacy Pools)**：由执行客户端管理，这些交易池采用按价格排序的堆或优先级队列来根据交易的价格组织交易。具体而言，使用两个堆来安排交易：一个优先考虑下一个区块的有效小费 (Effective Tip)，另一个专注于 Gas 费用上限。在饱和期内，会选择这两个堆中较大的一个来驱逐交易，从而优化交易池的效率和响应速度。[紧急堆和浮动堆规格 (urgent and floating heaps)](https://github.com/ethereum/go-ethereum/blob/064f37d6f67a012eea0bf8d410346fb1684004b4/core/txpool/legacypool/list.go#L525)

2. **Blob 交易池 (Blob Pools)**：与遗留池不同，Blob 池维护一个用于交易驱逐的优先级堆，但包含了独特的运行机制。值得注意的是，Blob 池的实现有详尽的文档记录，并附带可供审查的广泛注释部分，可在[这里](https://github.com/ethereum/go-ethereum/blob/064f37d6f67a012eea0bf8d410346fb1684004b4/core/txpool/blobpool/blobpool.go#L132)查看。Blob 池的一个关键特征是在其驱逐队列中使用对数函数。

请注意，这些示例使用的是 go-ethereum，在不同的客户端中，具体的命名和实现细节可能会有所不同，但基本原理保持不变。

### 以太坊虚拟机 (EVM)

[维基 - EVM (Wiki - EVM)](/wiki/EL/evm.md)
待办：将相关代码从规范移入 EVM

### DevP2P

[维基 - DevP2P (Wiki - DevP2P)](/wiki/EL/devp2p.md)

### 数据结构 (Data structures)

更多细节在关于 [执行层数据结构 (EL data structures)](/wiki/EL/data-structures.md) 的页面中。

### 存储和数据库后端 (Storage and database backends)

由执行客户端处理的区块链和状态数据需要存储在磁盘中。这对于验证新区块、核实历史记录以及服务网络中的对等方都是必要的。客户端存储历史数据，也称为古老数据库 (Ancient Database)，其中包括以前的区块。另一个具有树状结构的数据库包含当前状态和少量的近期状态。在实践中，客户端为不同的数据类别保存各种数据库。每个客户端可以实现不同的后端来处理这些数据，例如 LevelDB、Pebble、MDBX。

**Leveldb**

LevelDB 是客户端使用的数据库实现之一，它提供通用的有序键值接口，并且对以太坊特有的结构一无所知。

LevelDB 是一个基于日志结构合并树 (Log-structured Merge Tree, LSM-tree) 设计的嵌入式键值数据库。写入首先附加到预写日志 (Write-ahead Log) 中以进行崩溃恢复，并插入到内存中的 memtable（内存表）中。当 memtable 满了，它会被刷新到磁盘上，作为一个不可变的排序字符串表 (Sorted String Table, SSTable)。在磁盘上，这些表被组织成多个级别，并通过后台压缩定期合并。这种设计有利于高顺序写入吞吐量，但当数据被频繁更新时，会导致写入放大 (Write Amplification) 和波动的延迟。

执行客户端大多已经从这个特定的数据库实现转向更有源支持的现代重新实现，或者是具有改进性能的更具实验性的设计。

**Pebble**

Pebble 被一些执行客户端用作 LevelDB 的替代品，履行作为区块链数据、执行状态和索引的主要嵌入式键值存储的相同职责。Geth 已经选择 Pebble 作为替代品，并且现在是默认的后端，因为 LevelDB 停止了维护。随着 Pebble 的持续开发，它以某些优势提供了相同的功能集。

与 LevelDB 相比，Pebble 保留了 LSM 树架构，但针对以太坊执行中典型的重度写入、延迟敏感的工作负载进行了改进。它支持多个活动 memtable 以减少写入停顿，公开了详细的 SSTable 和压缩元数据，并对压缩行为提供了显著更多的控制。这些改进允许客户端更好地管理写入放大，并在频繁的状态更新下获得更可预测的性能。Pebble 还提供了更强的批处理语义和快照支持，使执行客户端能够更好地使数据库操作与区块执行和并发的 RPC 读取保持一致。

https://github.com/cockroachdb/pebble

**MDBX**

阅读关于其[特性/功能 (features)](https://github.com/erthink/libmdbx#features)的更多内容。此外，BoltDB 有一个关于与 leveldb 等其他数据库对比的页面，可在[这里](https://github.com/etcd-io/bbolt#comparison-with-other-databases)查看。Bolt 上提到的对比点也适用于 MDBX。

### 资源和参考资料 (Resources and References)

- [Engine Api 规范 (Engine Api Spec)](https://github.com/ethereum/execution-apis/blob/main/src/engine/paris.md#payload-validation) • [已归档](https://web.archive.org/web/20250318111700/https://github.com/ethereum/execution-apis/blob/main/src/engine/paris.md#payload-validation)
- [《引擎 API：视觉指南》 (Engine API: A Visual Guide)](https://hackmd.io/@danielrachi/engine_api) • [已归档](https://web.archive.org/web/20241006232802/https://hackmd.io/@danielrachi/engine_api)
- [Engine API | Mikhail | 第 21 讲 (Engine API | Mikhail | Lecture 21)](https://youtu.be/fR7LBXAMH7g)
- [《快照快照同步：针对 Go-Ethereum 同步节点的实用攻击》(苏黎世联邦理工学院) ("Snapping Snap Sync: Practical Attacks on Go Ethereum Synchronising Nodes" (ETH Zurich))](https://appliedcrypto.ethz.ch/content/dam/ethz/special-interest/infk/inst-infsec/appliedcrypto/research/TavernaPaterson-SnappingSnapSync.pdf)
- [Geth 文档 – 同步模式 (Geth Docs – Sync Modes)](https://geth.ethereum.org/docs/fundamentals/sync-modes?utm_source=chatgpt.com) • [已归档](https://web.archive.org/web/20240505050000/https://geth.ethereum.org/docs/fundamentals/sync-modes)
- [YouTube 视频 – 《如何使用快照同步来同步以太坊节点》 (YouTube – "How to Sync an Ethereum Node with Snap Sync")](https://www.youtube.com/watch?v=fk50UbUgkMM)
- [Ethereum.org – 执行层同步模式 (Ethereum.org – Execution Layer Sync Modes)](https://ethereum.org/en/developers/docs/nodes-and-clients/#execution-layer-sync-modes) • [已归档](https://web.archive.org/web/20240507022042/https://ethereum.org/en/developers/docs/nodes-and-clients/#execution-layer-sync-modes)
