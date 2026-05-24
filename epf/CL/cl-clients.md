# 共识客户端 (Consensus Client)

> **共识客户端 (Consensus Client)**，以前称为 *eth2 客户端*，运行以太坊的权益证明共识算法，使网络能够就信标链 (Beacon Chain) 的头部达成一致。共识客户端不参与验证/广播交易或执行状态转换：这些由执行客户端 (execution client) 完成。共识客户端不对新区块进行证明或提议：这些由验证者客户端 (validator client) 完成，它是共识客户端的一个可选附加组件。

## 概览表

这些客户端使用不同的编程语言开发，提供独特的功能和不同的性能表现。所有客户端都开箱即用地支持以太坊主网 (Mainnet) 以及活跃的测试网。多样化的实现使网络能够受益于客户端多样性 (client diversity)。如果你正在选择要使用的客户端，当前的客户端多样性应该是主要考虑因素之一。

| 客户端                                                                    | 语言         | 开发者                | 状态       |
| ----------------------------------------------------------------------- | ---------- | ------------------- | ---------- |
| [Lighthouse](https://github.com/sigp/lighthouse)                        | Rust       | Sigma Prime         | 生产环境   |
| [Lodestar](https://github.com/ChainSafe/lodestar)                       | TypeScript | ChainSafe           | 生产环境   |
| [Nimbus](https://github.com/status-im/nimbus-eth2)                      | Nim        | Status              | 生产环境   |
| [Prysm](https://github.com/prysmaticlabs/prysm)                         | Go         | Prysmatic Labs      | 生产环境   |
| [Teku](https://github.com/ConsenSys/teku)                               | Java       | ConsenSys           | 生产环境   |
| [Grandine](https://github.com/grandinetech/grandine)                    | Rust       | Grandine Developers | 生产环境   |
| [Caplin](https://github.com/ledgerwatch/erigon)                         | Go         | Erigon              | 开发中     |
| [LambdaClass](https://github.com/lambdaclass/lambda_ethereum_consensus) | Elixir     | LambdaClass         | 开发中     |


## 分布

目前绝大多数节点运营商使用 Prysm 或 Lighthouse 作为共识客户端。为了支持信标链（前身为 ETH2）的健康，建议使用不同的客户端。[为什么？](https://clientdiversity.org/#why)


### Lighthouse
Lighthouse 由 Sigma Prime 用 Rust 编写，强调安全性和性能。它被广泛采用，但需要注意：如果形成超级多数 (supermajority) 可能导致链分裂。Lighthouse 以 Apache 2.0 许可发布，以在生产环境中的健壮性著称。

Lighthouse 为包括 ARM 在内的所有平台提供二进制文件，并允许交叉编译。有便携版本，通过牺牲编译器性能选项换取更好的平台兼容性。发布的二进制文件由 gpg 密钥 `15E66D941F697E28F49381F426416DC3F30674B0` (security@sigmaprime.io) 签名。

值得注意的功能：
- [交叉编译](https://lighthouse-book.sigmaprime.io/installation_cross_compiling.html)
- [罚没保护 (Slashing Protection)](https://lighthouse-book.sigmaprime.io/validator_slashing_protection.html)
- [分身保护 (Doppelganger Protection)](https://lighthouse-book.sigmaprime.io/validator_doppelganger.html)
- [运行 Slasher](https://lighthouse-book.sigmaprime.io/advanced_slasher.html)
- [仅区块提议者模式](https://lighthouse-book.sigmaprime.io/advanced_proposer_only.html)
- [Prometheus 和 Grafana](https://lighthouse-book.sigmaprime.io/api_metrics.html)



### Lodestar
Lodestar 是 ChainSafe 开发的基于 TypeScript 的以太坊共识客户端，专为快速原型设计和浏览器兼容性而定制。它支持信标节点和验证者客户端功能，提供 BLS 和 SSZ 等用于以太坊协议开发的基本库。Lodestar 在 Apache License 2.0 和 GNU Lesser General Public License (LGPL) 下双许可，允许用户在宽松许可和著佐权 (copyleft) 许可模式之间选择。

值得注意的功能：
- [验证者客户端](https://chainsafe.github.io/lodestar/run/validator-management/vc-configuration)
- [MEV 和 Builder 集成](https://chainsafe.github.io/lodestar/run/beacon-management/mev-and-builder-integration)
- [轻客户端 (Light Client)](https://chainsafe.github.io/lodestar/libraries/lightclient-prover/lightclient)
- [证明器 (Prover)](https://chainsafe.github.io/lodestar/libraries/lightclient-prover/prover)
- [Prometheus 和 Grafana](https://chainsafe.github.io/lodestar/run/logging-and-metrics/prometheus-grafana)
- [远程监控](https://chainsafe.github.io/lodestar/run/logging-and-metrics/client-monitoring)

### Prysm
Prysmatic Labs 的 Prysm 客户端用 Go 编写，专注于可用性和可靠性。它包括完整的信标节点实现和验证者客户端，利用 gRPC 进行进程间通信，BoltDB 用于存储，libp2p 用于网络。Prysm 旨在安全地参与以太坊的权益证明共识。

值得注意的功能：
- [验证者客户端](https://docs.prylabs.network/docs/wallet/nondeterministic)
- [配置 MEV Builder](https://docs.prylabs.network/docs/advanced/builder)
- [运行 Slasher](https://docs.prylabs.network/docs/prysm-usage/slasher)
- [Prometheus 和 Grafana](https://docs.prylabs.network/docs/prysm-usage/monitoring/grafana-dashboard)
- [详细的最佳安全实践](https://docs.prylabs.network/docs/security-best-practices)

### Nimbus
Nimbus 由 Status 用 Nim 开发，针对资源效率进行了优化。它支持轻量级设备，如智能手机和 Raspberry Pi，在强大的服务器上运行时为其他任务节省资源。Nimbus 具有集成的验证者客户端支持、远程签名、性能分析工具和强大的验证者监控能力。

值得注意的功能：
- [运行执行客户端](https://nimbus.guide/eth1.html)
- [验证者客户端](https://nimbus.guide/validator-client.html)
- [MEV 和 Builder 集成](https://nimbus.guide/external-block-builder.html)
- [Prometheus 和 Grafana](https://nimbus.guide/metrics-pretty-pictures.html)

### Teku
ConsenSys 的 Teku 是一个基于 Java 的以太坊共识客户端，提供全面的企业级功能。它包括完整的信标节点实现、验证者客户端、用于节点管理的 REST API、用于监控的 Prometheus 指标以及验证者签名密钥的外部密钥管理。Teku 适用于需要可扩展性和运维控制的企业级以太坊部署。

值得注意的功能：
- [验证者客户端](https://docs.teku.consensys.io/concepts/proof-of-stake)
- [罚没保护 (Slashing Protection)](https://docs.teku.consensys.io/how-to/prevent-slashing/use-a-slashing-protection-file)
- [Builder 和 MEV-boost](https://docs.teku.consensys.io/concepts/builder-network)
- [检测分身 (Doppelganger)](https://docs.teku.consensys.io/how-to/prevent-slashing/detect-doppelgangers)
- [Prometheus 和 Grafana](https://docs.teku.consensys.io/how-to/monitor/use-metrics)

### Grandine
Grandine 是一个快速且轻量级的以太坊共识客户端，专注于高性能和简约性。它用 Rust 编写，与 Lighthouse 相同并共享其部分库。Grandine 以并行化和高效的资源利用为核心设计，旨在通过提供现有客户端的精简替代方案来推动以太坊共识层的边界。其架构经过优化，以最小化延迟并最大化高端机器上的吞吐量，使其非常适合需要高性能的环境。

值得注意的功能：
- [验证者客户端](https://docs.grandine.io/validator_client.html)
- [罚没保护 (Slashing Protection)](https://docs.grandine.io/slashing_protection.html)
- [Builder 和 MEV](https://docs.grandine.io/builder_api_and_mev.html)
- [运行 Slasher](https://github.com/grandinetech/grandine/tree/develop/slasher)
- [Prometheus](https://docs.grandine.io/metrics.html) 和 [Grafana](https://github.com/grandinetech/grandine/tree/develop/metrics)

### Caplin

Caplin 是一个集成在 Erigon 执行客户端中的共识客户端。它基本上是 Erigon 中的一个额外功能，允许在没有外部 CL 的情况下运行它。

- https://erigon.gitbook.io/erigon/advanced-usage/consensus-layer/caplin
- https://github.com/ledgerwatch/erigon/?tab=readme-ov-file#caplin

### LambdaClass

LambdaClass 开发了一个用 Elixir 编写的客户端。它在 EPF4 期间启动，并已发展成功能完整的实现。它仍在积极开发中，尚未用于生产环境。

- https://github.com/lambdaclass/lambda_ethereum_consensus

## 额外资源

- [ETH Docker](https://eth-docker.net/)
- [Ethernodes](https://ethernodes.org/)
- [客户端多样性 (Client Diversity)](https://clientdiversity.org/)
- [运行多数客户端，后果自负！](https://dankradfeist.de/ethereum/2022/03/24/run-the-majority-client-at-your-own-peril.html)
- [以太坊硬件资源分析](https://www.migalabs.io/blog/post/ethereum-hardware-resource-analysis-update)
