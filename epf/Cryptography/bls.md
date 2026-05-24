# Boneh-Lynn-Shacham (BLS) 签名

### TLDR;

- 权益证明（proof-of-stake）协议使用数字签名来识别参与者并让他们承担责任。
  - 信标链（Beacon chain）中以太坊的验证者使用 BLS 签名参与共识、签名区块、发布证明（attestations）等。
- BLS 签名可以被聚合在一起，使其在大规模环境下验证效率很高。
- 签名聚合使信标链能够扩展到数十万验证者。
- 执行层上普通的以太坊交易签名仍使用 ECDSA。然而，账户抽象钱包也可以使用 BLS。

BLS 是一种具有聚合属性的数字签名方案。给定一组签名（_signature_1_, ..., _signature_n_），任何人都可以生成一个聚合签名。对密钥和公钥也可以进行聚合。此外，BLS 签名方案是确定性的、不可塑的（non-malleable）且高效的。其简洁性和密码学特性使其适用于各种用例，特别是在需要最小存储空间或带宽的场景中。本页将涵盖 BLS 签名背后的通用思想和数学原理，并进一步讨论以太坊背景下的 BLS。

## BLS 如何工作？

BLS 签名的核心是通过椭圆曲线上的配对（pairings）实现的双线性映射（bilinear mapping）概念。关键组成部分是一个配对函数 $e$，定义在从椭圆曲线派生的两个群之间：

$$e: G_1 \\times G_2 \\rightarrow G_T$$

该函数可高效计算，且必须满足双线性属性：

- 对于 $G_1$ 中所有 $P,Q$ 和整数 $a$，双线性定义为：
  $$e(aP, Q) = e(P, Q)^a$$
  $$e(P, aQ) = e(P, Q)^a$$

- 此外，它必须对加法可分配：
  $$e(P + Q, R) = e(P, R) \\times e(Q, R)$$
  $$e(P, Q + R) = e(P, Q) \\times e(P, R)$$

这些属性使得签名聚合等密码学机制成为可能，这在区块链应用和密码学共识中是一个关键特性。

#### 为什么在数字签名中选择 BLS 而非 Schnorr 和 ECDSA？

传统的 ECDSA 签名（如比特币或以太坊交易中常用的）严重依赖 nonce 生成的随机性，并且需要单独验证所有涉及的公开密钥，计算量很大。专利到期后，Schnorr 签名成为一种替代方案，允许部分聚合，但仍缺乏 BLS 带来的全部效率优势。

BLS 签名利用双线性配对，提供了针对某些密码学攻击的健壮防护，并生成更短的签名。与 Schnorr 不同，BLS 不依赖随机数生成来保护签名，使其在本质上对随机性相关的漏洞更安全。

_请注意：虽然 BLS 签名本身不需要每次签名操作的随机 nonce（使其成为确定性签名），但 BLS 中生成私钥的初始步骤仍依赖于安全的随机数生成。与 ECDSA 不同（在 ECDSA 中 nonce 对每次签名维护安全至关重要，且 nonce 的随机性对于防止漏洞至关重要），BLS 在签名过程中避免了对此类 nonce 的需求。然而，生成私钥时的随机性仍然关键。这种初始随机性确保私钥安全且不可预测，这对密码学系统的整体安全性至关重要。_

#### BLS 签名生成与验证示例：

![展示 BLS 密钥对生成和验证的图表](https://epf.wiki/images/elliptic-curves/bls-alice.png)
*理解 BLS 签名工作原理的视觉辅助*

考虑 Alice 创建 BLS 签名。她从她的私钥 $a$ 开始，并使用椭圆曲线上的生成元点 $G$ 计算她的公钥 $P$：

$$P = aG$$

她将消息哈希并映射到曲线上的一点 $H(M)$。她的签名 $S$ 为：

$$S = a \\times H(M)$$

使用配对函数验证签名：

$$e(G, S) = e(P, H(M))$$

这可以证明如下：
$$e(G,S)=e(G,a×H(m))=e(a×G,H(m))=e(P,H(M))$$
其中 $G$ 是椭圆曲线上的生成元点。

该等式证明了签名确实是由与 $P$ 对应的私钥持有者创建的。

#### 示例

对于使用 BLS12-381 等曲线的 BLS 签名，示例值如下：

```
Message: "Hello"
Secret Key: 26daf744780a51072aa8de191259bf7ff080b8457512cfd0eedfb4f8c71b131d
Public Key: bfdab807246849b76b7bdf5229619b9ccb33713633644a48b7ab3a7e67af7c1ae9d597a1c0fac6f61e63c1278b26c2f527be3d58bce95451b36f0c692ee90e1f
Signature: dee15784b458419b4b8bbdbb13838da13c27dccab6ef50f0dcb4ff7352048c0b
```

对于使用 secp256k1 等曲线的 ECDSA，有一个 $R$ 值和一个 $S$ 值，产生更长的签名，其示例值如下：

```
Message: "Hello"
Private Key: 2aabe11b7f965e8b16f525127efa01833f12ccd84daf9748373b66838520cdb7
Public Key (EC Point):
    x: 39516301f4c81c21fbbc97ada61dee8d764c155449967fa6b02655a4664d1562
    y: d9aa170e4ec9da8239bd0c151bf3e23c285ebe5187cee544a3ab0f9650af1aa6
Signature:
    R: 905eceb65a8de60f092ffb6002c829454e8e16f3d83fa7dcd52f8eb21e55332b
    S: 8f22e3d95beb05517a1590b1c5af4b2eaabf8e231a799300fffa08208d8f4625
```

### BLS 中的聚合：

BLS 的主要优势是能够将多个签名聚合成一个紧凑的签名。这在涉及多个交易或签名者的场景中特别有用，大大减少了验证所需的区块链空间和计算量。例如如果有 100 个交易，每个签名用 $S_i$ 表示，且每个关联一个公钥 $P_i$（以及消息 $M_i$），BLS 允许将它们合并为一个而不是存储 100 个独立签名：

$$S = S_1 + S_2 + \\ldots + S_{100}$$

然后可以使用（乘法操作）验证：
$$e(G,S)=e(P_1,H(M_1))⋅e(P_2,H(M_2))⋅…⋅e(P_{100},H(M_{100}))$$

对此聚合签名的验证将涉及公钥和消息哈希的相应聚合，保持所有单个签名的完整性和不可否认性。

## 以太坊中的 BLS 签名

就区块链而言，数字签名通常利用椭圆曲线群。以太坊主要使用 [ECDSA](/wiki/Cryptography/ecdsa.md) 签名和 [secp256k1](https://www.secg.org/sec2-v2.pdf) 曲线，而信标链协议采用基于 [BLS12-381](https://hackmd.io/@benjaminion/bls12-381) 曲线的 BLS 签名。与 ECDSA 不同，BLS 签名利用了某些椭圆曲线的一个独特特性，称为"[配对](https://medium.com/@VitalikButerin/exploring-elliptic-curve-pairings-c73c1864e627)"。这允许聚合多个签名，提高了共识协议的效率。虽然 ECDSA 签名处理[快得多](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-bls-signature-04#section-1.1)，但 BLS 签名的聚合能力为区块链可扩展性和共识效率提供了显著优势。

创建和验证 BLS 签名的过程很直接，涉及一系列可以通过图表、描述和数学原理来说明的步骤，尽管理解数学细节对于实际应用并非必需。

BLS 数字签名过程中有 **4** 个组成部分。

- **密钥（secret key）：** 协议中的每个参与者，特别是验证者，都拥有一个密钥，也称为私钥。此密钥对于签名消息和维护验证者在网络内行动的机密性至关重要。
- **公钥（public key）：** 使用密码学方法直接从密钥派生出来的公钥，虽然与密钥相关联，但如果没有极大的计算量，无法从中反推出密钥。它作为验证者在协议中的公共身份，对所有参与者可见。
- **消息（message）：** 在以太坊中，消息由字节串组成，其结构和目的将在后续上下文中进一步探讨。初步可将这些消息理解为区块链协议内处理的基本数据单元。
- **签名（signature）：** 签名是密钥与消息结合的密码学过程的结果。此签名唯一标识消息是由密钥持有者创建的。通过使用相应的公钥验证签名，可以确认消息源自特定验证者，并且在签名后未被更改。

在数学术语中，我们使用 BLS12-381 椭圆曲线的 2 个子群：定义在基域 $F_q$ 上的 $G_1$，以及定义在域扩展 $F_{q^2}$ 上的 $G_2$。两个子群的阶都是 $r$，一个 77 位的素数。（任意选择的）$G_1$ 的生成元是 $g_1$，$G_2$ 的生成元是 $g_2$。

1. 密钥 $sk$ 是 $1$ 到 $r$ 之间的一个数字（技术上范围包括 $1$，但不包括 $r$。然而，非常小的 $sk$ 值将是极其不安全的）。
2. 公钥 $pk$ 是 $[sk]g_1$，其中方括号表示椭圆曲线群点的标量乘法。因此公钥是 $G_1$ 群的成员。
3. 消息 $m$ 是一个字节序列。在签名过程中，这将被映射到某个点 $H(m)$，该点是 $G_2$ 群的成员。
4. 签名 $\\sigma$ 也是 $G_2$ 群的成员，即 $[sk]H(m)$。

![展示我们如何在下图中描绘各组成部分的图表。](https://epf.wiki/images/elliptic-curves/bls-key.svg)
*密钥的钥匙。这是我们将在下图中描绘各组成部分的方式。同一对象的不同变体用不同的阴影表示。密钥在数学上是标量；公钥是 $G_1$ 群成员；消息根被映射到 $G_2$ 群成员；签名是 $G_2$ 群成员。*

### 密钥对

密钥对是一个密钥及其公钥。它们一起将每个验证者与其行为不可辩驳地关联起来。

每个验证者至少配备 1 个主"签名密钥"，用于例行操作如创建区块、提交证明等。根据其提款凭证（withdrawal credentials），验证者还可能有一个辅助的"提款密钥"，为增加安全性而离线存储。

密钥应在范围 $[1,r)$ 内随机生成，遵循 [EIP-2333](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-2333.md) 标准，该标准建议使用 IRTF BLS 签名标准草案中的 [`KeyGen`](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-bls-signature-04#section-2.3) 方法。虽然遵守此方法不是强制性的，但不鼓励偏离。大多数质押者使用以太坊基金会的 [`eth2.0-deposit-cli`](https://github.com/ethereum/eth2.0-deposit-cli) 工具生成他们的密钥。为了安全起见，密钥对通常存储在受密码保护的 [EIP-2335](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-2335.md) keystore 文件中。

密钥 $sk$ 是一个 32 字节的无符号整数。公钥 $pk$ 表示为 $G_1$ 曲线上的一个点，并在协议中以压缩格式序列化为 48 字节的字符串。

<a id="img_bls_setup"></a>

![公钥生成的图表。](https://epf.wiki/images/elliptic-curves/bls-setup.svg)
*验证者随机生成其密钥。然后从中派生出公钥。*

### 信标链中的签名

在以太坊的信标链中，唯一需要签名的消息是对象的[哈希树根（hash tree roots）](https://eth2book.info/capella/part2/building_blocks/merkleization/)。这些根被称为"签名根（signing roots）"，是 32 字节的字符串。 [`compute_signing_root()`](/wiki/CL/functions#compute_signing_root) 函数将哈希树根与特定的"域（domain）"相结合，增强了安全性。

<!-- 定义并链接 CL 中的上下文（域分离和分叉）-->

签名根被映射到 $G_2$ 群内的椭圆曲线点上。这个映射 $H(m)$（其中 $m$ 是签名根）将哈希转换为适合密码学操作的格式。这个复杂的过程在 [Hash-to-Curve 草案标准](https://datatracker.ietf.org/doc/draft-irtf-cfrg-hash-to-curve/)中有概述，但通常开发者依赖密码学库来高效处理此步骤。

#### 签名创建：

实际的签名涉及将 $G_2$ 点 $H(m)$ 乘以签名者的密钥 $sk$：

$$
\\sigma = [sk]H(m)
$$

如此生成的签名 $\\sigma$ 也是 $G_2$ 群的一部分，通常压缩为 96 字节的字符串以供实际使用。

![签名消息的图表。](https://epf.wiki/images/elliptic-curves/bls-signing.svg)
*验证者使用其密钥对消息签名，生成唯一的数字签名*

### 验证签名

要验证签名，需要相应验证者的公钥。此密钥在信标状态（beacon state）中随时可用，可通过验证者索引访问，确保密钥检索简单可靠。

验证过程很简洁：将消息、公钥和签名输入验证过程。如果签名是真实的——同时匹配公钥和消息——则被接受；否则由于潜在的损坏、密钥使用错误或消息篡改而被拒绝。

形式上，此验证利用椭圆曲线配对。对于 BLS12-381 曲线，配对将 $G_1$ 中的点 $P$ 和 $G_2$ 中的点 $Q$ 映射到群 $G_T$ 中的点：

$$
e: G_1 \\times G_2 \\rightarrow G_T
$$

配对表示为 $e(P, Q)$，对验证签名和公钥之间的对应关系至关重要：

$$
e(g_1, \\sigma) = e(pk, H(m))
$$

这利用[配对](/wiki/Cryptography/bls#how-bls-works)的基本属性检查使用密钥 $sk$ 签名的消息是否与观察到的签名 $\\sigma$ 匹配。

![验证签名的图表。](https://epf.wiki/images/elliptic-curves/bls-verifying.svg)
*验证使用验证者的公钥和原始消息来确认签名的真实性。*

**验证输出**：如果签名与公钥和消息都对齐，过程返回 `True`，确认其有效性。如果不符合，返回 `False`，表明签名的完整性或来源存在问题。

## 资源与参考

- [BLS and key-pairing](https://asecuritysite.com/encryption/js_bls)
- [BLS signatures and key-pairing concepts](https://www.youtube.com/watch?v=cVgJBdM5E2M)
- [BLS aggregation by Vitalik Buterin and Justin Drake](https://www.youtube.com/watch?v=DpV0Hh9YajU)
- [Pragmatic Signature Aggregation By Justin Drake](https://ethresear.ch/t/pragmatic-signature-aggregation-with-bls/2105?u=benjaminion)
- [Building blocks from Eth2 Handbook](https://eth2book.info/capella/part2/building_blocks/signatures/)
- [formal IETF Draft Standard](https://www.ietf.org/archive/id/draft-irtf-cfrg-bls-signature-05.html)
- [Pairing Friendly curves](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-pairing-friendly-curves-10)
- [Hashing to elliptic curves](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-hash-to-curve-09)
- [ERC2333](https://github.com/ethereum/ercs/blob/master/ERCS/erc-2333.md) 提供了一种基于熵种子（entropy seed）派生 BLS12-381 密钥树层级的方法。
- [ERC2334](https://github.com/ethereum/ercs/blob/master/ERCS/erc-2334.md) 定义了用于指定密钥用途的确定性账户层级。
- [ERC2335](https://github.com/ethereum/ercs/blob/master/ERCS/erc-2335.md) 指定了用于存储和交换 BLS12-381 密钥的标准 keystore 格式。
