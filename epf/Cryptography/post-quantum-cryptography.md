# 后量子密码学

经典密码学通过利用某些数学问题的固有难度来保护信息安全。这些包括素数分解、离散对数、图同构（graph isomorphism）、最短向量问题（shortest vector problem）等问题，属于称为["隐藏子群问题（Hidden Subgroup Problem, HSP）"](https://en.wikipedia.org/wiki/Hidden_subgroup_problem)的数学研究领域。

本质上，这些问题使得在没有"秘密"（私有）密钥的情况下，确定一个大群内秘密子群的结构（大小、元素）在计算上不可行。这种单向"陷门函数（trapdoor function）"被公钥密码学算法用于其安全性。

[RSA](<https://en.wikipedia.org/wiki/RSA_(cryptosystem)>) 的安全性依赖于**大素数的因式分解**。相比之下，[ECDSA](/wiki/Cryptography/ecdsa.md) 的安全性基于椭圆曲线**离散对数问题**。随着密钥大小的增加，解决这些隐藏子群问题都呈指数级变得更加困难，使经典计算机在计算上不可行。这种基本难度保护了加密数据。

然而，格局正在变化。

量子计算机利用量子力学原理，提供了新颖的计算方法。某些量子算法可以以指数级效率解决这些经典密码学问题，相较于其经典对手。这种新获得的能力对用经典密码学加密的数据安全构成了重大威胁。如果大规模量子计算机被建造出来，它们将能够破解目前使用的许多公钥密码学。

[Shor 算法](https://ieeexplore.ieee.org/document/365700)用于整数因式分解是量子计算最著名的应用。它在小于 $O(n^3)$ 的时间复杂度内对 n 位整数进行因式分解，相较于最佳经典算法有显著改进。

这就是后量子密码学（post-quantum cryptography）领域出现的背景。它旨在开发即使在强大量子计算机存在的情况下仍然安全的新算法。

## 时间线

根据[《2020 年量子威胁时间线报告》](https://globalriskinstitute.org/publication/quantum-threat-timeline-report-2020/)所做的调查，大多数专家认为，到 2030 年公钥密码学的威胁低于 5%。然而，预计到 2050 年风险将大幅增加至约 50%。

目前，最[先进的量子计算机](https://en.wikipedia.org/wiki/List_of_quantum_processors)拥有 <2000 个物理量子比特（physical qubits）。在一小时内（理想时间窗口）破解比特币的加密[大约需要 3.17 亿个物理量子比特](https://pubs.aip.org/avs/aqs/article/4/1/013801/2835275/The-impact-of-hardware-specifications-on-reaching)。

量子研究正在稳步推进；一位调查受访者指出：

> 情况并非总是如此[..]但我发现我的预测往往比实际发生的情况更悲观。我将此视为研究正在加速的迹象。

请注意，这些预测在某种程度上是主观的，可能无法反映真实的进展，因为大多数进展并未向公众开放。高级威胁行为者可能比公众更早获得强大的量子计算能力，并使用诸如[追溯解密（retrospective decryption）](https://en.wikipedia.org/wiki/Harvest_now%2C_decrypt_later)等策略。

### 2025 年

2025 年 2 月，微软宣布[在单个芯片上实现百万量子比特。](https://news.microsoft.com/source/features/innovation/microsofts-majorana-1-chip-carves-new-path-for-quantum-computing/)。[带上下文的视频解释](https://www.youtube.com/watch?v=jwnez8HdN7E)。

## 以太坊的后量子风险

以太坊账户由两层密码系统保护。私钥通过[椭圆曲线乘法](/wiki/Cryptography/ecdsa.md)生成公钥。该公钥使用 [keccak256](/wiki/Cryptography/keccak256.md) 进行哈希以派生以太坊地址。

直接的后量子威胁是能够逆转保护 ECDSA 的椭圆曲线乘法，从而暴露私钥。这使得所有外部拥有账户（externally owned accounts, EOA）容易受到量子攻击。假设将公钥映射到以太坊地址的哈希函数仍然安全，提取其私钥仍然具有挑战性，但仍然是脆弱的。

在实践中，大多数用户的私钥本身是一系列哈希计算的结果，使用 [BIP-32](https://github.com/bitcoin/bips/blob/b3701faef2bdb98a0d7ace4eedbeefa2da4c89ed/bip-0032.mediawiki)，它通过从主种子短语开始的一系列哈希生成每个地址。这使得揭示私钥的计算成本更加昂贵。

EthResearch 有一个[正在进行的提案](https://ethresear.ch/t/how-to-hard-fork-to-save-most-users-funds-in-a-quantum-emergency/18901)，用于在后量子紧急情况下进行硬分叉，关键行动包括：

1. 回滚在明显发生大规模盗窃的第一个区块之后的所有区块
2. 传统的基于 EOA 的交易被禁用
3. 添加新的交易类型以允许来自智能合约钱包的交易（例如 [RIP-7560](https://ethereum-magicians.org/t/rip-7560-native-account-abstraction/16664) 的一部分），如果尚不可用
4. 添加一种新的交易类型或操作码，通过它你可以提供一个 STARK 证明，证明你知道 (i) 一个私有原像 x，(ii) 来自 k 个已批准哈希函数列表中的一个哈希函数 ID `1 <= i < k`，以及 (iii) 一个公共地址 A，使得 `keccak(priv_to_pub(hashes[i](x)))[12:] = A`。STARK 还接受该账户新验证代码片段的哈希作为公共输入。如果证明通过，你的账户代码将被切换到新的验证代码，并且从那时起你将能够将其作为智能合约钱包使用。

然而，这种方法并不完美。一些用户仍会损失资金，因为并非所有攻击事件中的区块都会被回滚。这是因为如 [domothy 所强调的](https://ethresear.ch/t/how-to-hard-fork-to-save-most-users-funds-in-a-quantum-emergency/18901/14)，可靠地检测网络上的量子攻击极其困难：

> 想象一个大型交易所钱包被量子计算机耗尽。每个人自然都会认为这是交易所端某种安全故障。或者如果一个依赖离散对数假设的智能钱包被耗尽，智能合约 bug/漏洞利用会是首先想到的。或者量子能力的攻击者完全避开高价值目标，慢慢从各种大型 EOA 窃取资金，而我们甚至永远不知道量子攻击发生了。

此外，支撑 [EIP-4844](/wiki/research/scaling/core-changes/eip-4844.md) 的 KZG 承诺方案也需要升级以防止欺诈性承诺。

## 研究

后量子密码学是一个活跃的研究领域。多个组织正在致力于新后量子算法的原型设计、开发和标准化。

## NIST 后量子密码学

[NIST 后量子密码学标准化](https://csrc.nist.gov/projects/post-quantum-cryptography)进行了一项多年国际竞赛，以评估和标准化抗量子密码学算法。2024 年 8 月，NIST 发布了首批最终确定的 **PQC 标准**，作为联邦信息处理标准（Federal Information Processing Standards, FIPS）：

### 已发布标准（2024 年 8 月）

**密钥封装机制（Key encapsulation mechanism）：**

- **ML-KEM** ([FIPS 203](https://doi.org/10.6028/NIST.FIPS.203)) 源自 CRYSTALS-Kyber。一种**密钥封装机制（KEM）**：一组三个算法（KeyGen、Encaps、Decaps），通过公开通道建立共享密钥。基于**模学习带错误（Module Learning With Errors, MLWE）**问题。

| 参数集 | 安全强度 | 安全类别 |
|---|---|---|
| ML-KEM-512 | 128 位 | 1 |
| ML-KEM-768 | 192 位 | 3 |
| ML-KEM-1024 | 256 位 | 5 |

**数字签名算法：**

- **ML-DSA** ([FIPS 204](https://doi.org/10.6028/NIST.FIPS.204)) 源自 CRYSTALS-Dilithium。基于格的数字签名算法。

| 参数集 | 安全强度 | 安全类别 |
|---|---|---|
| ML-DSA-44 | 128 位 | 2 |
| ML-DSA-65 | 192 位 | 3 |
| ML-DSA-87 | 256 位 | 5 |

- **SLH-DSA** ([FIPS 205](https://doi.org/10.6028/NIST.FIPS.205)) 源自 SPHINCS+。NIST 的无状态基于哈希的数字签名标准。

  它由三个经过充分研究的组件构建：
  - **WOTS+**（Winternitz 一次性签名增强版），一次性签名原语
  - **XMSS**（扩展 Merkle 签名方案），基于 WOTS+ 构建的多次方案
  - **FORS**（随机子集森林），用于签名消息摘要的少量次数方案

  与 ML-DSA 不同，SLH-DSA 不需要**基于数论的困难假设**。安全性仅依赖于标准哈希函数属性（原像抗性和相关属性），使其能够抵抗量子攻击，而没有 Shor 算法可以利用的任何代数结构。

  每个安全级别提供两种变体：
  - `s` = 更小的签名，较慢的签名速度
  - `f` = 更大的签名，较快的签名速度

| 参数集                          | 安全类别 | 签名大小 |
|----------------------------------------|-------------------|----------------|
| SLH-DSA-SHA2-128s / SLH-DSA-SHAKE-128s | 1                 | 7,856 字节    |
| SLH-DSA-SHA2-128f / SLH-DSA-SHAKE-128f | 1                 | 17,088 字节   |
| SLH-DSA-SHA2-192s / SLH-DSA-SHAKE-192s | 3                 | 16,224 字节   |
| SLH-DSA-SHA2-192f / SLH-DSA-SHAKE-192f | 3                 | 35,664 字节   |
| SLH-DSA-SHA2-256s / SLH-DSA-SHAKE-256s | 5                 | 29,792 字节   |
| SLH-DSA-SHA2-256f / SLH-DSA-SHAKE-256f | 5                 | 49,856 字节   |

SHA2 和 SHAKE 变体仅在内部哈希函数实例化上不同（SHA-2 系列 vs FIPS 202 中的 SHAKE），安全级别或签名结构没有区别。

- **FN-DSA**（即将作为 [FIPS 206](https://csrc.nist.gov/presentations/2025/fips-206-fn-dsa-falcon)）源自 FALCON。全名：**基于 NTRU 格的快速傅里叶变换数字签名算法（Fast-Fourier Transform over NTRU-Lattice-Based Digital Signature Algorithm）**。一种 Hash-Then-Sign 范式的基于格的方案，签名产生一个接近从随机化消息哈希派生的目标的格点，使用 **FFT** 和 **LDL 树**进行离散高斯采样。这使得 FN-DSA 的签名和公钥显著小于 ML-DSA，适合带宽受限的环境，如证书链或具有严格大小限制的协议。
  
NIST 的[《NIST 后量子密码学标准化过程第四轮状态报告》](https://nvlpubs.nist.gov/nistpubs/ir/2025/NIST.IR.8545.pdf)（2025 年 3 月）总结了正在进行的第四轮。


### 后量子密码学联盟

[后量子密码学联盟（Post-Quantum Cryptography Alliance, PQCA）](https://pqca.org/)是由[Linux 基金会](https://www.linuxfoundation.org/press/announcing-the-post-quantum-cryptography-alliance-pqca)发起的一个开放协作倡议，旨在推动后量子密码学的进步和采用。

该倡议下的[开放量子安全（The Open Quantum Safe, OQS）](https://openquantumsafe.org/)项目是一个开源项目，旨在支持向抗量子密码学的过渡。

### Crypto Forum Research Group

互联网工程任务组（Internet Engineering Task Force）内的 [Crypto Forum Research Group](https://datatracker.ietf.org/rg/cfrg/about/) 已标准化了基于状态哈希的签名方案 ["XMSS: eXtended Merkle Signature Scheme"](https://datatracker.ietf.org/doc/rfc8391/)。

## 生产使用

以下试点项目和研究计划正在探索 PQC 在生产中的使用：

- [Anchor Vault](https://chromewebstore.google.com/detail/omifklijimcjhfiojhodcnfihkljeali) 是一个 Chrome 插件，允许使用 Lamport 签名为保护 ERC 代币添加抗量子证明。
- Signal 已在生产中实现了 ["Post-Quantum Extended Diffie-Hellman"](https://signal.org/docs/specifications/pqxdh/#introduction) 用于密钥协商协议。
- Chromium 开始支持 ["Hybrid Kyber KEM"](https://blog.chromium.org/2023/08/protecting-chrome-traffic-with-hybrid.html) 以保护传输中的数据。
- Apple 实现了 [PQ3](https://security.apple.com/blog/imessage-pq3/) 以保护 iMessage 免受量子攻击的密钥泄露。

## 资源

- 📝 Daniel J. Bernstein 等, ["Introduction to post-quantum cryptography"](https://pqcrypto.org/www.springer.com/cda/content/document/cda_downloaddocument/9783540887010-c1.pdf)
- 📝 Wikipedia, ["Quantum algorithm."](https://en.wikipedia.org/wiki/Quantum_algorithm)
- 📝 P.W. Shor, ["Algorithms for quantum computation: discrete logarithms and factoring."](https://ieeexplore.ieee.org/document/365700)
- 📝 NIST, ["Post-Quantum Cryptography."](https://csrc.nist.gov/projects/post-quantum-cryptography)
- 📝 ETHResearch, ["How to hard-fork to save most users' funds in a quantum emergency."](https://ethresear.ch/t/how-to-hard-fork-to-save-most-users-funds-in-a-quantum-emergency/18901)
- 📝 ETHResearch, ["ETHResearch: Post-Quantum"](https://ethresear.ch/tag/post-quantum)
- 📝 Vitalik Buterin, ["STARKs, Part I: Proofs with Polynomials."](https://vitalik.eth.limo/general/2017/11/09/starks_part_1.html)
- 📝 Wikipedia, ["Lamport's Signature."](https://en.wikipedia.org/wiki/Lamport_signature)
