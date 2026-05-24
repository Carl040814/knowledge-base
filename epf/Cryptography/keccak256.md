# Keccak256

Keccak256 是一种在以太坊区块链中广泛使用的密码学哈希函数（cryptographic hash function）。

## 简史

Keccak256 最初是提交给 [NIST 密码学哈希算法竞赛](https://keccak.team/files/Keccak-submission-3.pdf)的参赛作品。该竞赛旨在确定一个新的安全哈希算法来替代 SHA-1。KECCAK 算法由 Guido Bertoni、Joan Daemen、Michaël Peeters 和 Gilles Van Assche 组成的团队设计。

## Keccak 设计

Keccak 以其海绵结构（sponge construction）而著称，这一独特特性使其能够"吸收"任意长度的输入数据，然后"挤出"所需长度的哈希值。

海绵函数（sponge function）是 Keccak 设计的核心，在两个不同阶段运行：吸收和挤压。

### 吸收阶段（Absorption Phase）

- **输入处理**：在此阶段，输入数据被分成块并与海绵状态（sponge's state）进行异或（XOR）操作，该状态称为比特率（bitrate）。
- **比特率（`r`）**：比特率是定义状态中与输入直接交互的位数的参数。它决定了数据吸收过程的效率和吞吐量。
- **状态置换（State Permutation）**：每次 XOR 操作后，对整个状态应用置换函数，确保输入和状态的充分混合。

### 挤压阶段（Squeezing Phase）

- **输出生成**：一旦吸收完成，挤压阶段开始。此时，从状态的比特率部分生成输出哈希。
- **任意输出长度**：挤压阶段可以生成任意所需长度的输出。

要更深入地了解 Keccak 的内部工作原理，[Keccak 参考文档](https://keccak.team/files/CSF-0.1.pdf)提供了关于其算法和安全特性的详细见解。

## EVM 实现

EVM（以太坊虚拟机）以基于堆栈的架构处理以太坊区块链交易的执行。EVM 操作码（opcodes）是预定义的指令，EVM 解释并随后执行以完成交易并运行智能合约。操作码包括算术、环境、控制流和堆栈操作等类型。目前没有 keccak256 操作码，但有一个 SHA3 操作码。SHA3 操作码用于加密来自堆栈的输入数据并输出一个 Keccak256 哈希。

[以太坊黄皮书](https://ethereum.github.io/yellowpaper/paper.pdf)概述了 Keccak256 在以太坊区块链中的其他实现：

### 在区块创建和根数据结构中的使用

- **区块头字段**：区块头中的各种字段，如 `parentHash` 和 `stateRoot`，使用 Keccak 256 位哈希。这包括对父区块的整个头部、状态 trie 的根节点以及交易和收据 trie 结构的根节点进行哈希。
- **Merkle Patricia 树**：以太坊采用 Merkle Patricia 树来编码其状态，其中树中每个节点通过其内容的 Keccak 256 位哈希来标识。此结构支撑着区块头中的 stateRoot 字段。
- **存储内容编码**：该哈希用于编码账户的存储内容，将整数键的 Keccak 256 位哈希映射到 RLP 编码的整数值。

在所有这些实例中，Keccak256 的角色对确保数据完整性、促进高效的数据检索以及支持区块链的底层安全机制至关重要。

## Keccak256 vs SHA3-256

[引用以太坊的 Nick Johnson](https://github.com/ethereum/go-ethereum/pull/2940#issuecomment-274809794)：
> SHA3-256 是 Keccak256，只是数据填充方式有所改变。使用 Keccak256 是因为以太坊协议是在明确 Keccak256 赢得 SHA3 竞赛之后，但在填充方式改变之前定义的。

## 参考文献

- [NIST SHA-3 Competition](https://keccak.team/files/Keccak-submission-3.pdf)
- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- [EVM Opcodes](https://www.evm.codes/?fork=shanghai)
