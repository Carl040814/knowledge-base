# DevP2P

本节将涵盖执行层 (Execution Layer, EL) 所使用的网络协议 (Networking Protocol)。
首先，如[网络部分 (Networking Section)](../dev/cs-resources.md?id=networking)中所指出的，关注传输层 (Transport Layer)，DevP2P 使用的两种协议是 TCP（传输控制协议 (Transmission Control Protocol)）和 UDP（用户数据报协议 (User Datagram Protocol)）。
这两种协议都用于在互联网上传输数据，但它们具有不同的特性。正如 Tanenbaum (2021) 所指出的，TCP 是一种面向连接的协议 (Connection-oriented Protocol)，这意味着它在发送数据之前先在发送方和接收方之间建立连接。
它是可靠的，因为它确保数据按照正确的顺序且无错误地送达。UDP 是一种无连接协议 (Connectionless Protocol)，这意味着它在发送数据之前不建立连接。
它比 TCP 更快，因为它在发送数据之前不必建立连接，但它不太可靠，因为它不确保数据按正确的顺序或无错误地送达。

![TCP/UDP 比较](https://epf.wiki/images/el-architecture/tcpudp.png)


## 执行层网络规范 (EL's networking specs)

作为一个点对点 (Peer-to-Peer, P2P) 网络，以太坊暗含了一系列规则，以实现其参与节点之间的通信。本节将介绍这些规则是什么以及它们如何在执行层 (EL) 中实现。
考虑到每个以太坊节点都构建在两个不同的组件上：执行客户端 (Execution Client) 和共识客户端 (Consensus Client)，它们每一个都有自己的点对点网络，并有其独特用途。
执行客户端负责传播/广播交易 (Gossiping Transactions)，而共识客户端负责传播/广播区块 (Gossiping Blocks)。
> 存在不同的 CL/EL P2P 网络及其底层技术是有历史原因的。以太坊最初是构建在 devp2p 作为其自定义网络技术栈基础上的。当信标链 (Beacon Chain) 创建时，libp2p 已经具备生产环境可用性，并被其采纳。
牢记这一点，执行层 (EL) 网络范围涵盖了两个并行工作的不同技术栈：节点发现技术栈 (Discovery Stack) 以及信息传输技术栈 (Transport Stack) 本身。
发现栈负责寻找节点对等方 (Node Peers)，而传输栈负责在它们之间发送和接收消息。
考虑到计算机网络背景，我们可以推断出发现栈依赖于 UDP 协议，而信息交换栈依赖于 TCP 协议。
这背后的原因在于，信息交换需要节点之间建立可靠的连接，
以便它们在发送数据之前都能够确认连接，并拥有一种确保数据按正确顺序且无错误送达（或者至少拥有一种检测并纠正错误的方法）的手段，
而发现过程不需要可靠的连接，因为仅让其他节点知道该节点可进行通信就足够了。

### 发现协议 (Discv protocol (Discovery))

节点在网络中如何寻找彼此的过程，始于[规范中列出的硬编码引导节点 (Hard-coded Bootnodes)](https://github.com/ethereum/go-ethereum/blob/master/params/bootnodes.go)。
引导节点是网络（包括主网和测试网）中所有其他节点都知道的节点，它们用于引导（Bootstrap）发现对等节点的过程。
使用类似 Kademlia 的分布式哈希表 (Distributed Hash Table, DHT) 算法，节点能够通过引用列有引导节点的路由表 (Routing Table) 在网络中找到彼此。
Kademlia 的简要原理是，它是一个点对点协议，它使节点能够通过使用分布式哈希表在网络中寻找彼此，正如 Leffew 在其文章 (2019) 中所提到的。

也就是说，连接过程始于 PING-PONG（乒乓）交互，其中新节点向引导节点发送 PING 消息，引导节点响应以经过哈希的 PONG 消息。
如果两个消息匹配，则新节点能够与引导节点建立绑定 (Bond)。此外，新节点向引导节点发送 FIND-NEIGHBOURS（寻找邻居）请求，以便它可以接收到能够连接的邻居列表，
然后它可以用它们重复 PING-PONG 交互并与它们也建立绑定。

![节点发现 (Peer Discovery)](https://epf.wiki/images/el-architecture/peer-discovery.png)

#### 有线子协议 (Wire protocol)

PING/PONG 交互更广为人知的名称是有线子协议 (Wire Subprotocol)，它包括以下规范：

**PING 数据包结构 (PING packet structure)**
```
version = 4
from = [sender-ip, sender-udp-port, sender-tcp-port]
to = [recipient-ip, recipient-udp-port, 0]
packet-data = [version, from, to, expiration, enr-seq ...]
```

**PONG 数据包结构 (PONG packet structure)**
```
packet-data = [to, ping-hash, expiration, enr-seq, ...]
```

数据包数据 (Packet-data) 封装在 1280 字节的 UDP 数据报 (Datagram) 中，并带有报头 (Header)：
```
packet-header = hash || signature || packet-type
hash = keccak256(signature || packet-type || packet-data)
signature = sign(packet-type || packet-data)
packet = packet-header || packet-data
```

**FindNode 数据包结构 (FindNode packet structure)**（上面称为 FIND-NEIGHBOURS）
```
packet-data = [target, expiration, ...]
```
其中 target 是一个 64 字节的 secp256k1 节点公钥。

**Neighbors 数据包结构 (Neighbors packet structure)**
```
packet-data = [expiration, neighbours, ...]
neighbours = [ip, udp-port, tcp-port, node-id, ...]
```
其中 neighbours 是能够与新节点连接的 16 个节点的列表。

**ENR Request 数据包结构 (ENR Request packet structure)**
```
packet-data = [expiration]
```

**ENR Response 数据包结构 (ENR Response packet structure)**
```
packet-data = [request-hash, ENR]
```
其中 ENR 是以太坊节点记录 (Ethereum Node Record)，这是节点连通性的标准格式。具体解释如下。

---

这个类似 Kademlia 的协议包括路由表 (Routing Table)，路由表保存着关于邻近其他节点的信息，由 *k 桶 (k-buckets)* 组成（其中 *k* 是桶中的节点数，目前定义为 16）。
值得一提的是，所有表项都按照 *最近看到/最久未看到 (Last seen/least-recently seen)* 进行排序，最久未看到的排在头部，最近看到的排在尾部。
如果其中一个实体在 12 小时内没有做出响应，它就会从表中被移除，而下一个遇到的节点将被添加到列表尾部。


#### 发现协议版本 (Discovery Protocols (Discv4 & Discv5))

目前，大多数执行客户端都已采用 [Discv5 协议](https://github.com/ethereum/devp2p/blob/master/discv5/discv5.md)进行发现过程，而一些客户端仍处于从 [Discv4](https://github.com/ethereum/devp2p/blob/master/discv4.md) 过渡的阶段。下表根据截至 2025 年 5 月的 Discv5 支持状态对执行客户端进行了分类。

| **类别** | **执行客户端** |
|---------------------|-------------------------------------------|
| **支持 Discv5** | [Geth](https://github.com/search?q=repo%3Aethereum%2Fgo-ethereum%20discv5&type=code), [Nethermind](https://github.com/search?q=repo%3ANethermindEth%2Fnethermind+discv5&type=issues), [Reth](https://github.com/search?q=repo%3Aparadigmxyz%2Freth%20discv5&type=code) |
| **等待迁移** | [Besu](https://github.com/search?q=repo%3Ahyperledger%2Fbesu+discv5&type=issues), [Ethereumjs](https://github.com/ethereumjs/ethereumjs-monorepo/tree/master/packages/devp2p), [Erigon](https://github.com/search?q=repo%3Aerigontech%2Ferigon+discv4&type=code) |

##### Discv4

一个结构化的分布式系统，允许以太坊节点在没有中央协调的情况下发现对等节点 (Peers)。

- **节点身份 (Node Identities)**  
  - 每个节点由一个 secp256k1 密钥对标识。  
  - 公钥作为节点的唯一标识符（节点 ID (Node ID)）。  
  - 节点之间的距离是使用哈希公钥的 XOR（异或）计算出来的。  

- **节点记录 (Node Records, ENR)**  
  - 节点使用以太坊节点记录 (Ethereum Node Records, ENR) 存储和共享连接细节。  
  - "v4" 身份方案用于验证节点的真实性。  
  - 对等节点可以通过 **ENRRequest** 数据包请求节点的最新 ENR。  

- **[Kademlia 表 (Kademlia Table)](https://en.wikipedia.org/wiki/Kademlia)**  
  - 节点维护一个具有 256 个 **[k 桶 (k-buckets)](https://en.wikipedia.org/wiki/Kademlia#Fixed-size_routing_tables)** 的**路由表**（每个桶最多持有 16 个条目）。  
  - 一个桶存储特定距离范围内的节点（例如，`[2^i, 2^(i+1))`）。  
  - 节点按最后一次看到的时间排序，确保在表满时替换陈旧的节点。  

- **端点验证 (Endpoint Verification)（参与证明 (Proof-of-Participation)）**  
  - 通过在响应查询之前验证节点，来防止放大攻击 (Amplification Attacks)。  
  - 如果节点对最近的 **Ping** 请求发送了有效的 **Pong** 响应，则认为该节点已验证。  

- **递归查找算法 (Recursive Lookup Algorithm)**  
  - 寻找距离目标最近的 `k` 个（通常为 16 个）节点。  
  - 搜索始于向已知最近节点的一个小型选择子集（`α`，通常设为 3）发送查询。  
  - 查找是**迭代的 (Iterative)**，在前面的步骤中查询新发现的节点，直到找不到更近的节点。  

- **有线协议与消息类型 (Wire Protocol & Message Types)**  
  - 消息通过 **UDP** 发送。  
  - 每个数据包包含一个报头（`hash`、`signature`、`packet-type`），后跟编码的数据。  
  - 核心消息类型：  
    - **Ping (0x01):** 验证节点可用性。  
    - **Pong (0x02):** 对 Ping 的响应，证明可达性。  
    - **FindNode (0x03):** 请求靠近目标 ID 的节点。  
    - **Neighbors (0x04):** 回复 FindNode，提供已知最近的对等节点。  
    - **ENRRequest (0x05):** 请求节点的最新 ENR。  
    - **ENRResponse (0x06):** 响应请求以提供 ENR。  


##### Discv5

Discv5 是以太坊改进的去中心化对等节点发现协议，在 Discv4 的基础上构建，具有增强的服务发现和服务安全机制。与其前身一样，Discv5 使节点能够以去中心化的方式定位和连接对等节点，而不依赖于中心化目录。然而，它引入了加密通信、服务发现和自适应路由。

受 Kademlia DHT 启发，discv5 的不同之处在于它仅存储签名的节点记录 (ENR)，而不是任意的键值对。这确保了对等节点发现中的真实性和完整性，同时保持了协议扩展的灵活性。

- **以太坊节点记录 (Ethereum Node Records, ENR)**
  - 每个节点维护一个**以太坊节点记录 (Ethereum Node Record, ENR)**，存储**连接细节、密码学密钥和元数据**。
  - ENR 是签名的、自包含的，并且动态更新。
  - 对等节点可以使用 **ENRRequest 数据包**请求最新的 ENR。

- **加密的有线协议 (Encrypted Wire Protocol)**
  - 使用 **[AES-GCM 加密](https://en.wikipedia.org/wiki/AES-GCM-SIV)** 以确保机密性和真实性。
  - 通过 ECDH ([椭圆曲线迪菲-赫尔曼 (Elliptic Curve Diffie-Hellman)](https://en.wikipedia.org/wiki/Elliptic-curve_Diffie%E2%80%93Hellman)) 建立**会话密钥 (Session Keys)**。
  - 实施 **WHOAREYOU 挑战-响应机制 (WHOAREYOU Challenge-Response Mechanism)** 以防止欺骗。

- **基于 Kademlia 的路由与节点表 (Kademlia-Based Routing & Node Table)**
  - 节点维护一个**路由表 (k 桶)**，其中的对等节点按 XOR 距离进行排序。
  - 查找过程递归地查询已知最近的节点。
  - 该协议支持**自适应路由与自愈 (Adaptive Routing and Self-healing)**。

- **递归节点查找与对等节点发现 (Recursive Node Lookup & Peer Discovery)**
  - 节点通过**迭代的基于 Kademlia 的查找**来寻找对等节点。
  - 使用并行查询来**提高对对抗方的弹性**。
  - 引导节点促进新节点加入。

- **主题广告与服务发现 (Topic Advertisement & Service Discovery)**
  - 节点通过**主题广告 (Topic Advertisements)** 宣传服务。
  - 寻找提供服务的节点的行为在主题半径内使用**Kademlia 查找**。
  - 自适应**半径估计**确保了高效的搜索。

- **有线协议与消息类型 (Wire Protocol & Message Types)**
  | **消息 (Message)** | **功能 (Function)** |
  |---------------|-------------|
  | **Ping** (0x01) | 检查节点是否存活。 |
  | **Pong** (0x02) | 对 Ping 的响应，确认可达性。 |
  | **FindNode** (0x03) | 请求靠近目标 ID 的对等节点。 |
  | **Nodes** (0x04) | 响应 FindNode，返回已知对等节点。 |
  | **ENRRequest** (0x05) | 请求节点的最新 ENR。 |
  | **ENRResponse** (0x06) | 提供请求的 ENR。 |
  | **WhoAreYou** (0x07) | 身份验证挑战。 |
  | **Handshake** (0x08) | 建立加密会话。 |
  | **TalkReq / TalkResp** (0x09/0x0A) | 允许自定义应用程序协议。 |


##### 对比：Discv4 与 Discv5 (Comparison: Discv4 vs. Discv5)
| 特性 (Feature) | Discv4 | Discv5 |
|-------------------------|--------|--------|
| **节点记录 (Node Records)** | 基础 ENR | 带有元数据的可扩展 ENR |
| **安全性 (Security)** | 明文 | AES-GCM 加密 |
| **握手 (Handshake)** | 无 | 安全会话建立 |
| **服务发现 (Service Discovery)** | 受限 | 基于主题的查找 |
| **可扩展性 (Extensibility)** | 静态 | 支持多种身份方案 |
| **时钟依赖 (Clock Dependence)** | 必须 | 已消除 |
| **可扩展性/缩放性 (Scalability)** | 中等 | 针对大型网络进行了优化 |

### ENR：以太坊节点记录 (ENR: Ethereum Node Records)

ENR 是 P2P 连通性的标准格式，最初在 [EIP-778](https://eips.ethereum.org/EIPS/eip-778) 中被提出。
节点记录包含节点的网络端点（例如 IP 地址和端口），以及节点的公钥和记录的序列号 (Sequence Number)。

记录内容结构如下：

| 键 (Key) | 值 (Value) |
| --- |-------------------------------------------|
| id | id 方案，例如 "v4" |
| secp256k1 | 压缩的公钥，33 字节 |
| ip | IPv4 地址，4 字节 |
| tcp | TCP 端口，大端整数 |
| udp | UDP 端口，大端整数 |
| ip6 | IPv6 地址，16 字节 |
| tcp6 | IPv6 特定的 TCP 端口，大端整数 |
| udp6 | IPv6 特定的 UDP 端口，大端整数 |

除了 `id` 字段是必需的外，所有其他字段都是可选的。如果没有提供 `tcp6`/`udp6` 端口，则 `tcp`/`udp` 端口将同时用于 IPv4 和 IPv6。

节点记录由 `signature`（记录内容的密码学签名）和 `seq` 字段（记录的序列号，一个 64 位无符号整数）组成。
#### 编码 (Encoding)

该记录被编码为最大大小为 300 字节的 `[signature, seq, k, v,...]` RLP 列表。
签名的记录编码如下：
```
content = [seq, k, v, ...]
signature = sign(content)
record = [signature, seq, k, v, ...]
```
除了 RLP 编码之外，还有一种文本表示形式，即 RLP 编码的 Base64 编码。它以 `enr:` 为前缀。
例如 `enr:-IS4QHCYrYZbAKWCBRlAy5zzaDZXJBGkcnh4MHcBFZntXNFrdvJjX04jRzjzCBOonrkTfj499SZuOh8R33Ls8RRcy5wBgmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQPKY0yuDUmstAHYpMa2_oxVtw0RW_QAdpzBQA8yWM0xOIN1ZHCCdl8`，它包含了本地回环地址 `127.0.0.1` 和 UDP 端口 30303。节点 ID 为 `a448f24c6d18e575453db13171562b71999873db5b286df957af199ec94617f7`。

尽管 ENR 是 P2P 连通性的标准格式，但在以太坊网络中它并不是强制使用的。节点可以使用任何其他格式来交换有关其连通性的信息。
以太坊节点能够理解另外两种格式：multiaddr 和 enode。

* multiaddr 是最初使用的格式。例如，对于监听 TCP 端口 30303 且节点 ID 为 `a448f24c6d18e575453db13171562b71999873db5b286df957af199ec94617f7` 的回环 IP 节点，其 multiaddr 是 `/ip4/127.0.0.1/tcp/30303/a448f24c6d18e575453db13171562b71999873db5b286df957af199ec94617f7`。
* enode 是一种更具可读性的格式。例如，同一节点的 enode 是 `enode://a448f24c6d18e575453db13171562b71999873db5b286df957af199ec94617f7@127.0.0.1:30303?discport=30301`。这是一种类似于 URL 的格式，描述了在 @ 符号前编码的节点 ID、IP 地址、TCP 端口以及指定为 "discport" 的 UDP 端口。

### RLPx 协议 (Transport) (RLPx protocol (Transport))

到目前为止，本文仅指代了发现协议，但安全信息交换过程又是怎样的呢？RLPx 是一种基于 TCP 的传输协议，能够在执行层 (EL) 中实现安全的点对点通信。它处理以太坊节点之间的连接建立和消息交换。其名称源于 [RLP 序列化格式 (RLP Serialization Format)](../EL/RLP.md)。

在深入研究协议之前，这里有一个摘要以及一张图示：

* 通过加密身份验证实现安全连接
* 会话建立
* 消息成帧 (Message Framing) 和信息交换


![RLPx 通信图示](https://epf.wiki/images/el-architecture/rlpx-communication.png)

#### 安全连接建立 (Secure connection establishment)

一旦节点被发现，RLPx 通过基于密码学的握手互相进行身份验证，从而在它们之间建立安全连接。
该过程始于发起身份验证，其中发起节点使用 secp256k1 椭圆曲线生成一个临时密钥对 (Ephemeral Key Pair)。此临时密钥在为会话建立完全前向安全性 (Perfect Forward Secrecy) 方面起着至关重要作用。然后发起方将包含临时公钥和随机数 (Nonce) 的身份验证消息发送给接收方，接收方接受连接，使用在通信期间交换的公钥解密并验证 auth（身份验证）消息。

接收方向发起方发送确认消息 (Acknowledge Message)，然后发送第一个加密帧，该帧包含一个 [Hello 消息](https://github.com/ethereum/devp2p/blob/master/rlpx.md#hello-0x00)（包括端口、它们的 ID 以及它们的客户端 ID，和协议信息）。一旦节点互相完成了身份验证，它们就可以开始通信。

#### 会话与多路复用 (Session and multiplexing)

一旦验证通过，它们就可以首先通过以下过程创建安全会话来进行交互：
- RLPx 使用**椭圆曲线集成加密方案 (Elliptic Curve Integrated Encryption Scheme, ECIES)** 来实现安全的**握手和会话建立**。
- 该密码系统由以下部分组成：
  - **椭圆曲线 (Elliptic Curve)**: secp256k1
  - **密钥派生函数 (Key Derivation Function, KDF)**: NIST SP 800-56 Concatenation KDF
  - **消息认证码 (Message Authentication Code, MAC)**: HMAC-SHA-256
  - **加密算法 (Encryption Algorithm)**: AES-128-CTR

##### 加密过程 (Encryption Process)

1. **发起方生成随机临时密钥对**。
2. 使用**椭圆曲线迪菲-赫尔曼 (Elliptic Curve Diffie-Hellman, ECDH)** 计算**共享密钥 (Shared Secret)**。
3. 从**共享密钥**派生出加密密钥 (`kE`) 和 MAC 密钥 (`kM`)。
4. 使用 **AES-128-CTR** 对消息进行加密。
5. 对加密消息计算一个 **MAC** 以确保完整性。
6. 发送加密的有效载荷 (Payload)。

##### 解密过程 (Decryption Process)

1. **接收方提取发送方的临时公钥**。
2. 使用 **ECDH** 计算**共享密钥**。
3. 派生出 `kE` 和 `kM`，然后验证 **MAC**。
4. 使用 **AES-128-CTR** **解密**消息。


##### 节点身份 (Node Identity)

- **以太坊节点维护一个持久的 secp256k1 密钥对**以表示身份。
- **公钥**作为**节点 ID (Node ID)**。
- **私钥安全存储**，且在不同会话之间保持不变。

##### 生成的密钥/秘密 (Generated Secrets)

| 密钥 (Secret) | 描述 (Description) |
|--------|------------|
| `static-shared-secret` | `ECDH(node-private-key, remote-node-pubkey)` |
| `ephemeral-key` | `ECDH(ephemeral-private-key, remote-ephemeral-pubkey)` |
| `shared-secret` | `keccak256(ephemeral-key || keccak256(nonce || initiator-nonce))` |
| `aes-secret` | `keccak256(ephemeral-key || shared-secret)` |
| `mac-secret` | `keccak256(ephemeral-key || aes-secret)` |

##### 静态共享密钥 与 临时密钥 (Static-Shared-Secret vs. Ephemeral-Key)

###### 静态共享密钥 (Static-Shared-Secret)

- 使用椭圆曲线迪菲-赫尔曼 (ECDH) 在节点的长期（静态）私钥与对等方的长期公钥之间派生而来。
- 与同一对等方的多个会话中保持不变。

如果攻击者破解了节点的私钥，则过去和未来与该对等方的通信都可以被解密，这使得它容易受到长期密钥泄露的攻击。

###### 临时密钥 (Ephemeral-Key)（前向安全性）

- 为每次握手临时生成的密钥对，用于派生全新的会话密钥。
- 使用在握手期间交换的临时私钥之间的 ECDH 计算得出。

由于临时密钥在会话结束后即被丢弃，即使攻击者稍后获取了节点的长期私钥，过去的通信仍然安全。这种特性被称为前向安全性 (Forward Secrecy)。


##### 消息成帧 (Message Framing)

- **帧 (Frames) 封装了加密消息**，以实现高效且安全的通信。
- **多路复用 (Multiplexing)** 允许在单个 RLPx 连接上运行多个协议。

##### 帧结构 (Frame Structure)

| 字段 (Field) | 描述 (Description) |
|-------|------------|
| `header-ciphertext` | AES 加密的**帧头 (Header)**，包含帧元数据。 |
| `header-mac` | 对帧头计算的 **MAC**，用于完整性验证。 |
| `frame-ciphertext` | AES 加密的**消息数据 (Message Data)**。 |
| `frame-mac` | 对加密消息数据计算的 **MAC**。 |

##### MAC 计算 (MAC Calculation)

- 使用**两个 keccak256 MAC 状态**（一个用于**输入 (Ingress)**，一个用于**输出 (Egress)**）。
- MAC 状态随着帧的发送或接收而更新。
- 确保**消息完整性**并防止**篡改**。


##### 能力消息传递 (Capability Messaging)

- **能力 (Capabilities)** 定义了在给定连接上支持的协议。
- **多路复用 (Multiplexing)** 实现了多个能力的并发使用。

##### 消息结构 (Message Structure)

| 字段 (Field) | 描述 (Description) |
|-------|------------|
| `msg-id` | 消息类型的唯一标识符。 |
| `msg-data` | **RLP 编码**的消息有效载荷 (Payload)。 |
| `frame-size` | `msg-data` 的**压缩大小**。 |


#### P2P 能力消息 (P2P Capability Messages)

- **"p2p" 能力**是**强制性的**，用于初始协商。

#### 核心消息 (Core Messages)

| 消息 (Message) | ID | 功能 (Function) |
|---------|----|----------|
| `Hello` | `0x00` | 宣布支持的能力 (Capabilities)。 |
| `Disconnect` | `0x01` | 发起优雅的断开连接。 |
| `Ping` | `0x02` | 检查对等节点是否存活。 |
| `Pong` | `0x03` | 响应 `Ping`。 |

#### 断开连接原因 (Disconnect Reasons)

| 代码 (Code) | 原因 (Reason) |
|------|--------|
| `0x00` | 请求断开连接。 |
| `0x02` | 违反协议。 |
| `0x03` | 无用的对等节点。 |
| `0x05` | 已经连接。 |
| `0x06` | 不兼容的协议版本。 |
| `0x09` | 预料之外的身份。 |




### 应用层子协议 (Application-Level Subprotocols)

- **RLPx 支持多个应用层子协议**，这些子协议实现了以太坊节点之间的专业化通信。
- 这些子协议**构建在 RLPx 传输层之上**，用于数据交换、状态同步和轻客户端支持。

#### 常见的以太坊子协议 (Common Ethereum Subprotocols)

| **子协议 (Subprotocol)** | **目的 (Purpose)** |
|---------------|------------|
| **以太坊有线协议 (Ethereum Wire Protocol, `eth`)** | 处理**区块链数据交换**，包括区块传播和交易中继。 |
| **以太坊快照协议 (Ethereum Snapshot Protocol, `snap`)** | 用于**状态同步**，允许节点下载部分状态树。 |
| **轻量级以太坊子协议 (Light Ethereum Subprotocol, `les`)** | 支持**轻客户端**，使它们能够从全节点请求数据，而无需存储完整状态。 |
| **门户网络 (Portal Network, `portal`)** | 适用于轻量级客户端的去中心化**状态、区块和交易检索网络**。 |


### 延伸阅读 (Further Reading)
* [Geth devp2p 文档 (Geth devp2p docs)](https://geth.ethereum.org/docs/tools/devp2p)
* [以太坊 devp2p GitHub (Ethereum devp2p GitHub)](https://github.com/ethereum/devp2p)
* [以太坊网络层 (Ethereum networking layer)](https://ethereum.org/en/developers/docs/networking-layer/)
* [以太坊地址 (Ethereum Addresses)](https://ethereum.org/en/developers/docs/networking-layer/network-addresses/)
* Alchemy (2022). [以太坊交易如何传播（广播）？ (How are Ethereum transactions propagated (broadcast)?)](https://www.alchemy.com/overviews/transaction-propagation)
* Andrew S. Tanenbaum, Nick Feamster, David J. Wetherall (2021). *Computer Networks*. 第 6 版. Pearson. 伦敦.
* Kevin Leffew (2019). "以太坊协议中 Kademlia 的应用" (Kademlia usage in the Ethereum protocol). [*Kademlia 在各种去中心化平台中的应用简要概述* (A brief overview of Kademlia, and its use in various decentralized platforms)]. Medium.
