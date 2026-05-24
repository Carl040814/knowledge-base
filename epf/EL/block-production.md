# 区块构建、处理以及将交易应用至状态 (Block building, processing, and applying transaction to state):

## 简介 (Intro)

区块构建 (Block Building) 是以太坊区块链 (Ethereum Blockchain) 功能中至关重要的任务，涉及决定验证者 (Validator) 在提议区块之前如何获取该区块的各种流程。以太坊网络 (Ethereum Network) 由运行相互连接的共识客户端 (Consensus Clients, CL) 和执行客户端 (Execution Clients, EL) 的节点组成。两者对于参与网络并在每个时隙 (Slot) 生产区块都至关重要。虽然执行客户端 (EL) 拥有许多重要功能（您可以在 "el-architecture" 页面中了解更多），但这里的主要关注点是其在构建供共识层 (CL) 消耗的区块方面的作用。

当验证者在某个时隙被选中提议区块时，它会寻找由共识层 (CL) 生产的区块。重要的是，验证者并不局限于仅广播来自其自身执行客户端 (EL) 的区块。它还可以广播由外部构建者生产的区块；有关详细信息，请参阅 [提议者-构建者分离 (Proposer-Builder Separation, PBS)](https://ethereum.org/en/roadmap/pbs/)。本文专门探讨了区块是如何由执行客户端 (EL) 生产的，以及有助于其成功生产和交易执行 (Transaction Execution) 的要素。

## 有效载荷构建例程 (Payload building routine)

当共识层 (CL) 通过引擎 API (Engine API) 的分叉选择更新 (Fork Choice Updated) 端点指示执行层客户端 (EL Client) 构建区块时，区块便被创建，进而通过有效载荷构建例程 (Payload Building Routine) 启动区块构建过程。

注意：构建的有效载荷 (Payload) 的费用接收者 (Fee Recipient) 可能会与有效载荷属性 (Payload Attributes) 中建议的费用接收者不同：

<img src="https://epf.wiki/images/el-architecture/payload-building-routine.png" width="1000"/>

节点通过点对点网络使用传闻协议 (Gossip Protocol) 广播交易。这些交易会根据特定标准进行验证（例如，检查随机数 (Nonce) 正确性、余额是否充足以及签名是否正确），并存储在内存池 (Mempool) 中，等待被打包进区块。

每个时隙 (Slot) 都有一个指定的区块提议者 (Block Proposer)，由共识层通过伪随机过程选出。当验证者被选为某个时隙的区块提议者时，其共识客户端 (CL Client) 会通过执行引擎 (Execution Engine) 的分叉选择更新 (Fork Choice Updated) 方法启动区块构建，该方法为构建区块提供了必要的上下文。

我们可以简化并模拟构建区块的过程，尽管这种方法特定于 Geth 中使用的 Go 类型。然而，这些概念通常可以应用于不同的客户端。

```go
func build(env environment, pool txpool.Pool, state state.StateDB) (types.Block, state.StateDB, error) { //1
    var (
        gasUsed = 0
        txs []types.Transactions
    ) //2

  for ; gasUsed < 30_000_000 || !pool.Empty(); { //3
      transaction := pool.Pop() //4
      res, gas, err := vm.Run(env, transaction, state) //5
      if err != nil { // 6
          // transaction invalid
          continue
      }
      gasUsed += gas // 7
      transactions = append(transactions, transaction)
  }
  return core.Finalize(env, transactions, state) //8
}
```

1. 我们引入了环境 (Environment)，其中包含了所有必要的信息（类似于之前的区块头），包括时间戳 (Timestamp)、区块编号 (Block Number)、前一个区块 (Preceding Block)、基础费用 (Base Fee) 以及该区块中需要发生的所有提现 (Withdrawals)。本质上，源自作为中央决策实体的共识层 (CL) 的信息决定了构建区块的上下文。接下来，我们引入了交易池 (Transaction Pool)，它是交易的集合。为了简单起见，我们假设这些交易是根据其价值升序排列的。这种安排有助于我们为执行客户端构建最有利可图的区块（考虑到网络中观察到的交易）。此外，我们还考虑了状态数据库 (State DB)，它代表执行所有这些交易的状态。
   - 我们返回一个区块 (Block)、一个累积了区块中所有交易的状态数据库 (State DB)，以及可能的一个错误 (Error)。
2. 在 `build` 内部，我们跟踪已使用的 Gas (Gas Used)，因为我们可以使用的 Gas 数量是有限的。并且，还存储了将要放入区块中的所有交易。
3. 我们继续添加交易，直到交易池为空，或者消耗的 Gas 量大于 Gas 限制 (Gas Limit)——为了简单起见，在本例中该限制固定为 3000 万（大约是当前主网上的 Gas 限制）。
4. 为了获取交易，我们必须查询交易池，交易池被假定维护着一个有序的交易列表，以确保我们始终收到下一个最有价值的交易。
5. 交易在以太坊虚拟机 (Ethereum Virtual Machine, EVM) 中执行，假设 `run` 需要一个由区块和环境共同满足的接口。我们提供环境、交易和状态作为输入。这将在环境定义的上下文中执行交易，并向我们提供包含累积交易的更新后状态。
6. 如果交易执行不成功（由 `run` 期间发生错误表示），我们只需继续而不中断。这表明该交易是无效的，由于区块中仍有未使用的 Gas，我们不想立即生成错误。这是因为区块内尚未发生错误。然而，交易无效极有可能是因为它在执行期间做了一些坏事，或者是因为交易池稍微过时了。在这种情况下，我们允许自己继续并尝试将交易池中的下一笔交易放入此区块中。
7. 一旦我们验证了运行交易没有错误，我们就会将该交易添加到交易列表中，并将 `run` 返回的 Gas 添加到已使用的 Gas 中。例如，如果第一笔交易是一笔简单的转账（成本为 21,000 Gas），我们已使用的 Gas 将从 0 增加到 21,000，并且我们将继续重复此过程（步骤 3-7），直到满足步骤 3 的条件。
8. 我们通过使用一组交易和相关的区块信息来生成一个完整组装的区块，从而完成我们的状态过渡 (State Transition)。这样做的目的是在最后进行某些计算。由于区块头 (Block Header) 包含交易根 (Transactions Root)、收据根 (Receipts Root) 和提现根 (Withdrawals Root)，因此这些值必须通过对列表进行 Merkle 化 (Merkleizing) 来计算并添加到区块头中。
   - 我们返回我们的区块、状态数据库和我们的错误。

## 代码走读 (Code walk-through)

### Geth

以下示例使用 Geth 代码库来解释执行客户端是如何构建区块的。

 1. 首先，当验证者被选为区块构建者时，它会通过执行客户端 (EL) 上的引擎 API (Engine API) 调用 `engine_forkchoiceUpdatedV2` 函数。在此处，执行客户端启动区块构建过程。
    - https://github.com/ethereum/go-ethereum/blob/0a2f33946b95989e8ce36e72a88138adceab6a23/eth/catalyst/api.go#L398 
 2. 区块构建和交易执行的大部分核心逻辑都位于 Geth 的 `miner` 模块中。`buildPayload` 函数最初会创建一个空区块，这样节点就不会错过时隙 (Slot) 并且有东西可以提议。该函数实现还会启动一个 Go 协程 (Go Routine) 流程，其工作是潜在地填充我们留空的区块，然后并发地用填充的交易更新它。
    - https://github.com/ethereum/go-ethereum/blob/0a2f33946b95989e8ce36e72a88138adceab6a23/miner/payload_building.go#L180                                                                        - https://github.com/ethereum/go-ethereum/blob/0a2f33946b95989e8ce36e72a88138adceab6a23/miner/payload_building.go#L204
 3. 在 `buildPayload` 函数中，Go 协程正在等待多个通信操作（即 "cases"），在第一种情况下，它调用 `getSealingBlock`，其中的参数明确指定区块不应为空。查看 `fullParams` 变量，即 `noTxs:False`。
 4. 在 `getSealingBlock` 的定义中，请求正被发送到 `getWorkCh` 通道 (Channel)。该通道正被监听以从中检索数据并生成工作 (Work)。
    - [https://github.com/ethereum/goethereum/blob/master/miner/worker.go#L1222](https://github.com/ethereum/go-ethereum/blob/0a2f33946b95989e8ce36e72a88138adceab6a23/miner/worker.go#L1222)
 5. 此 `getWorkCh` 通道在同一文件中的 `mainLoop` 函数内被监听。来自 `getWorkCh` 通道的数据随后被发送到 `w.generateWork`。
    - https://github.com/ethereum/go-ethereum/blob/master/miner/worker.go#L537
 6. `generateWork` 函数是交易被填充进区块的地方。
    - https://github.com/ethereum/go-ethereum/blob/0a2f33946b95989e8ce36e72a88138adceab6a23/miner/worker.go#L1094
 7. `w.fillTransactions` 函数从内存池 (Mempool) 中检索所有待处理的交易并填充区块。这包括所有类型的交易，包括 Blob 交易 (Blobs)。
    - https://github.com/ethereum/go-ethereum/blob/master/miner/worker.go#L1024
 8. 交易根据其费用进行排序填充，然后传递给 `commitTransactions` 函数。  
https://github.com/ethereum/go-ethereum/blob/master/miner/worker.go#L1072
 9. `commitTransactions` 函数检查每笔交易是否有足够的剩余 Gas，然后提交该特定交易。此外，如 EIP-4844 中所指定，每个区块允许有一定数量的 Blob。  
      - https://github.com/ethereum/go-ethereum/blob/master/miner/worker.go#L888 
      - https://github.com/ethereum/EIPs/blob/master/EIPS/eip-4844.md
 10. 如果您查看 `commitTransaction` 函数，它会反过来调用 `w.applyTransaction` 函数。
     - https://github.com/ethereum/go-ethereum/blob/master/miner/worker.go#L760C18-L760C36 
 11. `applyTransaction` 函数接着调用核心包并调用 `core.ApplyTransaction`，该函数针对本地执行客户端 (EL) 状态执行所有交易。  
     - https://github.com/ethereum/go-ethereum/blob/master/miner/worker.go#L794.
 12. `ApplyTransaction` 函数针对本地执行客户端 (EL) 状态运行交易并执行所有状态更改。它创建 EVM 上下文 (EVM Contexts) 和环境以在 EVM 中执行交易。合约调用 (Contract Calls) 也全部发生在这里。如果一切顺利，状态便过渡成功。     
      - https://github.com/ethereum/go-ethereum/blob/master/core/state_processor.go#L161
 13. 此外，交易可能会失败。如果交易失败，它就不会应用于状态过渡。它可能由于链上原因失败，例如 Gas 耗尽、合约调用失败等。
 14. 从此时起，所有交易都会被逐一执行。然后，这些交易被捆绑到一个区块中。  
 15. 共识层 (CL) 随后通过引擎 API (Engine API) 向执行层 (EL) 发起请求以获取此填充了有效载荷的交易 `getPayload`。EL 向 CL 返回此有效载荷，CL 将此有效载荷放入信标区块 (Beacon Block) 并进行传播。
 
## 资源 (Resources)

1. [GETH 代码库 (GETH codebase)](https://github.com/ethereum/go-ethereum)
2. [《引擎 API：视觉指南》 (Engine API: A Visual Guide)](https://hackmd.io/@danielrachi/engine_api)
