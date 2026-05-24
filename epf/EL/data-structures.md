# 执行层中的数据结构 (Data Structures in Execution Layer)

执行客户端 (Execution Client) 存储当前状态 (Current State) 和历史区块链数据 (Historical Blockchain Data)。在实践中，以太坊数据存储在类似于树 (Trie) 的结构中，主要是默克尔·帕特里夏树 (Merkle Patricia Tree)。

## 递归长度前缀 (RLP)

[维基 - RLP (Wiki - RLP)](/wiki/EL/RLP.md)

## 默克尔树入门 (Primer on Merkle Tree)

默克尔树 (Merkle Tree) 是一种基于哈希的数据结构，在数据完整性 (Data Integrity) 和验证 (Verification) 方面非常高效。它是一种基于树的结构，其中叶子节点 (Leaf Nodes) 保存数据值，每个非叶子节点 (Non-leaf Node) 是其子节点哈希的哈希值。

默克尔树通过生成整个交易集的数字指纹来存储区块 (Block) 中的所有交易。它允许用户验证区块中是否包含某笔交易。默克尔树是通过反复计算节点对的哈希值来创建的，直到只剩下一个哈希值。这个哈希被称为**默克尔根 (Merkle Root)**，或根哈希 (Root Hash)。默克尔树是采用自底向上的方法构建的。

需要注意的是，默克尔树是一种**二叉树 (Binary Tree)**，因此它需要偶数个叶子节点。如果交易数量为奇数，最后一个哈希将被复制一次以创建偶数个叶子节点。

默克尔树提供了一种防篡改的结构来存储交易数据。哈希函数 (Hash Functions) 具有雪崩效应 (Avalanche Effect)，即数据的微小变化将导致生成的哈希发生巨大变化。因此，如果叶子节点中的数据被修改，根哈希将与预期值不匹配。
您也可以自己尝试使用 [SHA-256](https://emn178.github.io/online-tools/sha256.html) 哈希函数。
要了解有关哈希的更多信息，可以参考 [这里](https://github.com/ethereumbook/ethereumbook/blob/develop/04keys-addresses.asciidoc)。

默克尔根存储在**区块头 (Block Header)** 中。阅读关于以太坊内部区块结构的更多信息（*一旦相关文档准备就绪，将在此处链接*）。

主要父节点被称为根 (Root)，因此其中的哈希是根哈希 (Root Hash)。创建两个具有相同根哈希的非同状态的概率无限小（对于单个 SHA-256 哈希，概率为 1.16x10^77 分之一），任何尝试使用不同值修改状态的行为都会导致不同的状态根哈希。

下图描绘了默克尔树工作原理的简化版本：

- 叶子节点包含实际数据（为了简单起见，我们采用了数字）。
- 每个非叶子节点都是其子节点的哈希值。
- 第一层非叶子节点包含其子叶子节点的哈希值。
  `Hash(1,2)`
- 同样的过程一直持续到我们到达树的顶部，即所有先前哈希的哈希值。
  `Hash[Hash[Hash(1,2),Hash(3,4)],Hash[Hash(5,6),Hash(7,8)]]`

更多关于 [以太坊中的默克尔树 (Merkle Trees in Ethereum)](https://blog.ethereum.org/2015/11/15/merkling-in-ethereum)

![默克尔树 (Merkle Tree)](https://epf.wiki/images/merkle-tree.jpg)

## 帕特里夏树入门 (Primer on Patricia Tree)

帕特里夏树 (Patricia Tries，也称为基数树 (Radix Tree)) 是 n 叉树 (n-ary Trees)，与默克尔树不同，它们用于高效存储数据，而不是用于验证。

简单来说，帕特里夏树是一种树形数据结构，其中：
 - 每个节点的子节点数量最多为基数树的基数 r，其中对于某个整数 x ≥ 1，r = 2^x。
 - 与普通树不同，边 (Edges) 可以用字符序列（边标签 (Edge Labels)）进行标记，这使得结构的空间利用率高得多。
 - 每个作为唯一子节点的节点都会与其父节点合并，这比默克尔树等结构更节省空间。

一个简单的图示将展示帕特里夏树遍历是如何工作的。假设我们正在搜索与键 "romulus" 相关联的值。该值将保存在叶子节点中。
- 与普通树不同（普通树中数据可能存储在中间节点中），帕特里夏树仅在叶子节点中存储值。这提高了空间效率并使密码学验证变得更容易。
![帕特里夏树 (Patricia Trie)](https://epf.wiki/images/data-structures/patricia-trie.png)

1. 从根节点 (Root Node) 开始，它作为入口点，并包含 "r"，这将是所有存储键的起始前缀。
2. 遵循边标签（压缩路径表示），例如 "om"。帕特里夏树将公共前缀合并为单条边，而不是像标准树那样将每个字符存储在单独的节点中。这种前缀压缩 (Prefix Compression) 使树更加紧凑，从而提高了存储效率和查找效率。
3. 继续遍历，直到到达键 "romulus" 的叶子节点以获取值。

## 默克尔·帕特里夏树 (Merkle Patricia Trie)

既然我们对默克尔树和帕特里夏树都有了相当的了解，我们就可以深入研究以太坊存储执行层状态的主要数据结构：**默克尔·帕特里夏树 (Merkle Patricia Trie, MPT)**（读作 "try"）。它之所以这样命名，是因为它是一棵使用了 PATRICIA（Practical Algorithm To Retrieve Information Coded in Alphanumeric，检索以字母数字编码的信息的实用算法）特性的默克尔树，并且因为它旨在高效检索组成以太坊状态的项。

- 从默克尔树中，它继承了密码学验证属性 (Cryptographic Verification Properties)，其中每个节点都包含其子节点的哈希值。
- 从帕特里夏树中，它继承了通过基于前缀的节点组织实现的高效键值存储与检索功能。

MPT 内部有三种类型的节点：

- **分支节点 (Branch Nodes)**：分支节点由一个 17 元素的数组组成，其中包括一个节点值和 16 个分支。这种节点类型是分支和遍历树的主要机制。
- **扩展节点 (Extension Nodes)**：这些节点在 MPT 内部作为优化节点运行。当分支节点只有一个子节点时，它们就会发挥作用。MPT 不会为每个分支复制路径，而是将其压缩为一个扩展节点，同时容纳路径和子节点的哈希值。
- **叶子节点 (Leaf Nodes)**：叶子节点代表一个键值对 (Key-Value Pair)。值是 MPT 节点的内容，而键是该节点的哈希值。叶子节点存储特定的键值数据。

每一个节点都有一个哈希值。节点的哈希值计算为其内容的 SHA-3 哈希值。此哈希也作为引用该特定节点的键。
半字节 (Nibbles) 作为 MPT 中键值的区分单位。它代表单个十六进制数 (Hexadecimal Digit)。每个树节点最多可以分支到 16 个分叉，从而确保了简炼的表示和高效的内存使用。

下图说明了遍历和哈希如何在 MPT 中协同工作。作为从叶子节点检索数据的示例，让我们使用键 `11110AA` 获取值 `hi`。

![默克尔·帕特里夏树图 (Merkle Patricia Trie Diagram)](https://epf.wiki/images/data-structures/merkle-patricia-trie.png)

### 1. 从根节点开始（本场景中为扩展节点）
- 我们正在寻找的键是 `11110AA`。
- 根节点是一个**扩展节点 (Extension Node)**，因为此树中的所有键都共享公共前缀 `1111`。
  - 与其在多个分支节点上存储 `1111`，不如将其压缩到单条边中，从而使查找更高效。
- 扩展节点的哈希是根哈希，计算公式为：
  `N1 = hash(RLP(HP(prefix, path=1111), N2))`。
  - 由于 `N1` 取决于 `N2`，因此 `N2` 的任何更改都会改变根哈希。

### **2. 导航到分支节点 (N2)**
- 在 `1111` 之后，我们键 (`11110AA`) 中的下一个十六进制字符是 `0`，因此我们从 `N2` 获取 `0` 分支，这会将我们引导至**叶子节点 (N5)**。
- 分支节点的哈希计算公式为：
  `N2 = hash(RLP(N5, ...))`。
  - 由于 `N2` 取决于 `N5`，因此对 `N5` 的任何修改都会影响 `N2`，进而影响 `N1`。

### **3. 到达叶子节点 (N5)**
- `N5` 是遍历结束的**叶子节点 (Leaf Node)**。
- 叶子节点存储：
  - **前缀 (Prefix)**：`AA`（在 `11110` 之后键的剩余唯一部分）。
  - **值 (Value)**：`"hi"`。

> 这篇 [优秀的帖子](https://easythereentropy.wordpress.com/2014/06/04/understanding-the-ethereum-trie/) 详细解释了 PATRICIA 树，并提供了一个用于练习的 [Python 实现](https://github.com/ebuchman/understanding_ethereum_trie)。

# 以太坊 (Ethereum)

以太坊状态存储在四种不同的修改版默克尔·帕特里夏树 (Modified Merkle Patricia Tries, MMPTs) 中：

- 交易树 (Transaction Trie)
- 收据树 (Receipt Trie)
- 世界状态树 (World State Trie)
- 账户状态树 (Account State Trie)

![各种树 (Tries)](https://epf.wiki/images/tries.png)

在每个区块中，都有一个交易树、收据树和状态树，它们在区块头 (Block Header) 中通过其根哈希 (Root Hashes) 进行引用。
对于部署在以太坊上的每个合约，都有一个存储树 (Storage Trie) 用于保存该合约的持久变量，每个存储树通过其在状态账户对象中的根哈希进行引用，该状态账户对象存储在与该合约地址对应的世界状态树叶子节点中。

## 交易树 (Transaction Trie)

交易树 (Transaction Trie) 是负责存储特定区块内所有交易的数据结构。每个区块都有自己的交易树，对应于包含在该区块中的相应交易。
以太坊是一个基于交易的状态机 (State Machine)。这意味着以太坊中的每个动作或变化都是由于交易引起的。每个区块由区块头和交易列表组成（以及其他内容）。因此，一旦执行了交易并确定了区块，该区块的交易树就永远无法更改（与世界状态树相比）。

![交易树默克尔树 (Transaction Trie Merkle Tree)](https://epf.wiki/images/transaction-trie.png)

交易在树中被映射，使得键是交易索引 (Transaction Index)，值是交易 T。交易索引和交易本身都是 RLP 编码的。它们组成一个存储在树中的键值对：
`𝑅𝐿𝑃 (𝑖𝑛𝑑𝑒𝑥) → 𝑅𝐿𝑃 (𝑇)`

有关交易树中不同交易类型的字段定义，请参阅 [摘要 (Summary)](/wiki/EL/el-data-structures-summary?id=transaction-trie)。

## 收据树 (Receipt Trie)

收据树 (Receipt Trie) 与交易树类似，都是区块级的数据结构，并且树的每个叶子节点代表与交易相关的一些 RLP 编码数据。然而，收据树用于验证每笔交易中的指令是否实际被执行。此验证数据保存在叶子节点中，并包含几个字段，这些字段在维基的 [交易解剖 (Transaction Anatomy)](/wiki/EL/transaction.md#receipts) 部分中进行了描述。

在本节中，我们将重点关注 `收据树` 本身。

`收据树` 的 `ReceiptRoot` 是根节点的 Keccak 256 位哈希。

这是一个简单的收据树图，它遵循默克尔·帕特里夏树流程进行值查找。
![收据树 (Receipt Trie)](https://epf.wiki/images/data-structures/receipt-trie.png)

如果您知道区块中交易的索引，就可以在 `收据树` 中轻松找到其对应的收据。这是因为交易在区块中的位置（索引）被用作 `收据树` 叶子节点中的键，该叶子节点包含该交易的收据。使用交易索引作为收据键提供了一些好处，例如避免了需要计算或查找交易哈希来在树中定位收据。

收据树的主要作用是提供交易结果的规范、经身份验证的记录，主要用于检索历史数据，而无需重新执行交易。在快照同步 (Snap Sync) 期间，全节点 (Full Nodes) 下载区块体 (Block Bodies)——其中包含交易及其对应的收据——并在本地为每个区块重构收据树。然后根据区块头中的 `receiptsRoot` 验证重构的树。快照同步使全节点无需仅仅为了重新生成收据而重新执行历史交易，从而显着加快了同步过程。

虽然收据使轻客户端 (Light Clients) 能够通过针对 `receiptsRoot` 的默克尔证明 (Merkle Proofs) 来验证交易结果，但这只是次要用途。由于轻客户端仅存储区块头，因此它们依赖全节点来查询这些证明和 `receiptsRoot`。这种结构允许轻客户端在不存储完整交易历史的情况下，独立验证数据的合法性。

有关收据的字段定义，请参阅 [摘要 (Summary)](/wiki/EL/el-data-structures-summary?id=receipts-trie)。

## 世界状态树 (World State Trie)

**世界状态树 (World State Trie)** 是代表以太坊当前状态的核心数据结构。它利用**默克尔·帕特里夏树**将 Keccak-256 哈希的 20 字节账户地址映射到其 RLP 编码的状态，其中键值对以字节数组到字节数组的形式存储在树的叶子节点中。

账户可以分为带代码的智能合约账户，或与私钥相关联的外部拥有账户 (Externally Owned Accounts, EOAs)。EOA 用于发起与其它 EOA 或智能合约账户的交易，从而触发相关合约代码的执行。

**世界状态树**并不存储在链中，但在处理完区块中的所有交易后，树的 32 字节 Keccak-256 **状态根 (State Root)** 会存储在每个区块头中。状态根被用作整个系统状态的密码学承诺 (Cryptographic Commitment)，因为它在密码学上依赖于树中的所有数据。例如，给定状态根以及包含账户及其重建状态根所需的兄弟节点的**默克尔证明**，节点就可以证明该账户的存在。此外，每个区块中的状态根锚定了以太坊的共识：任何节点都可以通过将区块的交易应用于先前的状态树，来独立计算或验证此根。

下面是***世界状态树***的简化图示。

![世界状态树图 (World State Trie Diagram)](https://epf.wiki/images/eth-tries.png)

让我们遍历该树以找到余额为 **45 ETH** 的账户。该账户的键显示为 `a711355`，这意味着这七个十六进制数字引导我们从根节点向下到达叶子节点。

> 这个短键 `a711355` 仅用于演示。以太坊中的实际地址会被哈希 (32 字节)，因此在树中通常会产生多达 64 个半字节 (Nibbles)。但遍历步骤是相同的——每个半字节选择下一个分支/扩展节点，直到我们到达存储最终账户数据的叶子节点。

1. **键到半字节 (Key to Nibbles)**
   - 键字符串 `a711355` 代表七个十六进制数字：`a`、`7`、`1`、`1`、`3`、`5`、`5`。
   - 每个数字都是一个半字节（4 位），因此整个键是一个由七个半字节组成的序列。

2. **通过树的路径 (Path Through the Trie)**
   - 根处的**扩展节点 (Extension Node)** 可能存储像 `a7` 这样的前缀，消耗前两个半字节。
   - 紧随其后的是**分支节点 (Branch Node)**，允许通过每个后续半字节（`1`、`1`、`3`、`5`、`5`）进行导航。

3. **叶子节点 (Leaf Node)**
   - 消耗所有半字节会将我们带到**叶子节点 (Leaf Node)**。在我们简化的示例中，其存储的值是**“45 ETH”**。
   - 在以太坊的真实 MPT 中，该叶子节点实际上保存着 RLP 编码的账户对象 `[nonce, balance, storageRoot, codeHash]`。

有关状态树中账户的字段定义，请参阅 [摘要 (Summary)](/wiki/EL/el-data-structures-summary?id=state-trie)。

### 持久化存储 (Persistent Storage)

**世界状态树**是一个随着每个区块而演变的活结构，不像交易树和收据树那样是为每个区块从头开始重建的。以太坊作为一个状态机运行，其中当前状态通过执行区块中的交易来更新。每个节点必须跟踪此当前状态以验证交易并进行相应更新。因此，在区块处理期间存在中间状态，但节点仅保留最终的区块后状态。全节点将跟踪世界状态树的当前状态以及在分叉 (Re-org) 期间回滚所需的足够多的树节点。归档节点 (Archival Nodes) 将跟踪自创世区块 (Genesis) 以来所有以前的状态。

总之，以太坊的世界状态是在给定区块高度处所有账户当前状态的安全且可验证的表示。

## 存储树 (Storage Trie)

在上一节中，我们描述了**世界状态树**中的每个账户叶子节点如何包含一个 `storageRoot`，它是另一个独立的默克尔·帕特里夏树——**存储树 (Storage Trie)** 根节点的 Keccak-256 哈希。该树没有嵌入到世界状态树中，而是通过 `storageRoot` 进行引用，从而能够独立更新和证明存储，同时仍然对全局状态根做出贡献。

**存储树**将合约的持久状态表示为 256 位存储插槽 (Storage Slots) 索引（键）到 256 位 RLP 编码值（值）的映射。每个这样的键值对都被称为一个存储插槽。与世界状态树一样，它使用安全键方案，其中每个插槽索引在插入之前都使用 Keccak-256 进行哈希处理。这可以防止攻击者构建导致长遍历路径或高度不平衡的树结构的键，否则可能会通过在树查找或更新期间引发过度计算来被用于拒绝服务攻击 (Denial of Service Attacks, DOS Attacks)。

> 虽然高级语言（例如 Solidity）定义了合约变量如何在存储插槽中布局，但这种布局抽象源于语言本身。执行层仅实现此抽象。在 EL 层级别，树将所有插槽视为统一的键值条目。

每个账户都有自己的**存储树**，它最初是一个空树。在合约执行期间，该树通过 `SSTORE` 操作码 (Opcode) 进行修改，并通过 `SLOAD` 进行读取。对于 EOA，存储树保持为空且从未被访问。这些操作码在 EVM 中定义，并在 [EVM 文档的存储部分 (EVM documentation's storage section)](wiki/EL/evm.md#evm-data-locations) 中有进一步描述。

要从**存储树**的叶子节点检索存储插槽（例如索引 `0x00`）的值：
1. 对插槽索引进行 RLP 编码，并对结果进行 Keccak-256 哈希计算。
2. 将得到的哈希作为键，从 `storageRoot` 开始遍历该树。
3. 遵循使用哈希的半字节组成的路径到达对应的叶子节点。
4. 提取并解码存储在叶子节点处的 RLP 编码值。

可以从沿着此路径的节点构建证明，以针对 `storageRoot` 验证插槽的值。

总之，**存储树**是固化以太坊账户模型的基础，为每个合约提供了其独立且可验证的存储空间。与将地址映射到账户元数据的世界状态树不同，存储树跨区块维护合约特有的键值状态。

## 未来实现 (Future Implementations)

## 沃克尔树 (Verkle Trees)

[沃克尔树 (Verkle Trees)](https://verkle.info/) 是一种被提议用于替代当前默克尔·帕特里夏树的新数据结构。它由“向量承诺 (Vector Commitment)”和“默克尔树 (Merkle Tree)”结合命名，旨在比当前的 MPT 更高效、更具扩展性。它是一种基于树的数据结构，它将 MPT 中使用的沉重见证数据 (Witness) 替换为轻量级见证数据。沃克尔树是 [以太坊路线图 (Ethereum Roadmap)](https://ethereum.org/en/roadmap/#what-about-the-verge-splurge-etc) 中 The Verge 升级的关键部分。它们可以使无状态客户端 (Stateless Clients) 更高效且更具扩展性。

### 沃克尔树的结构 (Structure of Verkle Tree)

沃克尔树的布局结构就像 MPT，但树的基数（即子节点的数量）不同。就像 [MPT 优化 (MPT Optimization)](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/#optimization) 一样，它有根节点、内部节点、扩展节点和叶子节点。在构建树的键大小方面存在细微差异。MPT 使用 20 字节的键，而沃克尔树使用 32 字节的键，其中 31 字节用作树的茎部 (Stem)，而最后的 1 字节用于具有几乎相同茎部地址或相邻代码块的存储（打开相同的承诺更便宜）。此外，由于在计算见证数据时算法采用 252 位作为域元素，因此使用 31 字节作为树的后缀是很方便的。利用这一点，茎部数据可以承诺范围为 0-127 和 128-255 的两个不同承诺，即同一键的较低值和较高值，从而覆盖整个后缀空间。有关这方面的更多信息，请参阅 [这里](https://blog.ethereum.org/2021/12/02/verkle-tree-structure)。

![沃克尔树 (Verkle Tree)](https://epf.wiki/images/verkle_tree_structure.png)

### MPT 与沃克尔树的关键区别 (Key differences between MPT and Verkle Tree)

默克尔/帕特里夏树具有很大的深度，因为每个节点处的树结构都是二叉/十六叉的。这意味着叶子节点的见证数据是自根到叶子的路径。由于在每一层也需要兄弟哈希数据，这使得大型树的见证数据非常庞大。沃克尔树具有很大的宽度，因为每个节点处的树结构都是 n 叉的。这意味着叶子节点的见证数据是自叶子到根的路径。对于大型树，这可以非常小。目前提议沃克尔树每个节点有 256 个子节点。更多相关内容请参阅 [这里](https://ethereum.org/en/roadmap/verkle-trees/)

默克尔/帕特里夏树的中间节点是子节点的哈希值。沃克尔树的节点携带一种特殊类型的哈希，称为“向量承诺 (Vector Commitments)”，以承诺其子节点。这意味着沃克尔树中叶子节点的见证数据是自叶子到根的路径的子节点承诺。在此基础上，通过聚合承诺来计算证明，这使得验证过程非常紧凑。更多关于 [证明系统 (Proof System)](https://dankradfeist.de/ethereum/2021/06/18/pcs-multiproofs.html?ref=hackernoon.com) 的内容。

### 为什么选择沃克尔树？ (Why Verkle Trees?)

为了使客户端实现无状态 (Stateless)，至关重要的一点是，为了验证区块，客户端不应该存储整个/先前的区块链状态。传入的区块应该能够为客户端提供验证该区块的必要数据。这些额外的证明数据被称为*见证数据 (Witness)*，使无状态客户端能够在没有完整状态的情况下验证数据。

使用区块内部的信息，客户端还应该能够随着每个传入区块维护/增长本地状态。利用这一点，客户端可以保证对于当前区块（以及它验证的后续区块），状态过渡是正确的。它不能保证对于当前区块所引用的先前区块状态是正确的，因为区块生产者可以构建在无效或非规范的区块之上。

沃克尔树旨在提高存储和通信成本的效率。对于 1000 个叶子/数据，二叉默克尔树大约需要 4MB 的见证数据，而沃克尔树将其减少到 150 kB。如果我们把见证数据包含在区块中，它不会对区块大小产生太大影响，但它能使无状态客户端更高效且更具扩展性。使用这一点，无状态客户端将能够信任所完成的计算，而无需存储整个状态。

过渡到新的沃克尔树数据库带来了重大挑战。为了安全地创建新的沃克尔数据，客户端需要从现有的 MPT 中生成它们，这需要大量的计算和空间。目前正在研究沃克尔数据库的分配和验证。

## 资源 (Resources)

- [以太坊中的默克尔 (Merkle in Ethereum)](https://blog.ethereum.org/2015/11/15/merkling-in-ethereum)
- [关于默克尔·帕特里夏树的更多信息 (More on Merkle Patricia Trie)](https://ethereum.org/developers/docs/data-structures-and-encoding/patricia-merkle-trie)
- [关于沃克尔树的更多信息 (More on Verkle Tree)](https://notes.ethereum.org/@vbuterin/verkle_tree_eip#Simple-Summary)
- [Verge 过渡 (Verge transition)](https://notes.ethereum.org/@parithosh/verkle-transition)
- [实现默克尔树与帕特里夏树 (Implementing Merkle Tree and Patricia Trie)](https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591) • [已归档](https://web.archive.org/web/20210118071101/https://medium.com/coinmonks/implementing-merkle-tree-and-patricia-trie-b8badd6d9591)
- [基数树 (Radix Trie)](https://en.wikipedia.org/wiki/Radix_tree#) • [已归档](https://web.archive.org/web/20250105072609/https://en.wikipedia.org/wiki/Radix_tree)
- [基数树图示 (Radix Trie Diagram)](https://samczsun.com/content/images/2021/05/1920px-Patricia_trie.svg-1-.png) • [已归档](https://web.archive.org/web/20231209235318/https://samczsun.com/content/images/2021/05/1920px-Patricia_trie.svg-1-.png)
- [默克尔·帕特里夏树图示 (Merkle Patricia Trie Diagram)](https://www.researchgate.net/publication/353863430/figure/fig2/AS:1056193841741826@1628827643578/Ethereum-Encoded-Merkle-Patricia-Trie.png)
- [默克尔·帕特里夏树图示说明 (Merkle Patricia Trie Diagram Explanation)](https://www.researchgate.net/publication/353863430_Ethereum_Data_Structures)
- [包含图示的收据树 (Receipts Trie Including Diagram)](https://medium.com/coinmonks/ethereum-data-transaction-receipt-trie-and-logs-simplified-30e3ae8dc3cf) • [已归档](https://web.archive.org/web/20250000000000/https://medium.com/coinmonks/ethereum-data-transaction-receipt-trie-and-logs-simplified-30e3ae8dc3cf)
- [以太坊数据结构 (Ethereum Data Structures)](https://arxiv.org/pdf/2108.05513/1000) • [已归档](https://web.archive.org/web/20240430050355/https://arxiv.org/pdf/2108.05513/1000)
- [DevP2P 有线协议 (DevP2P Wire Protocol)](https://github.com/ethereum/devp2p/blob/master/caps/eth.md) • [已归档](https://web.archive.org/web/20250328095848/https://github.com/ethereum/devp2p/blob/master/caps/eth.md)
- [快照同步 (Snap Sync)](https://geth.ethereum.org/docs/fundamentals/sync-modes) • [已归档](https://web.archive.org/web/20250228111146/https://geth.ethereum.org/docs/fundamentals/sync-modes)
- [关于默克尔·帕特里夏树的更多信息 (More on Merkle Patricia Trie)](https://ethereum.org/developers/docs/data-structures-and-encoding/patricia-merkle-trie)
- [以太坊黄皮书 (Ethereum Yellow Paper)](https://ethereum.github.io/yellowpaper/paper.pdf) • [已归档](https://web.archive.org/web/20250228142704/https://ethereum.github.io/yellowpaper/paper.pdf)
- [状态树键 (State Trie Keys)](https://medium.com/codechain/secure-tree-why-state-tries-key-is-256-bits-1276beb68485#:~:text=This%20is%20because%20when%20Ethereum,the%20secure%20tree%20in%20Ethereum) • [已归档](https://web.archive.org/web/20230524084537/https://medium.com/codechain/secure-tree-why-state-tries-key-is-256-bits-1276beb68485)
