# EVM 对象格式 (EOF)

EVM 对象格式 (EVM Object Format, EOF) 是一个可扩展且带版本控制的 EVM 字节码容器格式，在部署时进行一次性验证。

目前，[EVM 字节码](/wiki/EL/evm.md#evm-bytecode)是指令的无结构序列。EOF 引入了容器的概念，为字节码带来结构，使其更容易被 EVM 解析和分析。其主要目标是通过允许在部署时进行一次性代码验证，并严格分离代码和数据，来实现 EVM 的重大升级。

例如，EOF1 字节码以 2 字节魔数 `0xEF00` 和版本号开头，后跟节头列表和一个 `0x00` 终止符。这种结构允许 EVM 实现在任何执行开始之前，在部署时验证整个合约的正确性，包括栈深度和结构有效性。
```
// EOF 容器（字节码）的示例布局：
魔数 (0xEF00) | 版本 | (section_kind, section_size_or_sizes)+ | 0x00
[类型节] [代码节 0] [代码节 1] … [数据节]
```

---

## 目的和优势

EOF 旨在解决原始 EVM 字节码格式的若干限制：

- **更好的验证**：在智能合约执行之前实现彻底的静态分析和验证。它拒绝截断的 PUSH 数据或未定义的操作码，使每个 EOF 合约在链上可证明有效。

- **改进的执行效率**：通过更好定义的代码段和移除运行时的 JUMPDEST 扫描，有利于优化 EVM 执行。

- **增强的安全性**：通过执行更严格的代码结构规则并通过验证消除遗留怪异行为（如 SELFDESTRUCT 或 DELEGATECALL 问题）来减少攻击向量。

- **代码/数据分离**：通过将数据（例如编译器元数据）保存在单独的节中，验证器和 L2 等工具可以轻松区分可执行代码和任意数据，从而提高安全性和 gas 效率。

---

## 相关 EIP 和实现

EOF 在多个 EIP 中规定，涵盖不同功能、对各种操作码的修改以及引入新操作码。所有这些 EIP 由一个元 EIP 跟踪：[EIP-7692](https://eips.ethereum.org/EIPS/eip-7692)。

### 核心容器和验证

- [**EIP-3540**](https://eips.ethereum.org/EIPS/eip-3540)：EOF V1 基础规范。定义容器、头部和代码/数据分离的"切实好处"。

- [**EIP-3670**](https://eips.ethereum.org/EIPS/eip-3670)：强制在创建时进行代码验证，确保没有无效操作码被部署。

- [**EIP-5450**](https://eips.ethereum.org/EIPS/eip-5450)：引入栈验证 (Stack Validation) 以防止运行时栈错误。

### 控制流和子程序

- [**EIP-4200**](https://eips.ethereum.org/EIPS/eip-4200)：静态相对跳转。添加 `RJUMP`、`RJUMPI` 和 `RJUMPV`，使用带符号立即偏移量，消除了对动态 `JUMP` 和 `JUMPDEST` 的需求。

- [**EIP-4750**](https://eips.ethereum.org/EIPS/eip-4750)：EOF 函数。支持多个代码段，并引入 `CALLF`/`RETF` 用于合约内函数调用。还添加了列出每个函数输入/输出的类型节。

- [**EIP-6206**](https://eips.ethereum.org/EIPS/eip-6206)：为尾调用跳转添加 `JUMPF`（操作码 `0xE5`），并允许将函数标记为不返回（输出 `0x80`）以进行优化。

### 现代化指令

- [**EIP-7480**](https://eips.ethereum.org/EIPS/eip-7480)：数据节访问。定义 `DATALOAD`、`DATALOADN`、`DATASIZE` 和 `DATACOPY` 来安全访问数据。EOF 中废弃了传统的 `CODECOPY`。

- [**EIP-663**](https://eips.ethereum.org/EIPS/eip-663)：无限制栈访问。添加 `SWAPN`、`DUPN` 和 `EXCHANGE`，允许编译器访问 1024 项栈中最多 256+ 项。

- [**EIP-7069**](https://eips.ethereum.org/EIPS/eip-7069)：改进的 CALL。用 `EXTCALL`、`EXTDELEGATECALL` 和 `EXTSTATICCALL` 替换传统的 `CALL`/`DELEGATECALL`（移除 gas 参数并使用 63/64 规则）。

- [**EIP-5920**](https://eips.ethereum.org/EIPS/eip-5920)：引入 `PAY` 操作码（`0xFC`），允许 ETH 转账而不调用接收者的代码，减轻重入风险。

- [**EIP-7761**](https://eips.ethereum.org/EIPS/eip-7761) 和 [**EIP-7880**](https://eips.ethereum.org/EIPS/eip-7880)：添加 `EXTCODETYPE` 检查账户类型（EOA vs. EOF）和 `EXTCODEADDRESS` 解析 EIP-7702 委托。

### 合约创建和元数据

- [**EIP-7620**](https://eips.ethereum.org/EIPS/eip-7620)：用 `EOFCREATE` 和 `RETURNCODE` 替换 `CREATE`/`CREATE2`。

- [**EIP-7873**](https://eips.ethereum.org/EIPS/eip-7873)：替换了之前的 EIP-7698。定义 `TXCREATE` 操作码和 InitcodeTransaction，用于从 EOA 直接部署 EOF。

- [**EIP-7834**](https://eips.ethereum.org/EIPS/eip-7834)：添加单独的元数据节 (Metadata Section)（种类 `0x05`），代码无法访问，非常适合 CBOR 元数据或 IPFS 哈希，无需移动代码偏移量。

---

## 与传统 EVM 字节码的区别

| 功能 | 传统 EVM | EOF |
|---|---|---|
| 容器结构 | 无结构字节码 | 明确定义的节（类型、代码、数据） |
| 代码验证 | 有限、运行时 | 全面、执行前（部署时） |
| 跳转机制 | 动态：`JUMP` / `JUMPI` | 静态相对跳转：`RJUMP` / `RJUMPI` |
| 栈验证 | 仅运行时检查 | 静态验证（无下溢/上溢） |
| 类型 | 无 | 函数签名和类型 |
| 数据处理 | 与代码混合 | 单独的数据节 (`DATALOAD`) |
| 内省 | `CODECOPY`, `EXTCODESIZE` | 已废弃；被更安全的替代方案取代 |

---

## 当前状态

EOF 于 2025 年 4 月 28 日在 [Interop Testing #34 会议](https://ethereum-magicians.org/t/interop-testing-34-april-28-2025/23822/2)期间正式从 Fusaka 硬分叉中移除。该决定由 Tim Beiko 在[更新 EIP-7607 的 GitHub PR](https://github.com/ethereum/EIPs/pull/9703) 中确认。

移除的主要原因有：

- **复杂性**：包括开发者 Pascal Caversaccio 在[2025 年 3 月的 Ethereum Magicians 帖子](https://ethereum-magicians.org/t/evm-object-format-eof/5727)中的批评者认为，EOF 引入了太多同时进行的变更、新语义、十几个操作码的添加和移除，其好处可以通过更小、更有针对性的更新来实现。
- **维护负担**：发布 EOF 需要无限期地维护传统 EVM 和 EOF 合约，增加客户端团队的长期复杂性。
- **流程失败**：Beiko 在移除 PR 中承认，ACD 流程未能在多个升级周期中充分解决社区反对意见，尽管核心开发者多次同意发布它。
- **时间线风险**：由于 PeerDAS 是 Fusaka 的主要优先事项，围绕 EOF 变体的持续规范辩论被视为升级整体进度的一个风险。

EOF 的倡导者仍有机会为随后的 **Glamsterdam** 硬分叉提出纳入方案，但截至目前尚无纳入计划。规范及其组成 EIP 仍作为历史参考提供。

---

## 资源

- [**EVM | Pawel Bylica | Lecture 17**](https://www.youtube.com/watch?v=gYnx_YQS8cM) — EOF 内部机制和设计决策的详细技术深入解析。

- [**Ethereum Magicians: EVM Object Format (EOF)**](https://ethereum-magicians.org/t/evm-object-format-eof/5727) — 核心开发者提出并辩论 EOF 的原始 EIP 讨论帖。

- [**Ethereum Notes: EOF**](https://notes.ethereum.org/t-1tLFnLSKCtLZpb-Rw0IA) — 关于 EOF 规范和开放设计问题的社区笔记和工作文档。
