# 协议设计哲学 (Protocol Design Philosophy)

这些是激发以太坊 (Ethereum) 架构和实现工作的核心原则：

- **简单性 (Simplicity)**：
  从诞生之初，以太坊协议的设计就考虑到了简单性，并制定了在沿途添加功能的宏伟路线图。这源于这样一种理念：任何普通程序员在理想情况下都应该能够理解并实现整个规范，这主要是为了最大限度地减少个人或精英开发者群体对协议的影响。虽然由于协议的重大修改，这种叙述已慢慢发生改变，但得益于模块化 (modularity) 和清晰的规范，简单性仍然得到了维持。

- **通用性 (Universality)**：
  以太坊设计哲学的基本教条之一是以太坊没有***功能***。以太坊提供了一个内部图灵完备虚拟机 (Turing-complete virtual machine)，称为 [EVM](/wiki/EL/evm.md)，您可以使用它来构建任何可以进行数学定义的智能合约或交易类型。以太坊旨在成为一个平台，让新时代的开发者能够构建去中心化且真正无需信任的应用程序，而无需担心底层复杂的结构。

- **模块化 (Modularity)**：
  使以太坊协议模块化，对于使其**适应未来 (future-proof)** 至关重要。虽然以太坊远非完美，但与协议共存的还有持续且严密的研究与工程努力。在开发过程中，应该能够很容易地在某个地方对协议进行微调，并让应用栈继续运行而无需进行任何进一步的修改。诸如 Dagger、Patricia 树和 SSZ 等创新已被实现为独立的库，并做到了功能完备，即使以太坊不需要某些功能，以便使它们也能在其他协议中使用。通过 [原始邓克分片 (Proto-Danksharding)](/wiki/research/scaling/core-changes/eip-4844.md) 等功能，以太坊为 Layer 2 链的扩容提供了基石。模块化源于 [封装复杂度 (encapsulated complexity)](#https://vitalik.eth.limo/general/2022/02/28/complexity.html) 的概念。当一个系统由子系统组成时，就会发生封装复杂度——子系统内部是复杂的，但通过高级接口向外提供服务。这在子系统的选择和单个组件的更好调试方面带来了极大的灵活性。

- **非歧视性 (Non-discriminant)**：
  以太坊诞生于诸如 [**开源软件 (FOSS)**](https://www.fsf.org/about/what-is-free-software) 和 [**密码朋克 (Cypherpunk)**](https://en.wikipedia.org/wiki/Cypherpunk) 等各种运动所奠定的支柱之上。非歧视性是以太坊设计哲学织锦的基石。该协议并不试图主动限制或阻止特定类别的用途，并且协议中的所有监管机制都旨在直接监管对协议本身的损害，而不是试图反对特定的、不讨喜的应用程序。您甚至可以在以太坊之上运行一个死循环脚本，只要您愿意在协议定义的合理限制内为其支付每次计算步骤的交易费用。

- **敏捷性 (Agility)**：
  以太坊协议的细节并非一成不变。以太坊改进提案 (Ethereum Improvement Process) 是提出协议新更改的开放标准。对 EVM 和地址系统等高级结构进行修改时必须极其审慎，开发过程后期的计算测试可能会导致发现对算法或 EVM 的某些修改将实质性地提高可扩展性或安全性。如果发现任何此类机会，都将予以利用。

---

# 原则 (Principles)

以太坊协议随着时间的推移而演进和变化，但它始终遵循某些原则。这些原则反映了整个社区的价值观，并体现在以太坊的一些主要设计决策中。

- **管理复杂度 (Managing Complexity)**：以太坊协议设计的主要目标之一是最大限度地减少复杂度：使协议尽可能简单，同时仍然制作出一个能够实现有效区块链所需功能的区块链。以太坊协议在这方面远非完美，特别是因为它的很大一部分是在 2014-16 年设计的，当时我们的认识要少得多，但我们仍然在尽一切努力减少复杂度。
  然而，这一目标的挑战之一是复杂度难以定义，有时，您必须在引入不同类型复杂度的两个选择之间进行权衡并付出不同的成本：
    1. **三明治模型复杂度 (Sandwich model complexity)**：三明治模型侧重于简化以太坊架构的底层，并且以太坊的接口应该尽可能易于理解。在复杂度不可避免的地方，应将其推入协议的“中间层”，这些中间层不是核心共识的一部分，但终端用户也看不到——高级语言编译器、参数序列化和反序列化脚本、存储数据结构模型、`leveldb` 存储接口以及有线协议等。
    2. **封装复杂度 (Encapsulated complexity)**：当一个系统具有内部复杂但对外部呈现简单“接口”的子系统时，就会发生这种情况。当系统的不同部分甚至无法干净地分离，并且彼此之间存在复杂的交互时，就会发生系统复杂度 (systemic complexity)。通常，具有较少封装复杂度的选择也是具有较少系统复杂度的选择，因此有一种选择显然更简单。但在其他时候，您必须在一种复杂度与另一种复杂度之间做出艰难的选择。此时应该明确的是，如果复杂度被封装，其危险性就会降低。系统复杂度的风险并不是规范长度的简单函数；一个与每个其他部分都有交互的 10 行规范小片段，其带来的复杂度要大于一个在其他情况下被视为黑盒的 100 行函数。这里有几个 [示例](https://vitalik.eth.limo/general/2022/02/28/complexity.html)。
  复杂度去向的偏好顺序为：Layer 2 > 客户端实现 > 协议规范。

- **自由 (Freedom)**：用户不应在以太坊协议的用途上受到限制，我们不应该试图根据其目的的性质来优先偏袒或排斥某些种类的以太坊合约或交易。这类似于“网络中立性 (net neutrality)”概念背后的指导原则。未遵循该原则的一个例子是比特币交易协议中的情况，即不鼓励将区块链用于“非标准”目的（例如数据存储、元协议），并且在某些情况下，为了攻击以“未经授权”方式使用区块链的应用程序，做出了明确的准协议更改（例如将 OP_RETURN 限制为 40 字节）。在以太坊中，相反强烈支持以大致兼容激励的方式设置交易费用的方法，从而使以产生膨胀的方式使用区块链的用户将自己活动成本内部化（即 [庇古税 / Pigovian taxation](https://en.wikipedia.org/wiki/Pigouvian_tax)）。

- **泛化 (Generalization)**：以太坊中的协议功能和操作码 (opcodes) 应该体现最大程度的底层概念，以便它们可以以任意方式进行组合，包括今天看起来没有用但以后可能会变得有用的方式，并且当不需要某些功能时，可以通过剥离部分功能来使底层概念的集合变得更加高效。遵循该原则的一个例子是，我们选择 LOG 操作码作为向去中心化应用（特别是轻客户端）馈送信息的方式，而不是像早期内部建议的那样简单地记录所有交易和消息——“消息”的概念确实是多个概念的结合体，包括“函数调用”和“外部观察者感兴趣的事件”，将两者分开是值得的。

- **我们没有功能 (We have no features)**：作为泛化的自然推论，我们经常拒绝将即使非常常见的高级用例构建为协议的固有部分，因为我们理解，如果人们真的想这样做，他们总是可以在合约内创建一个子协议（例如以以太币为支撑的子货币、比特币/莱特币/狗狗币侧链等）。这方面的一个例子是以太坊中缺乏类似于比特币的“锁定时间 (locktime)”功能，因为这种功能可以通过这样一种协议来模拟：用户发送“已签名的数据包”，并且如果数据包在某种合约特定的意义上有效，则可以将这些数据包喂入处理它们并执行某些相应功能的专用合约中。

---

# 区块链级协议 (Blockchain level protocol)

### **账户模型胜过 UTXO (Accounts over UTXOs)**
包括比特币及其衍生品在内的最早的区块链实现中，用户余额存储在基于未花费交易输出 (unspent transaction outputs, UTXOs) 的结构中。另一方面，以太坊使用基于账户的模型 (account-based model)。

> **UTXO**：未花费交易输出 (unspent transaction output, UTXO) 是数字货币模型子集中的一种独特元素。UTXO 代表了由发送者授权并可由接收者消费的特定数量的加密货币。

因此，系统中的用户“余额”是用户拥有能够产生有效签名的私钥的硬币集合的总值。基于账户的模型更加灵活，并且允许更复杂的交易。

以太坊遵循账户模型而非 UTXO 模型。虽然 UTXO 提供了更高程度的隐私，但它们也为像以太坊这样的系统引入了更多的复杂度。账户也是同质的 (fungible)，从而能够使去中心化交易所等实现具有更大的灵活性，这符合以太坊的最初宗旨。

#### 账户模型的优势
- **节省空间**：例如，如果一个账户有 5 个 UTXO，那么从 UTXO 模型切换到账户模型将使空间需求从 (20 + 32 + 8) * 5 = 300 字节（地址占 20 字节，交易 ID 占 32 字节，数值占 8 字节）减少到 20 + 8 + 2 = 30 字节（地址占 20 字节，数值占 8 字节，nonce 占 2 字节（见下文））。实际上，由于账户需要存储在 Patricia 树中（见下文），节省的空间并没有如此庞大，但它们仍然是巨大的。此外，交易可以更小（例如，以太坊中为 100 字节，而比特币中为 200-250 字节），因为每笔交易只需进行一次引用和一次签名，并产生一个输出。

- **极高的同质性**：UTXO 并不是完美同质的，因为一个 UTXO 可能会因为在交易中与被污染的 UTXO 一起使用而被污染，并且可以使用一些启发式方法来追踪硬币的历史。账户是完美同质的，因为任何硬币都可以被任何其他硬币替换。

- **简单性**：账户模型比 UTXO 模型更容易实现和理解。UTXO 需要更复杂的交易验证算法，且 UTXO 模型不如账户模型灵活和强大。例如，在 UTXO 模型中无法实现去中心化交易所，因为 UTXO 模型不允许存在未与特定 UTXO 绑定的“卖”单。

账户范式的一个弱点是，为了防止重放攻击 (replay attacks)，每笔交易必须有一个 [**随机数 (nonce)**](https://ethereum.stackexchange.com/questions/27432/what-is-nonce-in-ethereum-how-does-it-prevent-double-spending)，这样账户就可以跟踪已使用的 nonce，并且仅在交易的 nonce 比上次使用的 nonce 大 1 时才接受交易。这意味着即使是不再使用的账户也永远无法从账户状态中剪枝。解决该问题的一个简单方法是要求交易包含区块号，使其在一段时间后不可重放，并每隔一段时间重置一次 nonce。

### **默克尔-帕特里夏树 (Merkle Patricia Trie, MPT)**
以太坊的数据结构是“修改后的默克尔-帕特里夏树 (Merkle-Patricia Trie)”，之所以如此命名，是因为它借鉴了 PATRICIA（检索以字母数字编码的信息的实用算法）的一些特征，并且它是为了高效检索组成以太坊状态的项目而设计的。

默克尔-帕特里夏树是确定性的且可通过密码学进行验证的：生成状态根 (state root) 的唯一方法是通过计算状态的每个单独片段来得出，并且可以通过比较根哈希以及导向该根哈希的哈希值（默克尔证明，Merkle proof）来轻松证明两个相同的状态。相反，无法用相同的根哈希创建两个不同的状态，并且任何尝试以不同值修改状态的行为都会导致不同的状态根哈希。理论上，该结构为插入、查找和删除提供了 $O(\log(n))$ 效率的“终极目标”。

目前正在对能够启用更好功能和权衡的新数据结构进行持续研究。默克尔-帕特里夏树正在被考虑弃用，以被一种称为 [**向量承诺默克尔树 (vector commitment merkle trees)**](https://verkle.info/)（简称为 verkle / 沃克尔树）的更高效的数据结构所取代。

### **沃克尔树 (Verkle trees)**

> :warning: 沃克尔树 (Verkle trees) 目前是一个活跃的研究领域，本文可能未跟上最新进展。您可以在 [以太坊研究 (Ethereum Research) 论坛](https://ethresear.ch/t/portal-network-verkle/19339) 上参与开发和讨论。

MPT 目前被应用于在网络中发送成员身份证明的各种应用中，包括协议、公钥目录、比特币等加密货币以及安全文件系统。具有 $n$ 个叶子的默克尔树具有 $O(\log_2{n})$ 大小的证明。虽然 $O(\log{n})$ 的复杂度可能相当令人欣慰，但在大型树中，发送证明可能会占据带宽消耗的主导地位。具有分支因子 $k$ 的沃克尔树实现了 $O(kn)$ 的构建时间和 $O(\log_k{n})$ 的成员证明大小。这意味着分支因子 $k$ 提供了计算能力与带宽之间的权衡。

以太坊紧迫的问题之一是当前的状态大小。在撰写本文时，估计约为 1-2TB。节点无法将其保存在工作内存中，甚至也无法保存在较慢的永久存储中，因此，无状态性 (statelessness) 对于网络的增长变得至关重要。沃克尔树及其向量承诺允许小得多的证明（**称为见证数据，witnesses**）。证明者无需像默克尔树那样在每个层级提供所有“兄弟节点”的哈希值，而只需提供从每个叶子到根的路径上的所有父节点（加上一个额外的证明，称为可选项）。

### **递归长度前缀 (Recursive Length Prefix, RLP)**
完整的实现和细节可以在 [RLP 页面](/wiki/EL/RLP.md) 找到。

创建新序列化方案背后的基本原理在于其他方案的概率性质。RLP 通过简单且确定性的序列化解决了这一问题；并保证了绝对完美的字节级一致性。RLP 并不试图定义任何特定的数据类型，如布尔值、浮点数、双精度浮点数甚至整数——相反，它只是为了以嵌套数组的形式存储结构而存在。键/值映射 (maps) 也没有得到显式支持；支持键/值映射的半官方建议是将此类映射表示为 `[[k1, v1], [k2, v2], ...]`，其中 `k1, k2...` 使用字符串的标准顺序进行排序。

事实证明，数据结构对于序列化算法随着时间的推移完全匿名，对于布尔值和整数等固定长度的数据类型是低效的。以太坊 2.0 中引入了简洁序列化 (Simple Serialize, SSZ)，它支持可变大小和固定大小的数据类型，并具有默克尔化 (Merkleization) 等附加功能。

### **简洁序列化 (Simple serialize, SSZ)**

序列化是将数据结构转换为可以传输并在以后重构的格式的过程。SSZ 是以太坊 2.0 信标链中使用的序列化格式。设计为不是自描述的序列化方案——相反，它依赖于必须提前获知的模式 (schema)。SSZ 与 RLP 相比具有许多优势，例如对象的高效重新哈希和快速索引，这是 RLP 所缺乏的，会导致 $O(N)$ 复杂度。

根据 [Vitalik 的评论](https://ethresear.ch/t/replacing-ssz-with-rlp-zip-and-sha256/5706/12)，SSZ 试图解决的主要问题之一是 RLP 不允许默克尔化 (Merkleization)，这意味着剥夺了对任何内容进行简洁轻客户端证明的可能性。因此，无法实现无状态性——而无状态性仍然是当前以太坊研发的核心目标。

进一步的实现和细节可以在 [简洁序列化页面](/wiki/CL/ssz.md) 找到。

### **探寻最终性 (Hunt for Finality)**
在以太坊基于权益证明的共识机制中，最终性 (finality) 是指这样一种保证：在不销毁至少 33% 质押 ETH 总额的情况下，区块不能被更改或从区块链中移除。实现这一点的底层共识协议被称为 **Casper 友好最终性小工具 (Casper Friendly Finality Gadget, Casper FFG)**。有关与最终性相关的攻击的更多细节可以在 [此处](https://blog.ethereum.org/2016/05/09/on-settlement-finality) 找到。

- ***Casper FFG***
  [Casper FFG](https://arxiv.org/abs/1710.09437v4) 是提案机制之上的叠加层，负责通过选择代表规范交易账本的唯一链来最终确定区块。它采用 2014 年首次提出的 [罚没 (slashing)](https://blog.ethereum.org/2014/01/15/slasher-a-punitive-proof-of-stake-algorithm) 来实现这一目标。Casper 遵循了拜占庭容错 (BFT) 的传统，并进行了修改以实现 PoS。
  简单来说，每个验证者对检查点进行投票，经过两轮投票后，检查点被**最终确定**。所有最终确定的检查点都成为规范链（区块链历史的一部分）的一部分。虽然 Casper 通过对规范链中最新添加的区块进行证明来保证**最终性**，但它需要一种分叉选择规则，验证者通过对区块进行证明来发出对这些区块支持的信号。

- ***LMD GHOST***
  最新消息驱动的最贪婪最重观察子树算法 (Latest Message Driven Greediest Heaviest Observed Sub-Tree, LMD-GHOST) 是一种*分叉选择规则*，验证者通过证明区块来信号支持这些区块。这在某些方面类似于工作量证明网络中使用的分叉选择规则，在工作量证明网络中，完成工作量最多的分叉被选为规范链。

![LMD-GHOST-Algorithm](./img/lmt-ghost.png)

Gasper 是完整的权益证明协议，是以太坊实现的理想化抽象。它结合了 Casper FFG 和 LMD-GHOST 来驱动 Eth2 的共识机制。

### **使用 DHT (Using a DHT)**

![P2P Networks Comparison](./img/p2p-nets-comp.png)

DHT（分布式哈希表）的主要好处是查找在网络中仅产生对数级的通信开销。这使得它们适合在 p2p 网络中寻找（查询）内容。但是立即出现了一个问题，如果大多数节点都对相同的内容（最新区块）感兴趣，为什么我们需要在以太坊中*寻找*内容？基于每个共识时隙，链梢始终是相同的，并且只有一个区块需要进行广播。DHT 曾用于诸如 [BitTorrent](https://www.bittorrent.org/beps/bep_0005.html) 和 IPFS 等协议中，这些协议存储了广泛的内容，用户尝试*寻找*他们感兴趣的内容。以太坊网络中，DHT 用于寻找不同的节点（peer），而不是区块。

以太坊网络层中的发现协议使用 discv5，这是一种[基于 Kademlia 的 DHT (kademlia based DHT)](https://github.com/ethereum/devp2p/blob/master/discv5/discv5.md) 来存储 [ENR 记录 (ENR records)](https://github.com/ethereum/devp2p/blob/master/enr.md)。ENR 记录包含路由信息（互联网层），用于建立节点之间的连接。加入网络的节点使用*引导 (bootstrap)* 节点在 DHT 中中继对其自身 `node_id` 的查找查询。在此过程中，他们发现其他节点的 ENR 记录，这有助于他们填充路由表。常规情况下，节点还会查找随机的 `node_id` 以枚举网络，即寻找所有同伴。

以太坊网络中的区块是使用点对点技术栈的 gossip 协议进行分发的。在通过底层的 DHT 发现节点之后，节点使用叠加网络 ([gossipsub](https://github.com/libp2p/specs/blob/f25d0c22e5ef045c8c050bc91c297468de35f720/pubsub/gossipsub/gossipsub-v1.0.md)) 在整个网络中分发区块。叠加网络创建了自己的路由表，其中包含用于建立连接的路由信息以及叠加网络的特定信息（订阅的主题、扇出等）。这个叠加网络确实是一个无结构网络 (unstructured network)。

与仅仅连接到同伴（[好友对好友模型，friends-to-friends model](https://en.wikipedia.org/wiki/Friend-to-friend) 或 PEX）并直接下载/传播所需内容相比，DHT 还迈出了加入无结构网络的一步。
为什么要经过 DHT 的额外步骤在以后加入无结构网络？从引导的角度来看，Kademlia 提供了全局视图，而好友对好友 (f2f) 网络本质上只能提供网络的局部视图。非正式地，DHT 为节点加入网络提供了公共且非本地化的（可以说是稍微去中心化一些的）机制。这种使用结构化网络 (DHT) 引导到无结构网络的混合方法，在 BitTorrent 的 [同伴交换 (PEX)](https://www.bittorrent.org/beps/bep_0011.html) 协议中也可以观察到。[无结构网络](https://en.wikipedia.org/wiki/Peer-to-peer#Unstructured_networks) 被首选作为叠加网络，因为它们在高度动态（高流失率）的网络中具有很强的鲁棒性。

最重要的是，像 Kademlia DHT 这样的结构化网络的最大好处是其设计上的[简单性](https://github.com/ethereum/devp2p/blob/master/discv5/discv5-rationale.md#why-kademlia)。

# 参考文献 (References)

- Ethereum Builders, [Design Rationale](https://web.archive.org/web/20211121044757/https://ethereumbuilders.gitbooks.io/guide/content/en/design_rationale.html)
- Vitalik B., [Complexity](https://vitalik.eth.limo/general/2022/02/28/complexity.html)
- Dankrad F., [Why it's so important to go stateless](https://dankradfeist.de/ethereum/2021/02/14/why-stateless.html)
- John K., [Verkle Trees](https://math.mit.edu/research/highschool/primes/materials/2018/Kuszmaul.pdf)
- Vitalik B. et al., [Combining GHOST and Casper](https://arxiv.org/pdf/2003.03052)
- Vitalik B., [Slasher: A Punitive Proof-of-Stake Algorithm](https://blog.ethereum.org/2014/01/15/slasher-a-punitive-proof-of-stake-algorithm)
