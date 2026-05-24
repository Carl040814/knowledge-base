# 预编译合约 (Precompiled Contracts)

预编译合约 (precompiled contracts) 是一组特殊账户，每个账户包含一个内置函数，具有确定的 gas 成本，通常与复杂的密码学计算相关。目前，它们定义在 `0x01` 到 `0x0a` 之间的地址上。

与合约账户不同，预编译合约是以太坊协议的一部分，由[执行客户端](/wiki/EL/el-clients.md)实现。这带来了一个有趣的特性——它们在 EVM 中的 `EXTCODESIZE` 为 0。尽管如此，执行时它们的功能类似于具有代码的合约账户。

根据 [EIP-2929](https://eips.ethereum.org/EIPS/eip-2929) 的定义，预编译合约被包含在交易的初始 `"accessed_addresses"` 中，以节省 gas 成本。

## 预编译合约 vs. 操作码

预编译合约和操作码 (opcode) 都旨在执行任意计算。在选择时需要考虑以下几点：

- **有限的操作码空间**：EVM 的 1 字节操作码空间本质上受限（256 个操作码，`0x00`-`0xFF`）。这需要对基本操作进行审慎的分配。
- **效率**：预编译合约在 EVM 外部以原生方式执行，能够为许多跨链交互提供动力的复杂密码学计算提供高效实现。

引用 Vitalik 的 ["A Prehistory of the Ethereum Protocol"](https://vitalik.eth.limo/general/2017/09/14/prehistory.html)：

> 第二个是"预编译"的概念，解决了允许复杂密码学计算在 EVM 中使用而不必处理 EVM 开销的问题。我们还经历了许多更雄心勃勃的关于"原生合约"的想法，即如果矿工对某些合约有优化的实现，他们可以"投票"降低这些合约的 gas 价格，这样大多数矿工可以更快执行的合约自然会有更低的 gas 价格；然而，所有这些想法都被拒绝了，因为我们无法想出一种密码经济学上安全的方式来实现这样的事情。攻击者总是可以创建一个合约，执行某种带有后门的密码学操作，将后门分配给自己和他们的朋友，让他们能够更快地执行这个合约，然后投票降低 gas 价格，并利用这一点对网络进行 DoS 攻击。相反，我们选择了不那么雄心勃勃的方法，即在协议中指定较少数量的预编译合约，用于常见操作，如哈希和签名方案。

## 预编译合约列表

| 地址 | 名称                 | 描述                                | 起始版本                                                         |
| ------- | -------------------- | ------------------------------------------ | ------------------------------------------------------------- |
| 0x01    | ECRECOVER            | 椭圆曲线公钥恢复 (Elliptic curve public key recover)          | Frontier                                                      |
| 0x02    | SHA2-256             | SHA2 256 位哈希方案                   | Frontier                                                      |
| 0x03    | RIPEMD-160           | RIPEMD 160 位哈希方案                 | Frontier                                                      |
| 0x04    | IDENTITY             | 恒等函数 (Identity function)                          | Frontier                                                      |
| 0x05    | MODEXP               | 任意精度模幂运算 (Arbitrary precision modular exponentiation) | Byzantium ([EIP-198](https://eips.ethereum.org/EIPS/eip-198)) |
| 0x06    | ECADD                | 椭圆曲线加法 (Elliptic curve addition)                    | Byzantium ([EIP-196](https://eips.ethereum.org/EIPS/eip-196)) |
| 0x07    | ECMUL                | 椭圆曲线标量乘法 (Elliptic curve scalar multiplication)       | Byzantium ([EIP-196](https://eips.ethereum.org/EIPS/eip-196)) |
| 0x08    | ECPAIRING            | 椭圆曲线配对检查 (Elliptic curve pairing check)               | Byzantium ([EIP-197](https://eips.ethereum.org/EIPS/eip-197)) |
| 0x09    | BLAKE2               | BLAKE2 压缩函数                | Istanbul ([EIP-152](https://eips.ethereum.org/EIPS/eip-152))  |
| 0x0a    | KZG POINT EVALUATION | 验证 KZG 证明                       | Cancun ([EIP-4844](https://eips.ethereum.org/EIPS/eip-4844))  |

## 工作原理

预编译合约的优雅之处在于其接口设计，与外部智能合约调用完全相同，允许熟悉的交互——从开发者的角度看，使用预编译合约与外部调用没有区别。

预编译合约的 gas 成本与输入数据直接相关——固定输入对应固定成本。为确定这些成本，开发者依赖参考实现和基准测试的结合。基准测试通常测量特定硬件上的执行时间，而像 `MODEXP` 这样的则[直接以每秒 gas 使用量来定义消耗](https://eips.ethereum.org/EIPS/eip-2565#1-modify-computational-complexity-formula-to-better-reflect-the-computational-complexity)。这种细致的方法旨在通过确保可预测的资源分配来防止拒绝服务攻击 (DoS attack)。

在底层，客户端实现利用优化的库来执行预编译合约。虽然这种方法提高了效率，但也引入了潜在的安全风险。如果这些库中发现漏洞，可能会破坏整个协议层。为减轻这种风险，严格的[测试](/wiki/testing/overview.md)至关重要（例如 [MODEXP 测试规范](https://github.com/ethereum/execution-spec-tests/tree/main/tests/byzantium/eip198_modexp_precompile)）。

为防止安全漏洞，预编译合约被设计为避免嵌套调用 (nested call)。

## 调用预编译合约

像合约账户一样，预编译合约可以使用 `*CALL` 系列操作码调用。以下汇编代码示例展示了使用 `SHA-256` 预编译合约来哈希字符串 "Hello"：

```js
// 首先将参数放在内存中
PUSH5 0x48656C6C6F // UTF-8 编码的 Hello
PUSH1 0
MSTORE

// 调用 SHA-256 预编译合约 (0x02)
PUSH1 0x20 // retSize
PUSH1 0x20 // retOffset
PUSH1 5 // argsSize
PUSH1 0x1B // argsOffset
PUSH1 2 // address
PUSH4 0xFFFFFFFF // gas
STATICCALL

POP // 弹出 STATICCALL 的结果

// 将结果从内存加载到栈
PUSH1 0x20
MLOAD
```

产生的哈希为：

```
185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969
```

> ▶️ [在 EVM playground 中尝试！](https://www.evm.codes/playground?fork=cancun&unit=Wei&codeType=Mnemonic&code=***

参考 [EVM](/wiki/EL/evm.md) 的 wiki 页面了解汇编代码的工作原理。

## 提议中的预编译合约

[EIP](https://eips.ethereum.org/) 可以在硬分叉 (hard fork) 中引入新的预编译合约。由于测试面和维护成本高，增加新的预编译合约通常会遇到阻力。为解决这个问题，一种提议的方法涉及先在 Layer 2 解决方案上原型化预编译合约，然后仅在它们展示出稳定性和广泛采用后再集成到主网。

目前提议的预编译合约有：

- [EIP-2537: BLS12-381 曲线操作预编译](https://eips.ethereum.org/EIPS/eip-2537)
- [RIP-7212: secp256r1 曲线支持预编译](https://github.com/ethereum/RIPs/blob/master/RIPS/rip-7212.md)
- [EIP-7545: Verkle 证明验证预编译](https://eips.ethereum.org/EIPS/eip-7545)
- [EIP-5988: 添加 Poseidon 哈希函数预编译](https://eips.ethereum.org/EIPS/eip-5988)

引入新的预编译合约需要仔细考虑其网络效应。gas 成本计算错误的预编译合约可能通过消耗比预期更多的资源而导致拒绝服务。此外，越来越多的预编译合约可能导致 EVM 客户端中的代码膨胀，增加验证者的负担。

预编译合约的密码学函数及其对应参数的选择需要彻底分析，以平衡安全性和效率。这些参数通常在预编译逻辑中预设，因为从用户输入参数化可能存在安全问题。此外，为具有广泛参数的优化安全函数对于快速执行来说很困难，而这正是预编译合约的基本要求。

## 移除预编译合约

关于可能移除过时、未充分利用或阻碍客户端软件效率的预编译合约的讨论正在进行中。恒等预编译（已被 `MCOPY` 操作码替代）、`RIPEMD-160` 和 BLAKE 函数是退役的主要候选。

然而，与其完全移除，这些预编译合约可以迁移到高效的智能合约实现。这种方法将确保功能继续可用，但相应的 gas 成本会增加。

## 实现

- [Besu - `org.hyperledger.besu.evm.precompile` 包](https://github.com/hyperledger/besu/tree/3d5f45c35ffce4b5173b2ce5972827f9634317d6/evm/src/main/java/org/hyperledger/besu/evm/precompile)
- [Geth - `core/vm/contracts.go`](https://github.com/ethereum/go-ethereum/blob/b2b0e1da8cac279bf0466885d1abdc5d93402f41/core/vm/contracts.go)
- [Nethermind - `Nethermind.EVM.Precompiles` 命名空间](https://github.com/NethermindEth/nethermind/tree/f3edf2503d2637a37f8b509924e10f88491ddd6e/src/Nethermind/Nethermind.Evm/Precompiles)
- [Reth - REVM Precompiles crates](https://github.com/bluealloy/revm/tree/1ca3d39f6a9e9778f8eb0fcb74fe529345a531b4/crates/precompile/src)

## 研究

一种称为["渐进式预编译 (progressive precompiles)"](https://ethereum-magicians.org/t/eip-proposal-create2-contract-factory-precompile-for-deployment-at-consistent-addresses-across-networks/6083/26)的提议方法旨在改进部署过程。这些预编译将驻留在确定性的 CREATE2 地址上，允许用户合约无论预编译在主网还是特定 L2 上是否活跃，都能与同一地址交互。这种方法确保了原生客户端预编译可用时的更平滑过渡。

## 资源

- [附录 E: 以太坊黄皮书](https://ethereum.github.io/yellowpaper/paper.pdf)
- [第 10 周: Danno Ferrin 的预编译概述](/eps/week10-dev.md)
- [EVM 预编译目录](https://github.com/shemnon/precompiles/)
- [Go Ethereum 预编译实现](https://github.com/ethereum/go-ethereum/blob/master/core/vm/contracts.go)
- [以太坊协议前史](https://vitalik.eth.limo/general/2017/09/14/prehistory.html)
- [Stack Exchange: 什么是预编译合约，它们与原生操作码有何不同？](https://ethereum.stackexchange.com/questions/440/whats-a-precompiled-contract-and-how-are-they-different-from-native-opcodes)
- [Stack Exchange: 为什么更多常见算法不作为预编译实现？](https://ethereum.stackexchange.com/questions/155787/why-arent-more-common-algorithms-done-as-precompiles)
- [A call, a precompile and a compiler walk into a bar](https://blog.theredguild.org/a-call-a-precompile-and-a-compiler-walk-into-a-bar/)
