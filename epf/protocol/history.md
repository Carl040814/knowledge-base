# 协议历史与演进 (Protocol history and evolution)

> :warning: 本文是一个待完善的草稿 (stub)，欢迎通过 [做出贡献](/contributing.md) 并对其进行扩展来帮助维基。

本页面重点介绍了以太坊协议历史上的重要技术变化。

有用链接：[来自 Ethereum.org 的概述](https://ethereum.org/en/history) 和 [来自 Ethereum.org 的 Meta EIPs](https://eips.ethereum.org/meta)

## 前沿 (Frontier)

前沿 (Frontier) 版本的发布标志着以太坊协议 (Ethereum Protocol) 的启动。该版本本质上是一个测试版，允许开发者学习、实验并开始构建以太坊去中心化应用和工具。它于 UTC 时间 2015 年 7 月 30 日凌晨 3:26:13 启动，这是 [以太坊创世区块 (Ethereum genesis block)](https://etherscan.io/block/0) 的时间戳。Frontier 启动时的 Gas 限制 (gas limit) 为 5000。这个 Gas 限制被硬编码在协议中，以确保矿工和用户在协议初始启动期间可以通过安装客户端来正常运行。随后，在“前沿解冻分叉 (Frontier thawing fork)”中，Gas 限制被提高到了 300 万（准确地说是 3,141,592）。“金丝雀合约 (canary contracts)”是给出 0 或 1 二进制信号的合约。这些金丝雀合约是仅在以太坊前沿阶段使用的初始启动机制。如果客户端检测到多个合约已切换到信号 1，它们将停止挖矿并督促用户更新其客户端。这通过确保矿工不会阻碍链升级来防止长时间的网络中断。

通过以下资源了解有关前沿 (Frontier) 的更多信息：

- [Frontier is coming, what to expect and how to prepare](https://blog.ethereum.org/2015/07/22/frontier-is-coming-what-to-expect-and-how-to-prepare)
- [The thawing frontier](https://blog.ethereum.org/2015/08/04/the-thawing-frontier)
- [ethereum.org web archive](https://web.archive.org/web/20150802035735/https://www.ethereum.org/)
- [ethereum-protocol-update-1](https://blog.ethereum.org/2015/08/04/ethereum-protocol-update-1)

## 家园 (Homestead)

家园 (Homestead) 是以太坊协议的第二个主要版本，于 2016 年 3 月 14 日正式发布，标志着以太坊从测试阶段向更成熟、更稳定平台的过渡。
以下是家园 (Homestead) 阶段引入的一些显著功能和变化：

- [EIP-2](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-2.md)：EIP-2 包含许多修复程序，例如增加通过交易创建合约的 Gas 成本，确保合约创建要么成功要么失败（防止创建空合约），以及对难度调整算法 (difficulty adjustment algorithm) 的修改。

  1. **增加合约创建的 Gas 成本**：
     通过交易创建合约的 Gas 成本从 21,000 增加到 53,000。
     这一变化旨在减少通过交易创建合约的过度激励，而鼓励通过合约内的 `CREATE` 操作码 (opcode) 来创建，后者未受影响。
  2. **高 s 值签名的无效化**：
     s 值大于 `secp256k1n/2` 的交易签名现在被视为无效。
     这一举措解决了交易可延展性 (transaction malleability) 问题，防止通过翻转 s 值（`s` -> `secp256k1n - s`）来篡改交易哈希 (transaction hashes)。
     这一变化提高了交易跟踪的可靠性和完整性。
  3. **合约创建 Gas 耗尽处理**：
     如果合约创建没有足够的 Gas 来支付将合约代码添加到状态中的最终 Gas 费用，合约创建将失败（即 Gas 耗尽 (out-of-gas)），而不是留下一个空合约。
  4. **更改难度调整算法**：
     难度调整算法被修改，以解决在前沿阶段观察到的问题。
     新公式旨在通过根据区块之间的时间戳差异调整难度，来维持目标区块时间并防止过度偏差。

- [EIP-7](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7.md)：EIP-7 引入了 `DELEGATECALL` 操作码。

  在 `0xf4` 处添加了一个新的操作码 `DELEGATECALL`。
  它的功能类似于 `CALLCODE`，但会将发送者 (sender) 和数值 (value) 从父作用域传播到子作用域。
  传播发送者和数值使合约更容易将另一个地址存储为可变的代码源，并将调用“透传”给它。
  与 `CALL` 操作码不同，它没有额外添加的 Gas 津贴，这使得 Gas 管理更加可预测。

- [EIP-8](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-8.md)：EIP-8 通过引入 devp2p 向前兼容性要求，确保以太坊上的客户端支持未来的网络升级。

  **devp2p 有线协议 (devp2p Wire Protocol)**、**RLPx 发现协议 (RLPx Discovery Protocol)** 和 **RLPx TCP 传输协议 (RLPx TCP Transport Protocol)** 规定，实现应该在接受数据包时保持宽容：忽略 hello 和 ping 数据包中的版本号和附加列表元素，默默丢弃未知的数据包类型，并接受加密密钥建立握手数据包的新编码。
  这确保了所有客户端软件都能应对未来的协议升级并接受握手，从而允许宽容地接受来自他人的数据（参见 [波斯塔尔定律 / Postel's Law](https://en.wikipedia.org/wiki/Robustness_principle)）。

通过以下资源了解有关家园 (Homestead) 的更多信息：

- [Ethereum Homestead Documentation](https://readthedocs.org/projects/ethereum-homestead/downloads/pdf/latest/)
- [The Robustness Principle Reconsidered](https://queue.acm.org/detail.cfm?id=1999945)
- [Homestead blog release post](https://blog.ethereum.org/2016/02/29/homestead-release)
- [The Homestead release - github](https://github.com/ethereum/homestead-guide/blob/master/source/introduction/the-homestead-release.rst)

## 合并 (The Merge)

2022 年 9 月 15 日，以太坊激活了 [EIP-3675](https://eips.ethereum.org/EIPS/eip-3675)，并通过被称为“合并 (The Merge)”的事件将共识机制升级为权益证明 (Proof-of-Stake, PoS)。合并导致了工作量证明 (Proof-of-Work, PoW) 共识的弃用，该共识以前与执行在相同的逻辑层中实现。相反，它已被更复杂、更精细的权益证明共识所取代，消除了对高能耗挖矿的需求。新的权益证明共识已在自己的层中实现，具有独立的点对点网络 (p2p network) 和逻辑，也称为信标链 (Beacon Chain)。信标链自 2020 年 12 月 1 日起一直在运行并达成共识。在经历了长时间没有发生任何故障的稳定表现后，它被认为已准备好成为以太坊的共识提供者 (consensus provider)。“合并 (The Merge)”因这两个网络的联合而得名。

在以下资源和关于共识层的阅读中了解有关合并的更多信息。

 - [EIP-3675: Upgrade consensus to Proof-of-Stake](https://eips.ethereum.org/EIPS/eip-3675), [archived](https://web.archive.org/web/20240213102133/https://eips.ethereum.org/EIPS/eip-3675)
- [Gasper](https://ethereum.org/developers/docs/consensus-mechanisms/pos/gasper), [archived](https://web.archive.org/web/20240214225630/https://ethereum.org/developers/docs/consensus-mechanisms/pos/gasper)
- [Mega Merge Resources List](https://notes.ethereum.org/@MarioHavel/merge-resources), [archived](https://web.archive.org/web/20240302082121/https://notes.ethereum.org/@MarioHavel/merge-resources)
