# 执行客户端 (Execution Client)

> **执行客户端 (Execution Clients)**，前身为 *eth1 客户端*，是实现了以太坊[执行层 (Execution Layer)](https://github.com/ethereum/execution-specs)规范的软件，其任务是处理和广播交易，以及管理全局状态。
它们通过点对点 (P2P) 网络接收交易，使用[以太坊虚拟机 (Ethereum Virtual Machine, EVM)](https://ethereum.org/en/developers/docs/evm/)为每笔交易运行计算以更新状态，并确保遵循协议的所有规则。
执行客户端可以配置为存储全局状态和包含收据在内的历史区块链数据的全节点 (Full Node)，或者配置为保留所有历史状态的归档节点 (Archive Node)。

## 概述表 (Overview Table)

当前在生产环境中使用的执行客户端有：

| 客户端 (Client) | 语言 (Language) | 开发团队 (Developer) | 状态 (Status) |
|-------------|----------|-------------------|------------|
| [Besu](https://github.com/hyperledger/besu) | Java | Hyperledger | 生产环境 (Production) |
| [Erigon](https://github.com/ledgerwatch/erigon) | Go | Ledgerwatch | 生产环境 (Production) |
| [Geth](https://github.com/ethereum/go-ethereum) | Go | 以太坊基金会 (Ethereum Foundation) | 生产环境 (Production) |
| [Nethermind](https://github.com/NethermindEth/nethermind) | C# | Nethermind | 生产环境 (Production) |
| [Reth](https://github.com/paradigmxyz/reth) | Rust | Paradigm | 生产环境 (Production) |

还有更多处于活跃开发阶段、尚未达到成熟状态，或者在过去曾被使用过的执行客户端：

| 客户端 (Client) | 语言 (Language) | 开发团队 (Developer) | 状态 (Status) |
| --------------------------------------------------------------- | ---------- | ------------------- | ----------- |
| [Nimbus](https://github.com/status-im/nimbus-eth1) | Nim | Nimbus | 开发中 (Development) |
| [Silkworm](https://github.com/erigontech/silkworm) | C++ | Erigon | 开发中 (Development) |
| [JS Client](https://github.com/ethereumjs/ethereumjs-monorepo) | Typescript | 以太坊基金会 (Ethereum Foundation) | 开发中 (Development) |
| [ethrex](https://github.com/lambdaclass/ethrex) | Rust | LambdaClass | 开发中 (Development) |
| [Akula](https://github.com/akula-bft/akula) | Rust | Akula 开发者 (Akula Developers) | 已废弃 (Deprecated) |
| [Aleth](https://github.com/ethereum/aleth) | C++ | Aleth 开发者 (Aleth Developers) | 已废弃 (Deprecated) |
| [Mana](https://github.com/mana-ethereum/mana) | Elixir | Mana 开发者 (Mana Developers) | 已废弃 (Deprecated) |
| [OpenEthereum](https://github.com/openethereum/parity-ethereum) | Rust | Parity | 已废弃 (Deprecated) |
| [Trinity](https://github.com/ethereum/trinity) | Python | OpenEthereum | 已废弃 (Deprecated) |


## 客户端分布 (Distribution)

当前绝大多数节点运营商都在使用 Geth 作为其执行客户端。
为了支持执行层 (Execution Layer, EL) 的健康发展，[强烈建议在运行节点时使用不同的客户端以促进多样性](https://clientdiversity.org/#why)。

## 各个客户端 (Individual clients)

尽管所有客户端都实现了相同的规范，但每个客户端都提供了独特的功能和优势。它们使用不同的编程语言编写，这使得具有不同背景的开发者都能够做出贡献。

### Besu

Besu (Hyperledger Besu) 由 Consensys/Hyperledger 基金会使用 Java 开发，因其企业级功能以及与各种 Hyperledger 项目的兼容性而脱颖而出。
它同时支持公共和私有网络，提供强大的命令行工具和 JSON-RPC API。

值得关注的特性 (Noteworthy Features)：
- [私有网络 (Private Networks)](https://besu.hyperledger.org/private-networks/)
- [修剪 (Pruning)](https://besu.hyperledger.org/public-networks/how-to/bonsai-limit-trie-logs#prune-command-for-mainnet)
- [并行交易执行 (Parallel Transaction Execution)](https://besu.hyperledger.org/public-networks/concepts/parallel-transaction-execution)

### Erigon

Erigon 最初作为 Geth 的分支以 turbo-geth 的名称被引入，它专注于优化性能、提供快速同步能力并减少磁盘空间占用。Erigon 引入了一种管理 MPT 数据库的新方法，从而将归档节点的磁盘空间占用了大约 5 倍。
Erigon 的架构允许它在不到三天的时间内以少于 3 TB 的数据存储完成完整的归档节点同步，这使其成为运行归档节点的理想选择。它还包含了自己的嵌入式共识层客户端 (Caplin CL Client)，使其能够独立运行。

值得关注的特性 (Noteworthy Features)：
- [支持的网络 (Supported Networks)](https://erigon.gitbook.io/erigon/basic-usage/supported-networks)
- [修剪 (Pruning)](https://erigon.gitbook.io/erigon/basic-usage/usage/type-of-node#full-node-or-pruned-node)
- [Caplin 共识层客户端 (Caplin CL Client)](https://erigon.gitbook.io/erigon/advanced-usage/consensus-layer/caplin)

### Geth

作为以太坊的原始 Go 语言实现以及最古老的、一直活跃维护的客户端，Geth (Go-Ethereum) 在开发者和用户中都获得了极其广泛的采用。
它支持各种节点类型（全节点、轻节点、归档节点），并以其丰富的工具集、卓越的稳定性和强大的社区支持而闻名。
Geth 在部署上的灵活性——通过包管理器、Docker 容器或手动设置——确保了它在各种区块链环境中的多功能性。

值得关注的特性 (Noteworthy Features)：
- [修剪 (Pruning)](https://geth.ethereum.org/docs/fundamentals/pruning)
- [自定义 EVM 追踪器 (Custom EVM Tracer)](https://geth.ethereum.org/docs/developers/evm-tracing/custom-tracer)
- [监控仪表板 (Monitoring Dashboards)](https://geth.ethereum.org/docs/monitoring/dashboards)

### Nethermind

Nethermind 使用 C# .NET 编写，专为高稳定性和与现有技术基础设施的深度集成而设计。
它提供了优化的虚拟机性能、全面的分析支持和灵活的插件系统。
Nethermind 适用于私有以太坊网络和去中心化应用 (dApp) 开发，并强调数据完整性和性能可扩展性。

值得关注的特性 (Noteworthy Features)：
- [私有网络 (Private Networks)](https://docs.nethermind.io/fundamentals/private-networks)
- [性能调优 (Performance tuning)](https://docs.nethermind.io/fundamentals/performance-tuning)
- [Prometheus 和 Grafana 监控 (Prometheus and Grafana)](https://docs.nethermind.io/monitoring/metrics/grafana-and-prometheus)

### Reth

Reth (Rust Ethereum) 是一种模块化且高效的以太坊客户端，专为用户友好和超高性能而设计。受 Erigon 设计的启发，它围绕一种新颖的归档节点方案构建，该方案能够以极小的磁盘空间实现极其快速的同步。
它强调社区驱动的开发，并且非常适用于强健的生产环境。

值得关注的特性 (Noteworthy Features)：
- [Revm 虚拟机 (Revm)](https://bluealloy.github.io/revm/)
- [监控 (Monitoring)](https://reth.rs/run/observability.html)

### Nimbus

Nimbus 专注于作为超轻量级以太坊执行层客户端的高效性和安全性。最初该团队致力于开发 Nimbus 共识层客户端，随后分出一个新团队来使用 Nim 语言开发执行客户端。
它在支持以太坊执行层功能的同时将资源消耗降至最低，并能够与 Fluffy（一个门户网络 (Portal Network) 轻客户端）进行集成。
Nimbus 提供了卓越的内存节省和状态同步机制，非常适合资源受限的环境。

### Silkworm

Silkworm 是以太坊执行层协议的 C++ 实现，旨在成为速度最快的以太坊客户端。
它集成了 libmdbx 数据库引擎，并在 Erigon 项目（也称为 Erigon++）中强调了可扩展性、模块化和性能优化。

### JS 客户端 (JS Client)

JavaScript 客户端由以太坊基金会 JavaScript 团队开发，是 [EthereumJS 单体仓库 (EthereumJS Monorepo)](https://github.com/ethereumjs/ethereumjs-monorepo) 的一部分。它具有实验性质，主要用于测试，但由于其以 JavaScript 为中心的设计，它非常适用于 Web 浏览器端和 Node.js 环境。

## 附加资源 (Additional resources)

- [ETH Docker 配置 (ETH Docker)](https://eth-docker.net/)
- [以太坊节点统计 (Ethernodes)](https://ethernodes.org/)
- [客户端多样性倡议 (Client Diversity)](https://clientdiversity.org/)
- [“自担风险运行多数派客户端！” (Run the majority client at your own peril!)](https://dankradfeist.de/ethereum/2022/03/24/)
