# 轻客户端 (Light Clients)

> :warning: 本文是一个[存根 (stub)](https://en.wikipedia.org/wiki/Wikipedia:Stub)，请通过[贡献 (contributing)](/contributing.md)和扩充它来帮助本维基 (wiki)。

以太坊 (Ethereum) 用户通常使用执行客户端 (execution client) 的远程过程调用 (Remote Procedure Call, RPC) 连接到网络。这允许他们与网络交互、读取余额 (balances)、提交交易 (transactions) 等。运行客户端并验证当前状态 (state) 可能是一项极其繁重的任务，需要数百 GB 的存储空间、带宽和计算能力。大多数钱包 (wallets) 默认使用第三方应用程序接口 (Application Programming Interface, API) 来连接到网络，而不会验证所提供的数据。

轻客户端 (light client) 的核心思想是在无需运行全节点 (full node) 带来的高昂开销的情况下，实现对网络的去信任化 (trustless) 访问。轻客户端是该概念的通用术语，但实际的实现方法采用了不同的设计。存在多种类型的轻客户端，有些已经投入生产，有些仍处于研究与开发 (development) 阶段。

- 使用来自共识层 (Consensus Layer, CL) 客户端 (client) 的信标根 (Beacon root) 验证执行层 (Execution Layer, EL) RPC 数据
- 无状态客户端 (Stateless clients)
- 轻量级以太坊子协议 (Light Ethereum Subprotocol, LES) 协议 (LES protocol)
- 门户网络 (Portal Network)

## RPC 代理轻客户端 (RPC proxy light client)

这类轻客户端 (light clients) 连接到 RPC 提供者 (RPC provider)，并通过使用来自独立信标节点 (Beacon Node) 的证明来验证响应，从而提高安全性。它基本上是一个 RPC 代理或中间件 (middleware)，用于确保来自提供者的数据是有效的。
它改善了连接到第三方 RPC 的钱包 (wallet) 或服务 (service) 的信任模型 (trust model)，但它本身并不作为网络中的节点运行。通过这种轻客户端方法，用户仍然需要连接到某个作为中心化实体的 RPC 提供者 (RPC provider)。

在网络中通过点对点 (peer-to-peer, p2p) 协议 (protocol) 进行通信的客户端，并没有针对特定数据片段的特定功能（不像 RPC 那样）。它们可以从对等节点 (peer) 获取当前的最新区块提示 (tip)、请求历史区块 (blocks) 等。而为了验证这些数据，它们还需要连接到一个共识客户端 (consensus client)。无法直接通过 P2P 请求某个地址 (address) 的余额 (balance)，只能下载区块/状态 (blocks/state)，进行验证并自己查找。通过这种方法，我们基本上又回到了网络中普通节点的行为模式。

这种执行 RPC 验证的“轻客户端”实现包括例如 [Helios](https://github.com/a16z/helios) 或 [Kevlar](https://github.com/lightclients/kevlar)。用户可以将它们作为应用/钱包与 RPC 提供者之间的代理运行。它们提供到公共信标节点的默认连接，因此这两个提供者以完全相同的方式撒谎的可能性微乎其微。曾经有一个[试图在 Helios 中实现共识层 P2P (CL p2p) 的项目](ttps://github.com/eth-protocol-fellows/cohort-three/blob/master/projects/helios-cl-p2p.md)，以便直接使用共识层 libp2p (cl libp2p)，而不是依赖第三方的信标 API (Beacon API)。

## 无状态客户端 (Stateless clients)

使用在 P2P 网络中传播 (gossiped) 的见证数据 (witnesses) 来在没有完整状态的情况下验证数据。

## 门户网络 (Portal Network)

门户网络 (Portal) 创建了一个覆盖网络 (overlay network)，以概率性地 (probabilistically) 保证数据完整性 (data integrity)。

## 轻量级以太坊子协议 (Light Ethereum Subprotocol, LES)

由 Geth 首创的轻客户端模式 (Light client mode) 允许在轻量配置 (light config) 下运行一个节点，该节点订阅 les P2P 协议 (les p2p protocol)。该节点不下载整条链，仅从其他提供 les 服务的节点下载最新数据。全节点 (full node) 需要配置为提供 les 数据，这并不是默认选项。因此，网络中没有足够的 les 提供者 (les providers) 来使 Geth 在轻模式 (light mode) 下可靠运行。