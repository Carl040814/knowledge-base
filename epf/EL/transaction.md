# 交易 (Transaction)

**交易 (transaction)** 是由**外部账户 (external account, EOA)** 签发的、经过密码学签名的指令，通过 [JSON-RPC 协议](/wiki/EL/JSON-RPC.md) 提交给执行层客户端 (execution client)，然后使用 [DevP2P (以太坊点对点开发协议)](/wiki/EL/devp2p) 广播给整个网络。

一笔交易包含以下字段：

- **随机数 nonce ($T_n$)**：一个整数值，等于发送者发送的交易数量。Nonce 用于：

  - **防止重放攻击 (Prevent replay attack)**：假设 Alice 在一笔交易中向 Bob 发送了 1 ETH，Bob 可能会尝试将同一笔交易重新广播到网络中，以从 Alice 的账户中获取额外的资金。由于该交易是用唯一的 nonce 签名的，如果 Bob 再次发送，以太坊虚拟机 (EVM) 将直接拒绝它。从而保护 Alice 的账户免受未经授权的重复交易的影响。
  - **确定合约账户地址**：在“合约创建 (contract creation)”模式下，nonce 与发送者的地址一起用于确定合约账户地址。
  - **替换交易**：当一笔交易由于 Gas 价格过低而卡住时，矿工/验证者通常允许具有相同 nonce 的替换交易。一些钱包可能会利用这种行为提供取消交易的选项。实质上，发送了一笔具有相同 nonce、更高 Gas 价格且转账金额为 0 的新交易，从而有效地覆盖了原始的未决交易。然而，必须明确的是，替换未决交易并不一定能保证成功，因为它依赖于验证者的行为和网络状况。

- **Gas 价格 gasPrice ($T_p$)**：一个整数值，等于为每单位 Gas 支付的 **维 (Wei)** 的数量。**Wei** 是以太币的最小单位。$1 \textnormal{ETH} = 10^{18} \textnormal{Wei}$。Gas 价格用于对交易的执行进行优先级排序。Gas 价格越高，验证者就越有可能将该交易纳入区块。

- **Gas 限制 gasLimit ($T_g$)**：一个整数值，等于执行此交易要使用的最大 Gas 数量。如果 gasLimit 耗尽，此交易的执行将停止。

- **接收地址 to ($T_t$)**：此交易接收者的 20 字节地址。`to` 字段还决定了交易的模式或目的：

| `to` 的值        | 交易模式           | 描述                                     |
| ---------------- | ------------------ | ---------------------------------------- |
| _空 (Empty)_     | 合约创建           | 交易创建一个新的合约账户。               |
| 外部拥有账户 (EOA) | 以太币转账         | 交易将以太币转移到外部账户。             |
| 合约账户         | 合约执行           | 交易调用现有的智能合约代码。             |

- **转账金额 value ($T_v$)**：一个整数值，等于要转移给此交易接收者的 Wei 的数量。在“合约创建”模式下，value 成为新创建合约账户的初始余额。

- **数据 data ($T_d$) 或初始化 init ($T_i$)**：一个无限制大小的字节数组，指定以太坊虚拟机 (EVM) 的输入。在合约“创建模式”下，此值被视为“初始化字节码 (init bytecode)”，否则为“输入数据”的字节数组。

- **签名 Signature ($T_v, T_r, T_s$)**：发送者的 [ECDSA (椭圆曲线数字签名算法)](/wiki/Cryptography/ecdsa.md) 签名。

---

## 合约创建 (Contract creation)

让我们将以下代码部署到一个新的合约账户上：

```bash
[00] PUSH1 06 // 推入值 06
[02] PUSH1 07 // 推入值 07
[04] MUL      // 乘法运算
[05] PUSH1 0  // 推入值 00 (存储地址)
[07] SSTORE   // 将结果存储到存储插槽 00
```

括号表示指令偏移量。对应的字节码 (bytecode)：

```bash
6006600702600055
```

现在，让我们准备交易的 `init` 值以部署此字节码。Init 实际上由两个片段组成：

```
<初始化字节码 / init bytecode> <运行字节码 / runtime bytecode>
```

`init` 代码在账户创建时由 EVM 仅执行一次。运行初始化代码的返回值是**运行字节码 (runtime bytecode)**，它存储为合约账户的一部分。每当合约账户接收到交易时，都会执行运行字节码。

让我们准备我们的初始化代码，使其返回我们的运行代码：

```bash
// 1. 复制到内存
[00] PUSH1 08 // PUSH1 08 (我们运行代码的长度)
[02] PUSH1 0c // PUSH1 0c (运行代码在 init 中的偏移量)
[04] PUSH1 00 // PUSH1 00 (内存中的目标地址)
[06] CODECOPY // 将当前环境中运行的代码复制到内存中
// 2. 从内存返回
[07] PUSH1 08 // PUSH1 08 (返回数据的长度)
[09] PUSH1 00 // PUSH1 00 (返回数据的内存起始位置)
[0b] RETURN   // 返回运行代码并停止执行
// 3. 运行代码 (8 字节长)
[0c] PUSH1 06
[0e] PUSH1 07
[10] MUL
[11] PUSH1 0
[13] SSTORE
```

该代码执行了两个简单的操作：首先，将运行字节码复制到内存，然后从内存返回运行字节码。

`init` 字节码：

```javascript
6008600c60003960086000f36006600702600055
```

接下来，准备交易有效负载：

```javascript
[
  "0x", // nonce (零 nonce，因为是首笔交易)
  "0x77359400", // gasPrice (我们为每单位 Gas 支付 2000000000 wei)
  "0x13880", // gasLimit (80000 是部署的标准 Gas 限制)
  "0x", // to address (在合约创建模式下为空)
  "0x05", // value (我们很慷慨，向我们的新合约发送 5 wei)
  "0x6008600c60003960086000f36006600702600055", // init 代码
];
```

> 有效负载中各值的顺序至关重要！

对于本例，我们将使用 [Foundry (以太坊智能合约开发工具链)](https://getfoundry.sh/) 在本地部署交易。Foundry 是一个以太坊开发工具套件，提供了以下命令行工具：

- **Anvil**：专为开发设计的本地以太坊节点。
- **Cast**：用于执行以太坊 RPC 调用的工具。

安装并启动 [anvil](https://book.getfoundry.sh/anvil/) 本地节点：

```
$ anvil


                              _   _
                             (_) | |
       __ _   _ __   __   __  _  | |
      / _` | | '_ \  \ \ / / | | | |
     | (_| | | | | |  \ V /  | | | |
      \__,_| |_| |_|   \_/   |_| |_|

    0.2.0 (5c3b075 2024-03-08T00:17:08.007462509Z)
    https://github.com/foundry-rs/foundry

Available Accounts
==================

(0) "0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266" (10000.000000000000000000 ETH)
.....

Private Keys
==================

(0) 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
.....
Listening on 127.0.0.1:8545
```

使用 anvil 的虚拟账户之一签署交易：

```bash
$ node sign.js '[ "0x", "0x77359400", "0x13880", "0x", "0x05", "0x6008600c60003960086000f36006600702600055" ]' ac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

f864808477359400830138808005946008600c60003960086000f360066007026000551ca01446316c9bdcbe0cb87fac0b08a00e59552634c96d0d6e2bd522ea0db827c1d0a0170680b6c348610ef150c1b443152214203c7f66288ea6332579c0cdfa86cc3f
```

> 请参阅下文的**附录 A** 以获取 `sign.js` 辅助脚本的源代码。

最后，使用 [cast](https://book.getfoundry.sh/cast/) 提交交易：

```javascript
$ cast publish f864808477359400830138808005946008600c60003960086000f360066007026000551ca01446316c9bdcbe0cb87fac0b08a00e59552634c96d0d6e2bd522ea0db827c1d0a0170680b6c348610ef150c1b443152214203c7f66288ea6332579c0cdfa86cc3f

{
  "transactionHash": "0xdfaf2817f19963846490b330ae33eba7b42872e8c8bd111c8d7ea3846c84cd51",
  "transactionIndex": "0x0",
  "blockHash": "0xfde1475a716583d847f858c5db3e54156983b39e3dbefaa5829416e6e60a788a",
  "blockNumber": "0x1",
  "from": "0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266",
  "to": null,
  "cumulativeGasUsed": "0xd67e",
  "gasUsed": "0xd67e",
  // 新创建的合约地址 👇
  "contractAddress": "0x5fbdb2315678afecb367f032d93f642f64180aa3",
  "logs": [],
  "status": "0x1",
  "logsBloom": "0x0...",
  "effectiveGasPrice": "0x77359400"
}
```

查询本地 `anvil` 节点确认代码已部署：

```bash
$ cast code 0x5fbdb2315678afecb367f032d93f642f64180aa3
0x6006600702600055
```

并且初始余额已生效：

```bash
$ cast balance 0x5fbdb2315678afecb367f032d93f642f64180aa3
5
```

---

合约创建的模拟过程：

![Contract creation](https://epf.wiki/images/evm/create-contract.gif)

---

## 合约代码执行 (Contract code execution)

我们简单的合约将 6 和 7 相乘，然后将结果存入**存储插槽 0 (storage slot 0)**。让我们用另一笔交易来执行合约代码。

交易的有效负载类似，只是 `to` 地址指向该智能合约，且 `value` 和 `data` 为空：

```javascript
[
  "0x1", // nonce (加 1)
  "0x77359400", // gasPrice (我们为每单位 Gas 支付 2000000000 wei)
  "0x13880", // gasLimit (80000 是部署的标准 Gas 限制)
  "0x5fbdb2315678afecb367f032d93f642f64180aa3", // to address (我们的智能合约地址)
  "0x", // value (空；不发送任何以太币)
  "0x", // data (空)
];
```

签署交易：

```bash
$ node sign.js '[ "0x1", "0x77359400", "0x13880", "0x5fbdb2315678afecb367f032d93f642f64180aa3", "0x", "0x"]' ac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

f86401847735940083013880945fbdb2315678afecb367f032d93f642f64180aa380801ba047ae110d52f7879f0ad214784168406f6cbb6e72e0cab59fa4df93da6494b578a02c72fcdea5b7838b520664186707d1465596e4ad4eaf8781a721530f8b8dd5f2
```

发布交易：

```bash
$ cast publish f86401847735940083013880945fbdb2315678afecb367f032d93f642f64180aa380801ba047ae110d52f7879f0ad214784168406f6cbb6e72e0cab59fa4df93da6494b578a02c72fcdea5b7838b520664186707d1465596e4ad4eaf8781a721530f8b8dd5f2

{
  "transactionHash": "0xc82a658b947c6083de71a0c587322e8335448e65e7310c04832e477558b2b0ef",
  "transactionIndex": "0x0",
  "blockHash": "0x40dc37d9933773598094ec0147bef5dfe72e9654025bfaa80c4cdbf634421384",
  "blockNumber": "0x2",
  "from": "0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266",
  "to": "0x5fbdb2315678afecb367f032d93f642f64180aa3",
  "cumulativeGasUsed": "0xa86a",
  "gasUsed": "0xa86a",
  "contractAddress": null,
  "logs": [],
  "status": "0x1",
  "logsBloom": "0x0...",
  "effectiveGasPrice": "0x77359400"
}
```

使用 cast 读取存储**插槽 0** 的值：

```bash
$ cast storage 0x5fbdb2315678afecb367f032d93f642f64180aa3 0x
0x000000000000000000000000000000000000000000000000000000000000002a
```

果然，结果正是 [42](<https://simple.wikipedia.org/wiki/42_(answer)>) (0x2a) 🎉。

---

合约执行的模拟过程：

![Contract execution](https://epf.wiki/images/evm/contract-execution.gif)

---

## 交易收据 (Receipts)

收据 (Receipts) 是以太坊虚拟机 (EVM) 状态转换函数的输出制品。正如维基的[数据结构](./data-structures.md#receipt-trie)部分所述，每笔成功或失败执行的交易都会产生相应的收据。在此，我们将提供有关收据结构及其演进的额外细节。

一个 `receipt (收据)` 的内容是由五个项目组成的元组：
- **交易类型 (Transaction Type)**：这用于区分传统交易和有类型交易，稍后将进行更详细的讨论。
- **状态 (Status)**：交易状态为 `0` 或 `1`，其中 `1` 表示交易成功，`0` 表示交易失败。
- **累积 Gas 使用量 (Gas Used)**：区块中所有先前交易消耗的 Gas 总量 + 当前交易消耗的 Gas 量。
- **日志 (Logs)**：日志条目是由记录器地址、可能为空的一系列已索引的 32 字节日志主题 (topics) 以及若干个未索引的原始事件数据字节组成的元组。
- **日志布隆过滤器 (Logs Bloom)**：一个 256 字节的布隆过滤器，用于快速搜索区块中的相关日志，这允许应用程序高效地检查日志中是否包含了某个地址或事件签名。

有关如何使用日志布隆过滤器允许应用程序高效检查日志中是否包含地址或事件签名的更多信息，可以在 [此处](https://medium.com/coinmonks/ethereum-data-transaction-receipt-trie-and-logs-simplified-30e3ae8dc3cf#:~:text=the%20sections%20below.-,Logs%20Bloom,-Assume%20we%20want) 找到。

`receipt` 被提交到区块的 **收据树 (Receipt Trie)** 中。

---

## 有类型交易与收据 (Typed Transactions and Receipts)

[EIP-2718](https://eips.ethereum.org/EIPS/eip-2718) 通过“有类型信封 (typed envelopes)”的概念，为交易和收据引入了统一且可扩展的格式。这种扩展简化了新交易和收据类型的引入，同时保持了与传统交易的完全向后兼容性。

在 EIP-2718 之前，添加新的交易类型需要使用繁琐的技术在 RLP 编码的限制内对它们进行区分，这导致了脆弱的设计。EIP-2718 通过定义专用的***交易类型 (Transaction Type)*** 前缀解决了这个问题。

EIP-2718 之后的交易遵循信封格式：`有类型交易 (Typed Transaction) = 交易类型 (Transaction Type) + 交易有效负载 (Transaction Payload)`

其中：
- ***交易类型 (Transaction Type)***：一个单字节标识符，指定交易的类型。
- ***交易有效负载 (Transaction Payload)***：由相应交易类型规范定义的非透明字节数组。

请注意，传统交易的格式为 `RLP([nonce, gasPrice, ..., s])`。

### 收据编码 (Receipt Encoding)

收据现在采用了相同的信封模式：`有类型收据 (Typed Receipt) = 交易类型 (Transaction Type) + 收据有效负载 (Receipt Payload)`

其中：
- ***交易类型 (Transaction Type)***：一个单字节标识符，指定交易的类型。
- ***收据有效负载 (Receipt Payload)***：根据关联的***交易类型 (Transaction Type)*** 定义进行解释。

请注意，传统收据的格式为 `RLP([status, gasUsed, bloom, logs])`。

交易和收据都可以被高效地识别：
- 如果第一个字节 `∈ [0x00, 0x7f]`，它是一个**有类型 (typed)** 交易或收据。
- 如果第一个字节 `≥ 0xc0`，它是一个**传统 (legacy)** 交易或收据，这由 RLP 列表编码决定。

> 有类型收据的第一个字节**必须**与其关联交易的 `TransactionType` 相同。

此规则确保了客户端可以在不需要额外元数据的情况下确定性地对收据进行解码。

总之，EIP-2718 使以太坊交易和收据更具可扩展性，同时保留了与传统客户端的向后兼容性。

---

## 交易类型概述 (Overview of Transaction Types)

### 传统交易 (Legacy Transaction - 类型 0x00)
传统交易是在 [EIP-2718](https://eips.ethereum.org/EIPS/eip-2718) 引入交易类型概念之前的唯一交易格式。此类交易在以太坊中仍然兼容。即使在进行 [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) 等升级后，传统交易也会通过将 `max_fee_per_gas` 和 `max_priority_fee_per_gas` 设置为传统交易的 `gas_price`，来转换为 EIP-1559 兼容交易。

### 访问列表交易 (Access List Transaction - 类型 0x01)
访问列表交易类似于传统交易，但它们还包含一个可选的 `access_list` 字段，其中列出了运行该交易所需的所有地址和存储插槽。

*动机 (Motivation)*

在 2016 年，攻击者在被称为“上海拒绝服务 (DoS) 攻击”的事件中针对网络发起了攻击——最成功的策略是发送访问大量地址和存储插槽的恶意交易。该攻击迫使客户端在磁盘上搜索信息，导致处理时间极长的 I/O 密集型交易。这种攻击对攻击者而言成本极低，但对客户端而言成本极高。

因此，提出了 [EIP-2929](https://eips.ethereum.org/EIPS/eip-2929) 以增加“冷”（首次访问，在数据被复制到 RAM 之前）访问存储的操作码的 Gas 成本。这将使进一步的 DoS 攻击在经济上变得难以承受。

由于增加了存储访问的 Gas 成本，EIP-2929 引入了潜在的合约损坏风险。因此，[EIP-2930](https://eips.ethereum.org/EIPS/eip-2930) 引入了访问列表（`access_list` 字段），指定交易中要访问的所有账户和存储插槽。结果，客户端现在可以预先加载数据，从而为包含访问列表的交易降低了 Gas 成本。

### 动态费用交易 (Dynamic-Fee Transaction - 类型 0x02)
类型 2 交易带来了在 [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) 中描述并在伦敦分叉中引入的新费用市场变革。网络将试图通过调整 Gas 基础费用 (base fee)，将每个区块的目标 Gas 使用量维持在最大容量的 50%。如果当前的 Gas 使用量高于目标，则调高 Gas 价格以降低需求。相反，如果区块没有使用足够的 Gas，则降低 Gas 基础费用进行补偿，以刺激需求。然后基础费用将被销毁 (burn)，从而减少以太币 (Eth) 的总供应量。

基础费用增加或减少的一个关键方面是，这种变化只能以父区块 Gas 限制的最多 1/1024 进行递增/递减。这确保了 Gas 价格能够对需求变化做出快速反应，同时又防止了剧烈的波动。

此外，交易可以包含额外的优先费用 (priority fee)，以激励特定交易在区块中优先于其他交易被打包。这种优先费用不会被销毁，而是支付给验证者。

*动机 (Motivation)*

为什么要首先改变 Gas 费用市场？
在 EIP-1559 之前，Gas 价格随着网络需求而剧烈波动。常见的情况是发送了具有特定 Gas 价格的交易，然后由于 Gas 价格意外暴涨而导致交易卡在内存池 (mempool) 中。这导致了糟糕的用户体验，而新引入的逐步调整基础费用的机制解决了这一问题。

为什么要销毁基础费用？
除了通过提供减少供应和抵消通胀的机制来为以太币 (Eth) 持有者提供经济利益之外，销毁基础费用还降低了费用操纵的风险，防止验证者向区块中充斥“免费”交易并人为保持高 Gas 价格。

### 携带 Blob 交易 (Blob-Carrying Transaction - 类型 0x03)
<br>
<img src="https://epf.wiki/images/el-transactions/blob.png" alt="blob" />

[EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) 引入了短期的 blob 数据存储，网络会在临时时间段内（目前为 4096 个纪元，约 18 天）保持这些数据的可用性。

Blob 不存储在区块链上，也不在区块头或区块体中。相反，blob 数据是与区块一起发送的——就像侧车 (sidecar) 一样——连同交易数据一起被节点使用。区块头将仅包含每个数据 blob 的 KZG 承诺 (KZG commitment)。

与 EIP-1559 交易类似（尽管数学公式不同），blob 交易拥有自己的 Gas 价格市场，该市场根据需求进行调整，以达到总容量 50% 的目标使用率。

*动机 (Motivation)*

在以太坊中，可扩展性的长期愿景包括 Layer 1（执行层和共识层）和 Layer 2 链协同工作，Layer 1 为 Layer 2 提供安全保证。因此，Layer 2 是以太坊中的一等公民。

我们关心 blob 数据的可用性 (data availability)，因为维护临时可用的数据对于服务 Layer 2 是必不可少的。例如，像 Optimism 这样的乐观卷起 (optimistic rollups) 会乐观地在链上推送交易批次的承诺。交易可能会被恶意遗漏（例如，Bob 拥有 10 eth，但承诺的交易显示他有 0 eth）。如果是这种情况，资金受损的用户可以发布欺诈证明 (fraud proof) 来证明不良行为并纠正状况。创建此欺诈证明需要访问交易数据，因此以太坊将这些数据作为 blob 短期提供，让用户有合理的时间做出反应。因此，数据可用性对于确保 Layer 2 的合规状态过渡 (state transition) 是必不可少的。

### 为 EOA 设置代码交易 (Set Code Transaction for EOAs - 类型 0x04)
类型 4 交易旨在将智能合约代码附加到外部拥有账户 (EOA)。回想一下，EOA 的 `code_hash` 字段为空，因此它们通常自身没有任何程序。[EIP-7702](https://eips.ethereum.org/EIPS/eip-7702) 通过将该值设置为 `(0xef0100 || 委托合约地址)` 的值来改变了这一点，其中 `||` 代表拼接。一旦设置了代码，EOA 就可以将其代码执行委托给保存的地址。需要指出的是，EOA 本身并不持有代码，而只是一个指向代码的指针。此指针可以通过另一笔类型 4 交易进行移除或更改。因为我们可以区分 EOA 和合约，所以防止攻击者产生与现有合约冲突的地址并窃取资金的 [EIP-3607](https://eips.ethereum.org/EIPS/eip-3607) 仍然可以得到遵守。

选择在委托合约地址之前的数值 `0xef0100` 是为了提供一个不会发生冲突的唯一标识符。
实际上，根据 [EIP-3541](https://eips.ethereum.org/EIPS/eip-3541)，`0xef` 是一个保留字节。之后添加的 `0100` 是表示 EIP-7702 委托地址的标识符。因此，仍然可以通过查看代码哈希来区分智能合约和 EOA，因为 EOA 要么什么都没有，要么是一个以 `0xef0100` 开头的值。

*动机 (Motivation)*

没有 EIP-7702 的纯 EOA 账户面临几个用户体验 (UX) 问题。例如，通过智能合约发送 ERC-20 代币需要通过 `approve/transferFrom` 模式，这需要两笔独立的交易。在 [ERC-4337](https://eips.ethereum.org/EIPS/eip-4337) 中描述的账户抽象 (account abstraction)（智能合约钱包）旨在解决此问题，此外还提供了交易打包、Gas 赞助等额外功能。

通过 EIP-7702，以太坊生态系统现在可以通过允许 EOA 选择性加入 (opt-in) 来从账户抽象中受益。

---

## 附录 A：交易签名器 (Transaction signer)

`signer.js`：一个用于签署交易的简单 [node.js](https://nodejs.org/) 脚本。解释请参阅注释：

```javascript
/**
 * 用于签署交易有效负载数组的实用脚本。
 * 用法：node sign.js '[有效负载]' [私钥]
 */

const { rlp, keccak256, ecsign } = require("ethereumjs-util");

// 解析命令行参数
const payload = JSON.parse(process.argv[2]);
const privateKey = Buffer.from(process.argv[3].replace("0x", ""), "hex");

// 验证私钥长度
if (privateKey.length != 32) {
  console.error("私钥必须为 64 个字符长！");
  process.exit(1);
}

// 步骤 1: 将有效负载进行 RLP 编码
// 了解更多：https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp/
const unsignedRLP = rlp.encode(payload);

// 步骤 2: 对 RLP 编码后的有效负载进行哈希
// 了解更多：https://ethereum.org/en/glossary/#keccak-256
const messageHash = keccak256(unsignedRLP);

// 步骤 3: 对消息进行签名
// 了解更多：https://epf.wiki/#/wiki/Cryptography/ecdsa
const { v, r, s } = ecsign(messageHash, privateKey);

// 步骤 4: 将签名附加到有效负载中
payload.push(
  "0x".concat(v.toString(16)),
  "0x".concat(r.toString("hex")),
  "0x".concat(s.toString("hex"))
);

// 步骤 5: 输出 RLP 编码后的已签名交易
console.log(rlp.encode(payload).toString("hex"));
```

---

## 相关资源 (Resources)
- 📝 Gavin Wood, ["以太坊黄皮书" (Ethereum Yellow Paper).](https://ethereum.github.io/yellowpaper/paper.pdf)
- 📘 Andreas M. Antonopoulos, Gavin Wood, ["精通以太坊" (Mastering Ethereum).](https://github.com/ethereumbook/ethereumbook)
- 📝 Ethereum.org, ["RLP 编码" (RLP Encoding).](https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp/)
- 📝 Ethereum.org, ["交易" (Transactions).](https://ethereum.org/en/developers/docs/transactions/)
- 📝 Random Notes, ["如何以困难的方式签名交易" (Signing transactions the hard way).](https://lsongnotes.wordpress.com/2018/01/14/signing-an-ethereum-transaction-the-hard-way/) • [archived](https://web.archive.org/web/20240229045603/https://lsongnotes.wordpress.com/2018/01/14/signing-an-ethereum-transaction-the-hard-way/)
- 🎥 Lefteris Karapetsas, ["理解 EVM 兼容区块链中的交易" (Understanding Transactions in EVM-Compatible Blockchains).](https://archive.devcon.org/archive/watch/6/understanding-transactions-in-evm-compatible-blockchains-powered-by-opensource/?tab=YouTube)
- 🎥 Austin Griffith, ["交易 - ETH.BUILD."](https://www.youtube.com/watch?v=er-0ihqFQB0)
- 🧦 Paradigm, ["Foundry：以太坊开发工具套件" (Foundry: Ethereum development toolkit).](https://github.com/foundry-rs/foundry)
- [有线协议中的收据 (Receipts in Wire Protocol)](https://github.com/ethereum/devp2p/blob/master/caps/eth.md) • [archived](https://web.archive.org/web/20250328095848/https://github.com/ethereum/devp2p/blob/master/caps/eth.md)
- [EIP-2718](https://eips.ethereum.org/EIPS/eip-2718) • [archived](https://web.archive.org/web/20250328095848/https://eips.ethereum.org/EIPS/eip-2718)
- [收据内容 (Receipt Contents)](https://medium.com/coinmonks/ethereum-data-transaction-receipt-trie-and-logs-simplified-30e3ae8dc3cf) • [archived](https://web.archive.org/web/20250000000000/https://medium.com/coinmonks/ethereum-data-transaction-receipt-trie-and-logs-simplified-30e3ae8dc3cf)
