# 网络

共识客户端使用 [libp2p][libp2p-docs] 作为对等协议，[libp2p-noise][libp2p-noise] 进行加密，[discv5][discv5] 进行节点发现，[SSZ][ssz] 进行编码，以及可选的 [Snappy][snappy] 进行压缩。

对于那些希望加深对 libp2p 理解的人，学习小组第 5 周，Dapplion 的[第 19 讲](https://epf.wiki/#/eps/day19)是一个极好的资源。

## 规范

[Phase 0 -- 网络][consensus-networking] 页面详细说明了网络基础、协议以及设计理由/选择。后续的分叉也规定了各自分叉中所做的更改。

## libp2p - P2P 协议

[libp2p][libp2p-docs] 是一种对等通信协议，最初为 [IPFS](https://ipfs.io) 开发。[libp2p 与以太坊][libp2p-and-eth] 是一篇深入探讨 libp2p 历史及其在共识层中采用情况的精彩文章。它允许通过多种传输协议进行通信，如 TCP、QUIC、WebRTC 等。

![libp2p_protocols](https://epf.wiki/images/cl/cl-networking/libp2p-protocols.png)
*libp2p 中的各种协议。左：当前 右：使用 QUIC*

libp2p 协议是一个多传输协议栈。

1. **传输 (Transport)**：必须支持 TCP（传输控制协议），可以支持 [QUIC][quic]（快速 UDP 互联网连接），两者都必须允许传入和传出连接。TCP 和 QUIC 都支持 IPv4 和 IPv6，但为了更好的兼容性，要求支持 IPv4。
2. **加密与身份识别**：[libp2p-noise][libp2p-noise] 安全通道用于加密，并使用 [multiaddr][multiaddr]（通常缩写为 multiaddr）作为将多层地址编码到单一"面向未来的"路径结构中的约定。

- **Multiaddress**：Multiaddress 定义了常见传输和覆盖协议的人类可读和机器优化的编码，并允许多层寻址组合使用。<br/>例如：下面给出的地址格式是"位置 multiaddr"（IP 和端口）和身份 multiaddr（libp2p 对等节点 ID）的组合。

![multiaddr](https://epf.wiki/images/cl/cl-networking/multiaddr.png)
*Multiaddr 格式*

3. **多路复用 (Multiplexing)**：多路复用允许多个独立的通信流在单个网络连接上并发运行。两种多路复用器在 libp2p 实现中很常见：[mplex][mplex] 和 [yamux][yamux]。它们的协议 ID 分别是：`/mplex/6.7.0` 和 `/yamux/1.0.0`。客户端必须支持 mplex，可以支持 yamux，优先使用后者。
4. **消息传递**：为了在网络上传递消息，libp2p 实现了 [Gossipsub][gossipsub]（PubSub）和 [Req/Resp][req-resp]（请求/响应）。Gossipsub 使用主题 (topic)，Req/Resp 使用消息进行通信。

### **libp2p 协议栈**

| **层**                    | **协议**                                                                              | **用途**                                                  |
| ------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| 🧠 **应用层**             | `pubsub`, `gossipsub`, `ping`, 自定义协议                                              | 运行用户定义或内建的逻辑（聊天、文件传输、发布-订阅等）     |
| 🔀 **多路复用层**         | `yamux`, `mplex`                                                                      | 在单个连接上允许多个逻辑流                                 |
| 🔐 **安全层**             | `noise`, `tls`, `secio` (已弃用)                                                       | 加密和认证对等连接                                        |
| 🔌 **传输层**             | `tcp`, `websockets`, `quic` (具有多路复用和安全), `webrtc`, `webtransport`            | 处理机器之间的物理或虚拟数据传输                            |
| 🌍 **NAT/中继层**         | `relay`, `dcutr`, `autonat`, `pnet`                                                   | 启用通过 NAT/防火墙或在私有网络中的连接                     |
| 📡 **发现层**             | `mdns`, `kademlia`, `rendezvous`, `identify`                                          | 发现并了解网络上的对等节点                                 |

### 注意事项：

- **并非所有协议都是必需的** — libp2p 是模块化的，你可以只选择你需要的。
- 最小连接至少包括：`transport` + `security` + `multiplexing` + `application protocol`。
- 当 NAT 阻止直接连接时，使用 `relay` 和 `dcutr`。

libp2p 的关键特性：

1. **协议 ID：**是用于协议协商的唯一字符串标识符。其基本结构是：`/app/protocol/version`。以下是一些常见协议，都使用 protobuf 定义消息模式：

- Ping：`/ipfs/ping/1.0.0` 是一个简单的协议，用于测试已连接对等节点的连通性和性能
- Identify：`/ipfs/id/1.0.0` 允许对等节点交换彼此的信息，主要是公钥和已知网络地址。使用以下 protobuf 属性：
  | 字段                | 类型       | 用途                                                                    |
  |-------------------|-----------|-------------------------------------------------------------------------|
  | `protocolVersion` | `string`  | libp2p 协议版本（例如，`ipfs/1.0.0`）                                    |
  | `agentVersion`     | `string`  | 客户端信息（类似浏览器的 user-agent，例如，`go-ipfs/0.1.0`）              |
  | `publicKey`        | `bytes`   | 节点公钥（用于身份识别，如果使用安全通道则为可选）                         |
  | `listenAddrs`      | `bytes[]` | 对等节点正在监听的 Multiaddr                                             |
  | `observedAddr`     | `bytes`   | 对等节点看到的你的 IP 地址（有助于 NAT 检测）                              |
  | `protocols`        | `string[]`| 支持的应用协议列表（例如，`/chat/1.0.0`）                                 |
  | `signedPeerRecord` | `bytes`   | `listenAddrs` 的认证版本，用于与其他对等节点共享                           |

- Identify/push：`/ipfs/id/push/1.0.0` 与 "Identify" 相同，只是这是主动发送的，而不是响应请求。它有助于向其已连接的对等节点推送新地址。

**kad-dht**：libp2p 使用基于 [Kademlia][kademlia] 路由算法的分布式哈希表 (Distributed Hash Table, DHT) 来实现其路由功能。

2. **处理函数 (Handler Functions)：**在传入流期间被调用
3. **双向二进制流 (Bi-Directional Binary Stream)：**libp2p 协议运行的媒介

### **对等节点 (Peers)**

#### 对等节点 ID (Peer Ids)

[对等节点身份 (Peer Identity)][peer-identity] 是网络中特定对等节点的唯一引用，只要该对等节点存活就保持不变。对等节点 ID 是 [multihashes][multihash]，它们是具有以下格式的自描述值：<br/>

`<varint hash function code><varint digest size in bytes><hash function output>`

![multihash_format](https://raw.githubusercontent.com/multiformats/multihash/master/img/multihash.006.jpg)
*Multihash 格式，十六进制*

- 密钥编码在一个包含密钥类型和编码密钥的 protobuf 中。有 4 种指定的编码方法：RSA、Ed25519（必须）、Secp256k1、ECDSA。
- 对等节点 ID 在文本中有 2 种字符串表示方式：`base58btc`（以 `QM` 或 `1` 开头）和作为 multibase 编码的 CID，libp2p 正在缓慢向后一种方式迁移。

### **连接是如何建立的？**

要理解连接建立的工作原理，请阅读此[规范][libp2p-connection]。

![flowchart of setting up a connection](https://epf.wiki/images/cl/cl-networking/libp2p-connection.png)
*建立连接的流程图*

1. **发现 (Discovery)**：如何找到另一个对等节点？

- `mdns`（多播 DNS）：零配置发现同一本地网络上的对等节点，非常简单。发送查询所有对等节点，接收响应和其他对等节点信息到本地数据库。
- `rendezvous`：对等节点在共享的公共节点或服务器（集合点, rendezvous point）上注册自己，其他节点查询同一点以获取对等节点信息
- `kademlia` (DHT)：用于全球发现的分布式哈希表。对等节点使用对等节点 ID 查询 DHT 以获取其最新的 multiaddr。
- `identify`：允许对等节点在连接后交换元数据（地址、支持的协议、版本等）的协议

结果：我们现在有一个可通过 Peer ID 识别并通过 multiaddr（IP + 端口 + 协议栈）可达的对等节点列表

2. **传输 (Transport)**：如何连接到对等节点？

- `TCP`：最基本的传输方式，可靠但可能被 NAT/防火墙阻止
- `WebSockets`：基于 HTTP 的 TCP，对 NAT/防火墙友好
- `QUIC`：基于 UDP，连接建立更快，原生支持多路复用和加密（TLS 1.3）
- `WebRTC`：使两个私有节点（例如，两个浏览器）能够建立直接连接
- `WebTransport`：建立到服务器的流多路复用和双向连接，运行在 HTTP/3 连接之上，并使用 HTTP/2 作为后备

如果对等节点在 NAT 后面（直接连接失败时）：

- `relay`：类似于 TURN，通过另一个对等节点路由
- `dcutr`（通过中继的直接连接升级, Direct Connection Upgrade Through Relay）：用于尝试用直接连接替换中继连接，使用[打洞 (hole punching)][hole-punching]技术。它涉及双方同时进行拨号尝试。

3. **加密 (Encryption)**：如何使连接私有且经过认证？

- `noise`：构建安全协议的框架，快速，许多实现的默认选择，Noise XX 握手用于相互认证。
- `tls`（传输层安全, Transport Layer Security）：强大的安全保障，使用对等节点密钥进行相互认证
- `secio`：由于复杂性和比 Noise/TLS 低的安全保障而弃用。

4. **多路复用 (Multiplexing)**：如何在同一个连接上打开多个逻辑流？

- `yamux`：简单、快速，目前是许多实现中的默认选择。
- `mplex`：轻量级且较旧

5. **应用 (Application)**：在上述设置之上运行应用协议

- `ping`：基本的活跃性检查，测量往返时间
- `pubsub`、`gossipsub`、`episub`：用于广播消息
- 实现定义的自定义协议

### GossipSub 提供了什么优化？

**方法 1：** 维护一个全连接网格（所有对等节点彼此 1:1 连接），这种方式扩展性差（O(n^2)）。为什么扩展性差？每个节点可能从其他 (n-1) 个节点接收到相同的消息，因此浪费了大量带宽。如果消息是区块数据，那么浪费的带宽呈指数级增长。

**方法 2：** 使用 Pubsub（发布-订阅模型）消息模式，发送者（发布者）不直接将消息发送给接收者（订阅者）。相反，消息发布到一个公共通道（或主题），订阅者从该通道接收消息，而无需与发布者直接交互。节点为某个主题与其他特定数量的节点建立网格连接，再与其他节点建立不同的网格连接。因此，实现了更高效的消息传递。

![gossipsub_optimization](https://epf.wiki/images/cl/cl-networking/gossipsub_optimization.png)
*Gossipsub 优化*

###### **Gossipsub：TODO**

###### **Req/Resp：TODO**

###### **QUIC：TODO**

## libp2p-noise - 加密

[Noise 框架][noise-framework] 本身不是一个协议，而是一个设计密钥交换协议的框架。[规范][noise-specification] 是一个很好的起点。

有许多[模式][noise-patterns]描述了密钥交换过程。共识客户端中使用的模式是 [`XX`][noise-xx]（传输-传输），意味着发起者和响应者在密钥交换的初始阶段都传输其公钥。

## ENR (Ethereum Node Records，以太坊节点记录)

[以太坊节点记录 (Ethereum Node Records, ENR)][ENR] 提供了一种结构化、灵活的方式来存储和共享以太坊对等网络中节点身份和连接性详情。它是一种面向未来的格式，允许在新对等节点之间更容易地交换识别信息，并且是以太坊节点的首选[网络地址格式][network-add-format]。

其核心组件包括：

1. **签名 (Signature)**：每条记录使用身份方案（例如，secp256k1）签名以确保真实性。
2. **序列号 (Sequence Number, seq)**：一个 64 位无符号整数，每当记录更新时递增，使对等节点能够确定最新版本。
3. **键/值对 (Key/Value Pairs)**：该记录以键值对的形式保存各种连接性详情。

在其文本形式中，ENR 的 RLP 编码以 base64 编码，可以作为字符串在客户端之间共享，例如：
`
enr:-Jq4QOXd31zNJBTBAT0ZZIRWH4z_NmRhnmAFfwNan0zr_-IUUAsOTbU_Lhzh4BSq8UknFGvr1rXQUYK0P-_ZUVenXkABhGV0aDKQaGGQMVAAEBv__________4JpZIJ2NIJpcIRBbZouiXNlY3AyNTZrMaEDxEArICqVUZNxhUxBYHZjzsm4KxqraeSION3yYorLZSuDdWRwgiMp
`
## discv5

发现版本 5 [(discv5)][discv5]（协议版本 v5.1）是一种基于 UDP 的节点发现协议。在协议层面，它是一个受 Kademlia 启发的 DHT，存储和中继经过签名的以太坊节点记录 (ENR)，而不是任意的键值对。每个节点通过节点 ID 之间的 XOR 距离在 k-bucket 的路由表中组织其他节点，使其能够跟踪地址空间中附近的对等节点并逐步改善其对网络的视图。

discv5 中的对等节点发现通过迭代查找进行。节点从它已知的最近对等节点开始，发送 `FINDNODE` 查询，接收 `NODES` 响应，并继续查询更接近的结果，直到查找收敛到目标最近的、可达的节点。这种相同的查找机制支撑着网络采样、服务发现和 ENR 解析。与负责维护对等连接和承载协议流量的 libp2p 不同，discv5 专注于维护一个可搜索的活跃节点及其通告能力的发现索引。

其三个核心功能是：

- 通过遍历发现 DHT 并随时间刷新本地路由表来采样所有活跃参与者的集合
- 通过主题广告 (topic advertisement) 和围绕主题哈希的查找来搜索提供特定服务的参与者
- 通过检索已知节点 ID 的最新 ENR 并比较 ENR 序列号来进行节点记录的权威解析

在以太坊共识客户端中，discv5 通常与 libp2p 一起运行，后者处理对等连接和协议流量。

![discv5](https://epf.wiki/images/cl/cl-networking/discv5.png)
*discv5*

### 查找流程

discv5 查找是在节点 ID 空间上的迭代搜索，而不是向整个网络的广播。发起者反复向已经接近目标的节点询问更接近的节点，这赋予了 Kademlia 对数级的扩展特性，并让发现表在正常操作中作为副作用不断改善。

1. 节点从其本地路由表中选择已知的、最接近目标节点 ID 的对等节点。
2. 它向这些对等节点的一小部分并发集合发送 `FINDNODE` 请求。
3. 响应的对等节点返回 `NODES` 消息，其中包含接近查询距离的 ENR。
4. 发起者将返回的 ENR 合并到其本地视图中，按到目标的 XOR 距离排序，并继续查询更近的候选节点。
5. 一旦节点已经查询了其所了解到的最近的、可达的候选节点，查找就收敛了。

这种相同的模式被复用于多个任务。随机目标让节点可以遍历 DHT 并采样活跃参与者。已知的节点 ID 让调用者可以解析该节点的最新 ENR。基于主题的发现复用了围绕主题哈希的 Kademlia 查找，使节点可以找到通告特定服务的对等节点。

### discv5 与 libp2p

在共识客户端中，discv5 和 libp2p 解决相邻但不同的问题。discv5 是发现平面：它维护一个签名节点记录的可搜索索引，验证活跃性，并帮助节点了解谁存在以及如何到达它们。libp2p 是传输和消息平面：一旦知道了有用的对等节点，客户端就通过 libp2p 拨号连接它们，然后通过 gossip 和请求/响应协议交换信标链流量。

这种分离很重要，因为发现和传输有不同的约束。discv5 使用 UDP 和针对与大量远程节点进行许多短交互而优化的加密握手。相比之下，libp2p 维护更长寿命的认证连接，并承载实际的应用协议，如 gossipsub、ping 和 req/resp。在实践中，discv5 回答"接下来我应该尝试哪些对等节点？"，而 libp2p 回答"如何与它们维护会话并交换协议消息？"

### 安全性与握手

discv5 不会将发现流量作为纯未认证的 UDP 数据包发送。普通数据包是加密和认证的，当节点无法从某个端点 `decrypt` 消息时，它回复一个 `WHOAREYOU` 挑战，而不是盲目返回发现数据。这迫使发起者证明对其节点身份的控制，并在接收者接受诸如 `FINDNODE` 的请求之前建立新的会话密钥。

在高层次上，交换过程如下：

1. 节点 A 向节点 B 发送一个普通请求，例如 `FINDNODE`。
2. 如果节点 B 没有该端点的有效会话，它回复 `WHOAREYOU`，携带一个 nonce 和它当前知道的关于 A 的 ENR 序列号。
3. 节点 A 将请求作为握手数据包重新发送，包括身份签名、临时公钥，以及当 B 需要更新的副本时的 ENR。
4. 双方派生会话密钥，认证数据包，并继续使用加密的请求/响应消息。

这个握手同时服务于多个目的。它降低了放大攻击风险，因为未知的发送者首先只收到一个小的挑战数据包。它将会话状态绑定到特定的节点 ID 和 UDP 端点，使伪造和跨端点重放更加困难。它还为节点提供了 ENR 同步的内建路径：当接收者通告一个较旧的 `enr-seq` 时，发送者可以在握手期间附加其更新的记录，或在稍后通过 `PING`/`PONG` 和显式的 ENR 检索来刷新它。

即便有了这种密码学机制，UDP 仍然是一个刻意的选择。发现流量由许多与大量远程节点的短交互、频繁的表刷新和重复的活跃性检查组成。使用 UDP 使这些交换保持轻量级，并避免了仅为询问几个发现问题而建立完整传输会话的连接管理开销。代价是数据包可能丢失或乱序到达，因此 discv5 的设计围绕短超时、通过新查找的重试，以及一个持续刷新而非假设在任何时刻都完美的路由表。

## SSZ - 编码

[简单序列化 (Simple Serialize, SSZ)][ssz] 取代了执行层上使用的 [RLP][rlp] 序列化，在共识层中除对等发现协议以外的所有地方使用。SSZ 被设计为确定性的，并且能够高效地进行 Merkle 化。SSZ 可以被认为有两个组件：一个序列化方案和一个设计为与序列化数据结构高效配合的 Merkle 化方案。

## Snappy - 压缩

[Snappy][snappy] 是 Google 工程师于 2011 年创建的压缩方案。其主要设计考虑优先考虑压缩/解压速度，同时仍具有合理的压缩比。

## 相关研发

- [EIP-7594][peerdas-eip] - 对等数据可用性采样 (Peer Data Availability Sampling, PeerDAS)

  一种网络协议，允许信标节点执行数据可用性采样 (Data Availability Sampling, DAS)，以确保 blob 数据已可用，而只需下载数据的一个子集。

  - [共识规范][peerdas-specs]
  - [ETH Research][peerdas-ethresearch]

## 资源

- [ENR rust docs][enr-rust-docs]
- [Eth1+Eth2 客户端关系][eth1+2-client]
- Libp2p, ["docs"][libp2p-docs] 和 ["specs"][libp2p-specs]
- 技术报告, ["Gossipsub-v1.1 Evaluation Report"][gossipsub-report]
- [Libp2p 资源][PLN-launchpad]
- [Libp2p 教程][libp2p-tutorial]
- [打洞 (Hole Punching)][hole-punching]

[consensus-networking]: https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/p2p-interface.md
[libp2p-and-eth]: https://blog.libp2p.io/libp2p-and-ethereum/
[libp2p-noise]: https://github.com/libp2p/specs/tree/master/noise
[libp2p-docs]: https://docs.libp2p.io/
[libp2p-specs]: https://github.com/libp2p/specs
[noise-framework]: https://noiseprotocol.org/
[noise-patterns]: https://noiseexplorer.com/patterns/
[noise-specification]: https://noiseprotocol.org/noise.html
[noise-xx]: https://noiseexplorer.com/patterns/XX/
[discv5]: https://github.com/ethereum/devp2p/blob/master/discv5/discv5.md
[peerdas-eip]: https://github.com/ethereum/EIPs/pull/8105
[peerdas-ethresearch]: https://ethresear.ch/t/peerdas-a-simpler-das-approach-using-battle-tested-p2p-components/16541
[peerdas-specs]: https://github.com/ethereum/consensus-specs/pull/3574
[rlp]: https://ethereum.org/developers/docs/data-structures-and-encoding/rlp
[snappy]: https://en.wikipedia.org/wiki/Snappy_(compression)
[ssz]: https://ethereum.org/developers/docs/data-structures-and-encoding/ssz
[blog]: https://medium.com/coinmonks/dissecting-the-ethereum-networking-stack-node-discovery-4b3f7895f83f
[enr-rust-docs]: https://docs.rs/enr/latest/enr
[eth1+2-client]: https://ethresear.ch/t/eth1-eth2-client-relationship/7248
[gossipsub-report]: https://research.protocol.ai/publications/gossipsub-v1.1-evaluation-report/vyzovitis2020.pdf
[ENR]: https://eips.ethereum.org/EIPS/eip-778
[network-add-format]: https://dean.eigenmann.me/blog/2020/01/21/network-addresses-in-ethereum/
[quic]: https://datatracker.ietf.org/doc/rfc9000/
[yamux]: https://github.com/libp2p/specs/blob/master/yamux/README.md
[mplex]: https://github.com/libp2p/specs/tree/master/mplex
[gossipsub]: https://github.com/libp2p/specs/tree/master/pubsub/gossipsub
[req-resp]: https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/p2p-interface.md#the-reqresp-domain
[multiaddr]: https://github.com/libp2p/specs/blob/master/addressing/README.md
[PLN-launchpad]: https://pl-launchpad.io/curriculum/libp2p/objectives/
[kademlia]: https://docs.ipfs.tech/concepts/dht/#kademlia
[libp2p-tutorial]: https://proto.school/introduction-to-libp2p
[multihash]: https://github.com/multiformats/multihash?tab=readme-ov-file
[multistream-select]: https://github.com/multiformats/multistream-select
[libp2p-connection]: https://github.com/libp2p/specs/blob/master/connections/README.md
[hole-punching]: https://github.com/libp2p/specs/blob/master/connections/hole-punching.md
[peer-identity]: https://github.com/libp2p/specs/blob/master/peer-ids/peer-ids.md
[hole-punching]: https://blog.ipfs.tech/2022-01-20-libp2p-hole-punching/
