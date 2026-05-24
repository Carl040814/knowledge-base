# 以太坊虚拟机 (EVM)

以太坊虚拟机 (Ethereum Virtual Machine, EVM) 是以太坊世界计算机的核心。它执行最终确认交易所需的关键计算，并将结果永久存储在区块链上。本文探讨 EVM 在以太坊生态系统中的角色及其运作方式。

## 以太坊状态机

当 EVM 处理交易时，它会改变以太坊的整体状态 (state)。从这个角度来看，以太坊可以被视为一个**状态机 (state machine)**。

在计算机科学中，**状态机**是一种用于对系统行为进行建模的抽象。它展示了一个系统如何通过一组不同的状态来表示，以及输入如何驱动状态的变化。

一个熟悉的例子是自动售货机，一种在收到付款后分发产品的自动化系统。

我们可以将自动售货机建模为存在于三种不同的状态：空闲 (idle)、等待用户选择 (awaiting selection) 和分发产品 (dispensing a product)。诸如投币或产品选择等输入会触发这些状态之间的转换，如下图所示：

![自动售货机](https://epf.wiki/images/evm/vending-machine.gif)

让我们正式定义状态机的组成部分：

1. **状态 ($S$)**：状态表示系统在给定时间点可能处于的不同条件或配置。
   对于自动售货机，可能的状态有：

$$ S\in \set {Idle, Selection, Dispensing} $$

2. **输入 ($I$)**：输入是系统中的动作、信号或环境变化。输入触发**状态转换函数 (state transition function)**。
   对于自动售货机，可能的输入包括：

$$ I\in \set {InsertCoin, SelectDrink, CollectDrink} $$

3. **状态转换函数 ($\Upsilon$)**：状态转换函数定义了系统如何基于输入和当前状态，从一个状态转换到另一个状态（或回到相同状态）。本质上，它决定了系统如何响应输入而演化。

$$\Upsilon (S,I) \Longrightarrow S'  $$

> 其中 S' = 下一状态，S = 当前状态，I = 输入。

自动售货机的状态转换示例：

$$\Upsilon (Idle,InsertCoin) \Longrightarrow Selection $$
$$\Upsilon (Selection,SelectDrink) \Longrightarrow Dispensing $$
$$\Upsilon (Idle,SelectDrink) \Longrightarrow Idle $$

注意在最后一种情况下，当前状态转换回了自身。

### 以太坊作为状态机

以太坊作为一个整体，可以被视为一个**基于交易的状态机 (transaction-based state machine)**。它接收交易作为输入，并转换到新的状态。以太坊的当前状态被称为**世界状态 (world state)**。

考虑一个简单的以太坊应用——一个 [NFT](https://ethereum.org/en/nft/) 市场。

在当前世界状态 **S3**（绿色）中，Alice 拥有一个 NFT。下面的动画展示了一笔将所有权转移给你的交易（**S3** ➡️ **S4**）。类似地，将 NFT 卖回给 Alice 会将状态转换到 **S5**：

![以太坊状态机](https://epf.wiki/images/evm/ethereum-state-machine.gif)

注意当前世界状态以*脉动绿色气泡*的形式动画展示。

在上图中，每笔交易都提交到一个新状态。然而，实际上，一组交易被打包进一个**区块 (block)**，结果状态被添加到先前状态的链上。现在应该清楚为什么这项技术被称为**区块链 (blockchain)**了。

考虑到状态转换函数的定义，我们得出以下结论：

> ℹ️ 注意
> **EVM 是以太坊状态机的状态转换函数。它基于输入（交易）和当前状态，决定以太坊如何转换到新的（世界）状态。**

在以太坊中，世界状态本质上是 20 字节地址到账户状态的映射。

![以太坊世界状态](https://epf.wiki/images/evm/ethereum-world-state.jpg)

每个账户状态由各种组件组成，如存储 (storage)、代码 (code)、余额 (balance) 以及其他数据，并与特定地址关联。

以太坊有两种类型的账户：

- **外部账户 (External Account)**：由关联的私钥控制的账户，EVM 代码为空。
- **合约账户 (Contract Account)**：由关联的非空 EVM 代码控制的账户。此类账户中的 EVM 代码通常被称为*智能合约 (smart contract)*。

关于世界状态如何实现的详细信息，请参阅[以太坊数据结构](wiki/EL/data-structures.md)。

## 虚拟机范式

理解了状态机之后，下一个挑战是**实现**。

软件需要转换为目标处理器的机器语言（指令集架构，Instruction Set Architecture, ISA）才能执行。这种 ISA 因硬件而异（例如 Intel vs. Apple silicon）。现代软件还依赖主机操作系统进行内存管理和其他基本功能。

在由多样化硬件和操作系统组成的碎片化生态系统中确保功能正常运行是一个重大障碍。传统上，软件必须为每个特定目标平台编译为原生二进制：

![平台依赖执行](https://epf.wiki/images/evm/platform-dependent-execution.jpg)

为了解决这一挑战，采用了由两部分组成的解决方案。

首先，实现面向**虚拟机 (virtual machine)**，即一个抽象层。源代码被编译为**字节码 (bytecode)**，即表示指令的字节序列。每个字节码映射到虚拟机执行的特定操作。

第二部分涉及一个平台特定的虚拟机，它将字节码转换为原生代码以执行。

这提供了两个关键优势：可移植性 (portability)（字节码无需重新编译即可在不同平台上运行）和抽象 (abstraction)（将硬件复杂性与软件分离）。开发人员因此可以为单一的虚拟机编写代码：

![虚拟机范式](https://epf.wiki/images/evm/virtual-machine-paradigm.jpg)

Java 的 [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine) 和 Lua 的 LuaVM 是虚拟机的流行示例。它们创建平台中立的字节码，使代码无需重新编译即可在各种系统上运行。

## EVM

虚拟机概念作为抽象层。以太坊虚拟机 (EVM) 是这种抽象的*特定*软件实现。EVM 的解剖结构如下所述：

在计算机体系结构中，字 (word) 指的是 CPU 一次可以处理的固定大小的数据单元。EVM 的字大小为 **32 字节**。

![EVM 解剖结构](https://epf.wiki/images/evm/evm-anatomy.jpg)

*为清晰起见，上图简化了以太坊状态。实际状态包括额外的元素，如消息帧 (Message Frames) 和瞬态存储 (Transient Storage)。*

在上面描述的解剖结构中，EVM 展示为操作账户实例的存储、代码和余额。

在实际场景中，EVM 可能执行涉及多个账户的交易（每个账户具有独立的存储、代码和余额），从而在以太坊上实现复杂的交互。

对虚拟机有了更好的理解后，让我们扩展我们的定义：

> ℹ️ 注意
> EVM 是以太坊状态机的状态转换函数。它基于输入（交易）和当前状态，决定以太坊如何转换到新的（世界）状态。**它被实现为虚拟机，因此可以在任何平台上运行，独立于底层硬件。**

## EVM 字节码

EVM 字节码是将程序表示为[**字节 (bytes)**](https://en.wikipedia.org/wiki/Byte)（8 位）序列的形式。字节码中的每个字节要么是：

- 称为**操作码 (opcode)** 的指令，或
- 操作码的输入，称为**操作数 (operand)**。

![二进制字节码](https://epf.wiki/images/evm/opcode-binary.jpg)

为简洁起见，EVM 字节码通常以[**十六进制 (hexadecimal)**](https://en.wikipedia.org/wiki/Hexadecimal)表示法表达：

![十六进制字节码](https://epf.wiki/images/evm/opcode-hex.jpg)

为了进一步提高可理解性，操作码具有人类可读的助记符。这种简化的字节码称为 **EVM 汇编 (EVM assembly)**，是 EVM 代码的最低人类可读形式：

![EVM 汇编](https://epf.wiki/images/evm/opcode-assembly.jpg)

从操作数中识别操作码很简单。目前，只有 `PUSH*` 操作码有操作数（这可能会随着 [EOF](https://eips.ethereum.org/EIPS/eip-7569) 而改变）。`PUSHX` 定义了操作数长度（PUSH 之后的 X 字节）。

本讨论中使用的一些操作码：

| 操作码 | 名称           | 描述                                        |
| ------ | -------------- | -------------------------------------------------- |
| 60     | `PUSH1`        | 将 1 字节压入栈中                           |
| 01     | `ADD`          | 将栈顶 2 个值相加                  |
| 02     | `MUL`          | 将栈顶 2 个值相乘             |
| 39     | `CODECOPY`     | 将当前环境中运行的代码复制到内存 |
| 51     | `MLOAD`        | 从内存加载字                              |
| 52     | `MSTORE`       | 将字存储到内存                               |
| 53     | `MSTORE8`      | 将字节存储到内存                               |
| 59     | `MSIZE`        | 获取扩展内存的字节大小           |
| 54     | `SLOAD`        | 从存储加载字                             |
| 55     | `SSTORE`       | 将字存储到存储                              |
| 56     | `JUMP`         | 修改程序计数器                          |
| 5B     | `JUMPDEST`     | 标记跳转目的地                         |
| f3     | `RETURN`       | 停止执行并返回输出数据               |
| 35     | `CALLDATALOAD` | 从 calldata 复制 32 字节到栈               |
| 37     | `CALLDATACOPY` | 从 calldata 复制输入数据到内存            |
| 80–8F  | `DUP1–DUP16`   | 复制第 N 个栈项到栈顶                    |
| 90–9F  | `SWAP1–SWAP16` | 交换栈顶与第 N+1 个栈项                     |

完整列表请参阅[黄皮书附录 H](https://ethereum.github.io/yellowpaper/paper.pdf)。

> ℹ️ 注意
> [EIP](https://eips.ethereum.org/) 可以提出 EVM 修改。例如，[EIP-1153](https://eips.ethereum.org/EIPS/eip-1153) 引入了 `TLOAD` 和 `TSTORE` 操作码。

以太坊客户端如 [geth](https://github.com/ethereum/go-ethereum) 实现了 [EVM 规范](https://github.com/ethereum/execution-specs)。这确保所有节点就交易如何改变系统状态达成一致，从而在整个网络中创建统一的执行环境。

我们已经介绍了 EVM 是**什么**，接下来让我们探索它是**如何**工作的。

# EVM 数据位置

EVM 在执行期间有四个主要的数据存储位置：

- **栈 (Stack)**
- **内存 (Memory)**
- **存储 (Storage)**
- **Calldata**

让我们更深入地探索这些数据存储。

## 栈

栈是一种具有两种操作的简单数据结构：**PUSH** 和 **POP**。Push 将项添加到栈顶，而 pop 移除最顶部的项。栈以后进先出（Last-In-First-Out, LIFO）原则运行——最后添加的元素最先被移除。如果你尝试从空栈中弹出，将发生**栈下溢错误 (stack underflow error)**。

由于栈是大多数操作码操作的地方，它负责保存用于读写**内存**和**存储**的值，我们稍后会详细说明。

EVM 使用栈的主要用途是存储计算中的中间值，以及为操作码提供参数。

![EVM 栈](https://epf.wiki/images/evm/stack.gif)

> EVM 栈的最大大小为 1024 项，每项 32 字节，并在每次合约执行后重置。只有栈顶的 16 项可被访问。如果栈空间耗尽，合约执行将失败。

这 16 项栈大小的限制是由 **DUP** 和 **SWAP** 操作码造成的。

- **`DUPn`**：复制第 n 个栈项到栈顶。
- **`SWAPn`**：交换栈顶与第 n+1 个栈项。

**DUP** 和 **SWAP** 的最大 n 是 `16`，分别对应操作码 0x80–0x8f 和 0x90–0x9f。这些操作码在 EVM 中被明确定义并构成一个固定集合——使 16 项限制成为 EVM 级别的约束。

在字节码执行期间，EVM 栈作为一个*草稿纸*：操作码从栈顶消费数据并将结果压回（参见下面的 `ADD` 操作码）。考虑一个简单的加法程序：

![EVM 栈加法](https://epf.wiki/images/evm/stack-addition.gif)

提醒：所有值都是十六进制，因此 `0x06 + 0x07 = 0x0d`（十进制：13）。

让我们花点时间来庆祝我们的第一个 EVM 汇编代码 🎉。

### 程序计数器

回想一下，字节码是一个扁平的字节数组，每个操作码为 1 字节。EVM 需要一种方法来跟踪字节码数组中下一个要执行的字节（操作码）。这就是 EVM **程序计数器 (program counter)** 的作用。它将跟踪下一个操作码的偏移量 (offset)，即字节数组中下一个要在栈上执行的指令的位置。

在上面的示例中，汇编代码左侧的值表示每个操作码在字节码中的字节偏移量（从 0 开始）：

| 字节码 | 汇编 | 指令长度（字节） | 偏移量（十六进制） |
| -------- | -------- | ------------------------------ | ------------- |
| 60 06    | PUSH1 06 | 2                              | 00            |
| 60 07    | PUSH1 07 | 2                              | 02            |
| 01       | ADD      | 1                              | 04            |

注意上表不包含偏移量 01。这是因为操作数 06 占据了偏移量 01 的位置，同样的概念适用于操作数 07 占据偏移量 03 的位置。

本质上，**程序计数器**确保 EVM 知道每个下一条指令的位置以及何时停止执行，如下例所示。

![EVM 程序计数器](https://epf.wiki/images/evm/program-counter.gif)

`JUMP` 操作码直接设置程序计数器，实现动态控制流 (control flow)，并通过允许灵活的程序执行路径为 EVM 的[图灵完备性 (Turing completeness)](https://en.wikipedia.org/wiki/Turing_completeness) 做出贡献。

![EVM JUMP 操作码](https://epf.wiki/images/evm/jump-opcode.gif)

该代码在无限循环中运行，反复加 7。它引入了两个新的操作码：

- **JUMP**：将程序计数器设置为栈顶值（在我们的例子中是 02），决定下一条要执行的指令。
- **JUMPDEST**：标记跳转操作的目的地，确保预期的目的地并防止不想要的控制流中断。

> 高级语言如 [Solidity](https://soliditylang.org/) 利用跳转来实现诸如 if 条件、for 循环和内部函数调用等构造。

### Gas

我们那个看似无害的小程序可能看起来很无害。然而，EVM 中的无限循环构成了重大威胁：它们可能**吞噬资源**，潜在地导致网络[**DoS 攻击 (Denial-of-Service attack)**](https://en.wikipedia.org/wiki/Denial-of-service_attack)。

EVM 的 **gas** 机制通过充当计算资源的货币来应对此类威胁。Gas 成本被[设计为反映](https://web.archive.org/web/20170904200443/https://docs.google.com/spreadsheets/d/15wghZr-Z6sRSMdmRmhls9dVXTOpxKy8Y64oy9MvDZEQ/edit#gid=0)硬件的限制，如存储容量或处理能力。交易以 **Ether (ETH)** 支付 gas 来使用 EVM，如果它们在完成之前耗尽 gas（如无限循环），EVM 将停止它们以防止资源霸占。

这保护网络不被资源密集型或恶意活动堵塞。由于 gas 将计算限制在有限步数内，EVM 被认为是**准图灵完备 (quasi Turing complete)** 的。

![EVM Gas](https://epf.wiki/images/evm/evm-gas.gif)

为了简单起见，我们的例子假设每个操作码消耗 1 单位 gas，但实际成本因复杂性而异。核心概念保持不变。

具体 gas 成本请参阅[黄皮书附录 G](https://ethereum.github.io/yellowpaper/paper.pdf)。

## 内存

EVM 内存是一个 $2^{256}$（或[实际上无限](https://www.talkcrypto.org/blog/2019/04/08/all-you-need-to-know-about-2256/)）字节的字节数组。内存中的所有位置最初都明确定义为零。

![EVM 内存](https://epf.wiki/images/evm/evm-memory.gif)

与为单个指令提供数据的栈不同，内存存储的是与整个程序相关的短暂数据。由于栈有一个字的硬限制，**内存**通过允许对任意大小的数据进行索引访问来补充栈。栈值可以按需存储到**内存**或从**内存**加载。

### 写入内存

`MSTORE` 从栈中取两个值：一个地址**偏移量**和一个 32 字节的**值**。然后将该值写入指定偏移量处的内存。

![MSTORE](https://epf.wiki/images/evm/mstore.gif)

`MSIZE` 在栈上报告当前使用的内存大小（以字节为单位，本例中为 32 字节或十六进制的 20）。

`MSTORE8` 接受与 `MSTORE` 相同的参数，但写入 1 字节而不是 1 个字。

![MSTORE8](https://epf.wiki/images/evm/mstore8.gif)

注意：当将 07 写入内存时，现有值 (06) 保持不变。它被写入相邻的字节偏移量。

活动内存的大小仍然是 1 个字。

### 内存扩展

在 EVM 中，内存以 1 个字"页"的倍数动态分配。Gas 按扩展的页数收费。

![内存扩展](https://epf.wiki/images/evm/memory-expansion.gif)

在 1 字节偏移量处写入一个字会溢出初始内存页，触发扩展到 2 个字（64 字节或 0x40）。

### 读取内存

`MLOAD` 从内存中读取一个值并将其压入栈中。

![MLOAD](https://epf.wiki/images/evm/mload.gif)

EVM 没有直接等效于 `MSTORE8` 的读取操作。你必须使用 `MLOAD` 读取整个字，然后使用[掩码 (mask)](<https://en.wikipedia.org/wiki/Mask_(computing)>) 提取所需的字节。

> EVM 内存以 32 字节块显示，以说明内存扩展的工作原理。实际上，它是一个无缝的字节序列，没有任何固有的分割或块。

## Calldata

**Calldata** 是通过消息调用 (message call) 指令或从交易传递给 EVM 的只读输入数据，存储为可通过特定操作码访问的字节序列。

### 从 Calldata 读取

当前环境的 calldata 可以通过以下方式访问：

- `CALLDATALOAD` 操作码，从所需偏移量读取 32 字节到栈上，[了解更多](https://veridelisi.medium.com/learn-evm-opcodes-v-a59dc7cbf9c9)。
- 或使用 `CALLDATACOPY` 将 calldata 的一部分复制到内存。

## 存储

存储被设计为**按字寻址的字数组**。与内存不同，存储与以太坊账户关联，并作为世界状态的一部分在交易之间**持久化 (persisted)**。它可以被看作是与智能合约关联的键值**数据库**，这就是为什么它包含合约的"状态"变量。存储大小固定为 2^256 个槽位 (slot)，每个 32 字节。

![EVM 存储](https://epf.wiki/images/evm/evm-storage.jpg)

存储只能通过其关联账户的代码访问。外部账户没有代码，因此无法访问自己的存储。

## 写入存储

`SSTORE` 从栈中取两个值：一个存储**槽位**和一个 32 字节的**值**。然后将该值写入账户的存储中。注意：槽位可以被认为是存储的基本单元，因此当写入存储时，我们处理的是槽位而不是单个字节。写入存储是昂贵的。高级语言如 Solidity 通过在多个变量的组合大小小于或等于 32 字节时将它们打包到单个 32 字节槽位中来优化存储。

![EVM 存储写入](https://epf.wiki/images/evm/sstore.gif)

我们一直在运行合约账户的字节码。直到现在，我们才看到账户和世界状态，它与 EVM 内部的代码匹配。

再次强调，重要的是要注意存储不是 EVM 本身的一部分，而是当前正在执行的合约账户的。更具体地说，合约账户包含一个**存储根 (storage root)**，指向该合约存储的一个单独的 Merkle Patricia Trie。

上面的例子只显示了账户存储的一小部分。像内存一样，存储中的所有值都明确定义为零。此外，当合约执行 `SSTORE` 操作码将存储槽位的值从非零设置回零时，该操作有资格获得 gas 退款 (gas refund)（gas 不会立即返回，而是计入交易的退款计数器）。此退款作为对账户释放状态树 (state trie) 中宝贵存储空间的奖励，并在交易结束时应用，以抵消部分 gas 成本。


### 读取存储

`SLOAD` 从栈中取存储**槽位**，并将其值加载回栈上。

![EVM 存储读取](https://epf.wiki/images/evm/sload.gif)

注意存储值在示例之间保持持久，展示了它在世界状态中的持久性。由于世界状态在所有节点之间复制，存储操作在 gas 上很昂贵。

> ℹ️ 注意
> 查看关于[交易](/wiki/EL/transaction.md)的 wiki 页面，了解 EVM 的实际应用。

## 总结

我们探索了操作码 (opcode) 如何成为 EVM 执行的核心指令，在栈上操作以执行计算。计算结果可以短暂存储在内存中，或持久化在合约存储中。

开发者很少直接编写 EVM 汇编代码，除非性能优化至关重要。相反，大多数开发者使用高级语言如 [Solidity](https://soliditylang.org/)，然后将其编译为字节码。

以太坊是一个不断演进的协议，虽然我们讨论的基础知识在很大程度上仍然相关，但建议关注[以太坊改进提案 (EIP)](https://eips.ethereum.org/) 和[网络升级](https://ethereum.org/history)，以了解以太坊生态系统的最新发展。

## EVM 升级

虽然以太坊协议在每次升级中经历许多变化，但 EVM 的变化相当微妙。EVM 的重大变化可能会破坏合约和语言，需要维护多个 EVM 版本，这会引入大量复杂性开销。EVM 本身仍会进行某些升级，如新增操作码或修改现有操作码而不破坏其逻辑。一些例子如 [1153](https://eips.ethereum.org/EIPS/eip-1153)、[4788](https://eips.ethereum.org/EIPS/eip-4788)、[5000](https://eips.ethereum.org/EIPS/eip-5000)、[5656](https://eips.ethereum.org/EIPS/eip-5656) 和 [6780](https://eips.ethereum.org/EIPS/eip-6780)。这些 EIP 提议添加新操作码，除了最后一个特别有趣，因为它在不破坏兼容性的情况下中和了 `SELFDESTRUCT` 操作码。EVM 的另一个重要升级将标志着一个更重大的变化，即 [EOF](https://notes.ethereum.org/@ipsilon/mega-eof-specification)。它为字节码创建了一种格式，EVM 可以更容易地理解和处理，它包含各种 EIP，并且已经被讨论和打磨了相当长的时间。

## 资源

### 状态机和计算理论

- 📝 Mark Shead，["Understanding State Machines."](https://medium.com/free-code-camp/state-machines-basics-of-computer-science-d42855debc66) • [archived](https://web.archive.org/web/20210309014946/https://medium.com/free-code-camp/state-machines-basics-of-computer-science-d42855debc66)
- 🎥 Prof. Harry Porter，["Theory of computation."](https://www.youtube.com/playlist?list=PLbtzT1TYeoMjNOGEiaRmm_vMIwUAidnQz)
- 📘 Michael Sipser，["Introduction to the Theory of Computation."](https://books.google.com/books/about/Introduction_to_the_Theory_of_Computatio.html?id=4J1ZMAEACAAJ)
- 🎥 Shimon Schocken et al.，["Build a Modern Computer from First Principles: From Nand to Tetris."](https://www.coursera.org/learn/build-a-computer)

### EVM

以下资源根据不同的 EVM 学习阶段进行了分类。

#### EVM 基础

- 🎥 Whiteboard Crypto，["EVM: An animated non-technical introduction."](https://youtu.be/sTOcqS4msoU)
- 📝 Vasa，[Getting Deep Into EVM: How Ethereum Works Backstage](https://medium.com/swlh/getting-deep-into-evm-how-ethereum-works-backstage-ab6ad9c0d0bf)
- 📝 Zaryab Afser，[The ABCs of Ethereum Virtual Machine](https://www.decipherclub.com/the-abcs-of-ethereum-virtual-machine/)
- 📝 Preethi，[EVM Tweet Thread](https://twitter.com/iam_preethi/status/1483459717670309895)
- 📝 Decipher Club，[EVM learning resources based on your level of expertise](https://www.decipherclub.com/evm-learning-resources/)

#### 理解 EVM 架构和核心组件

- 📝 Gavin Wood，["Ethereum Yellow Paper."](https://ethereum.github.io/yellowpaper/paper.pdf)
- 📝 Ethereum Book，[Chapter 13, Ethereum Book](https://cypherpunks-core.github.io/ethereumbook/13evm.html?ref=decipherclub.com)
- 📘 Andreas M. Antonopoulos & Gavin Wood，["Mastering Ethereum."](https://github.com/ethereumbook/ethereumbook)
- 🎥 Jordan McKinney，["Ethereum Explained: The EVM."](https://www.youtube.com/watch?v=kCswGz9naZg)
- 📝 LeftAsExercise，["Smart contracts and the Ethereum virtual machine."](https://leftasexercise.com/2021/08/08/q-smart-contracts-and-the-ethereum-virtual-machine/) • [archived](https://web.archive.org/web/20230324200211/https://leftasexercise.com/2021/08/08/q-smart-contracts-and-the-ethereum-virtual-machine/)
- 📝 Femboy Capital，["A Playdate with the EVM."](https://femboy.capital/evm-pt1) • [archived](https://web.archive.org/web/20221001113802/https://femboy.capital/evm-pt1)
- 🎥 Alex，[EVM - Some Assembly Required](https://www.youtube.com/watch?v=yxgU80jdwL0)

#### EVM 深入探讨

- 📝 Takenobu Tani，[EVM illustrated](https://github.com/takenobu-hs/ethereum-evm-illustrated)
- 📝 Shafu，["EVM from scratch."](https://evm-from-scratch.xyz/)
- 📝 NOXX，["3 part series: EVM Deep Dives - The Path to Shadowy Super Coder."](https://noxx.substack.com/p/evm-deep-dives-the-path-to-shadowy) • [archived](https://web.archive.org/web/20240106034644/https://noxx.substack.com/p/evm-deep-dives-the-path-to-shadowy)
- 📝 OpenZeppelin，["6 part series: Deconstructing a Solidity."](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-i-introduction-832efd2d7737) • [archived](https://web.archive.org/web/20240121025651/https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-i-introduction-832efd2d7737)
- 📝 TrustLook，["Understand EVM bytecode."](https://blog.trustlook.com/understand-evm-bytecode-part-1/) • [archived](https://web.archive.org/web/20230603080857/https://blog.trustlook.com/understand-evm-bytecode-part-1/)
- 📝 Degatchi，["A Low-Level Guide To Solidity's Storage Management."](https://degatchi.com/articles/low_level_guide_to_soliditys_storage_management) • [archived](https://web.archive.org/web/20231202105650/https://degatchi.com/articles/low_level_guide_to_soliditys_storage_management/)
- 📝 Zaryab Afser，["Journey of smart contracts from Solidity to Bytecode"](https://www.decipherclub.com/ethereum-virtual-machine-article-series/)
- 🎥 Ethereum Engineering Group，[EVM: From Solidity to byte code, memory and storage](https://www.youtube.com/watch?v=RxL_1AfV7N4&t=2s)
- 📝 Trust Chain，[7 part series about how Solidity uses EVM under the hood.](https://trustchain.medium.com/reversing-and-debugging-evm-smart-contracts-392fdadef32d)
- [Learn EVM Opcodes](https://veridelisi.medium.com/learn-evm-opcodes-v-a59dc7cbf9c9) • [archived](https://web.archive.org/web/20240806231824/https://veridelisi.medium.com/learn-evm-opcodes-v-a59dc7cbf9c9)
- [More on EVM Storage](https://medium.com/coinmonks/solidity-storage-how-does-it-work-8354afde3eb) • [archived](https://web.archive.org/web/20230808231549/https://medium.com/coinmonks/solidity-storage-how-does-it-work-8354afde3eb)
- [Storage, Memory, and Stack Overview](https://ethereum.stackexchange.com/questions/23720/usage-of-memory-storage-and-stack-areas-in-evm) • [archived](https://web.archive.org/web/20240529150647/https://ethereum.stackexchange.com/questions/23720/usage-of-memory-storage-and-stack-areas-in-evm)
- [Calldata](https://learnevm.com/chapters/fn/calldata) • [archived](https://web.archive.org/web/20250306133755/https://learnevm.com/chapters/fn/calldata)

### 工具和 EVM 谜题

- 🧮 smlXL，["evm.codes: Opcode reference and interactive playground."](https://www.evm.codes/)
- 🧮 smlXL，["evm.storage: Interactive storage explorer."](https://www.evm.storage/)
- 🧮 Ethervm，[Low level reference for EVM opcodes](https://ethervm.io/)
- 🎥 Austin Griffith，["ETH.BUILD."](https://www.youtube.com/watch?v=30pa790tIIA&list=PLJz1HruEnenCXH7KW7wBCEBnBLOVkiqIi)
- 💻 Franco Victorio，["EVM puzzles."](https://github.com/fvictorio/evm-puzzles)
- 💻 Dalton Sweeney，["More EVM puzzles."](https://github.com/daltyboy11/more-evm-puzzles)
- 💻 Zaryab Afser，["Decipher EVM puzzles."](https://www.decipherclub.com/decipher-evm-puzzles-game/)

## 实现

- 💻 Solidity: Brock Elmore，["solvm: EVM implemented in solidity."](https://github.com/brockelmore/solvm)
- 💻 Go: [Geth](https://github.com/ethereum/go-ethereum)
- 💻 C++: [EVMONE](https://github.com/ethereum/evmone)
- 💻 Python: [py-evm](https://github.com/ethereum/py-evm)
- 💻 Rust: [revm](https://github.com/bluealloy/revm)
- 💻 Js/CSS: Riley，["The Ethereum Virtual Machine."](https://github.com/jtriley-eth/the-ethereum-virtual-machine)

### 基于 EVM 的编程语言

- 🗄 [Solidity](https://soliditylang.org/)
- 🗄 [Huff](https://github.com/huff-language/)
- 🗄 [Vyper](https://docs.vyperlang.org/en/stable/)
- 🗄 [Fe](https://fe-lang.org/)
