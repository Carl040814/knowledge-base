# 协议架构概述 (Protocol Architecture Overview)

> :warning: 本文是一个待完善的草稿 (stub)，欢迎通过 [做出贡献](/contributing.md) 并对其进行扩展来帮助维基。

当前的协议架构是多年演进的结果。该协议由两个主要部分组成——执行层 (execution layer) 和共识层 (consensus layer)。执行层 (execution layer, EL) 处理实际的交易和用户交互，是全球计算机执行其程序的地方。共识层 (consensus layer, CL) 提供了权益证明 (Proof-of-Stake, PoS) 共识机制——这是一种加密经济安全机制 (cryptoeconomic security mechanism)，可确保所有节点都遵循相同的链梢并驱动执行层的规范链 (canonical chain)。

在实践中，这些层在各自通过 API 连接的客户端 (clients) 中实现。每个层都有自己的点对点网络 (p2p network)，用以处理不同类型的数据。

![](./img/clients-overview.png)

深入探索每个客户端的内部结构，它们由许多基本功能组成：

![](./img/protocol-overview.png)
