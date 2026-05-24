# 密码学

> :warning: 本文为[存根](https://en.wikipedia.org/wiki/Wikipedia:Stub)，请[贡献](/contributing.md)并扩展此 wiki。

密码学研究者打造开发者所使用的武器或变革工具。他们运用高等代数来利用宇宙对现实施加的硬性限制，并设计出满足特定属性的密码学方案（cryptographic schemas）。在某种意义上，他们是现实黑客。他们通过底层数学来"黑入"现实，创造出遵循客观属性的系统。

## 以太坊中的密码学：核心原语与协议角色

以太坊的安全模型依赖于精心选择的密码学构造，每种构造都针对特定的约束条件——无论是交易验证、共识还是可扩展性。以下是关键方案及它们在协议层面的高层功能概述。其中部分方案在密码学子章节中有更详细的讨论。

1. ### ECDSA 与交易认证**
**用途：** 每笔以太坊交易必须使用 secp256k1 曲线上的 ECDSA（椭圆曲线数字签名算法）进行签名。

#### 为什么重要：

与账户抽象模型（ERC-4337）不同，EOA（外部拥有账户）的每次操作都需要签名。

v, r, s 签名格式存在历史遗留问题——传统交易使用 v ∈ {27, 28}，而 [EIP-155](https://eips.ethereum.org/EIPS/eip-155) 将 v 定义为 ∈ {0, 1} 以防止重放攻击（replay attacks）。

**注意事项：**

签名可塑性（signature malleability）在 EIP-155 之前是一个隐患（通过包含 chainID 修复）。

硬件钱包通常实现自定义签名逻辑（例如 Ledger 的 nonce 派生方式）。

2. ### Keccak-256：不仅仅是"一个哈希函数"

#### 协议角色：

区块哈希（包括 PoW（工作量证明）中的 mixHash）。

状态 Trie 键（Merkle-Patricia Trie 依赖 Keccak 进行节点引用）。

#### 设计选择：

以太坊在 NIST 标准化 SHA-3 之前就采用了 Keccak，导致细微差异（例如填充规则）。

曾考虑过 Blake2b 等替代方案，但因兼容性原因被拒绝。

3. ### PoS 中的 BLS 签名：权衡与优化
#### 为什么选择 BLS？

签名聚合（signature aggregation）允许将数千个验证者证明（validator attestations）压缩为单个 96 字节的证明。

BLS12-381 曲线平衡了性能和安全性（支持配对，但比 secp256k1 慢）。

实现细节：

以太坊的 BLS 规范（EIP-2335）强制执行严格的子群检查（subgroup checks）以避免恶意密钥攻击（rogue-key attacks）。

像 <name>blst</name> 和 <name>herumi</name> 等工具针对不同环境（如 x86 vs. ARM）进行了优化。

4. ### Verkle 树：状态证明的转变
**当前局限：** 存储的 Merkle 证明太大（每个证明约 1 KB）。

**Verkle 解决方案：**

用多项式承诺（polynomial commitments，KZG）取代哈希，将见证数据（witness）大小减少到约 200 字节。

需要可信设置（trusted setup）（以太坊的 KZG 仪式）。

**待解决问题：**

过渡后如何处理历史证明？

证明验证的 gas 成本会改变吗？

5. ### ZKP 实践：不仅仅是 Rollup
**zkEVM 挑战：**

证明 EVM 执行需要处理可变操作码成本（例如 SSTORE vs. ADD）。

某些电路（例如 Keccak）以难以优化著称。

**超越 Rollup：**

轻客户端可以使用 ZK 证明进行同步委员会验证（PBS 提案）。

隐私池（privacy pools）（例如基于 ZK 的匿名交易）正在研究中。

6. ### 后量子准备：不只是理论
**直接风险：**

量子计算机可以伪造 ECDSA 签名，但要破解哈希函数（Keccak）则更困难。

**迁移路径：**

基于格的方案（lattice-based schemes）（例如 Dilithium）是领先候选方案，但密钥大小存在问题（每个密钥约 2 KB）。

混合方案（hybrid approaches）（例如 BLS + SPHINCS+）可能成为过渡桥梁。


- [BLS12-381 Keystore](https://eips.ethereum.org/EIPS/eip-2335)
- [Vitalik 的 ZK-SNARK 解释](https://vitalik.eth.limo/general/2021/01/26/snarks.html)
- [以太坊中的 Secp256k1](https://ethereum.org/en/developers/docs/evm/)
- [Verkle 树](https://verkle.info/)
- [不同类型的 ZK-EVM](https://vitalik.eth.limo/general/2022/08/04/zkevm.html)
- [ZK-SNARKS](https://eprint.iacr.org/2018/046)

https://summerofprotocols.com/wp-content/uploads/2023/12/53-BEIKO-001-2023-12-13.pdf
