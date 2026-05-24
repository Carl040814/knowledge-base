# ECDSA 简要介绍

现代密码学重新定义了我们所有数字交互中的信任——从通过加密保护银行账户登录，到通过数字证书验证您喜爱的应用程序的真实性——其重要性怎么强调都不为过。

公钥密码学（public key cryptography）是支撑这些交互的关键概念。它由一对密钥组成：

**公钥（Public key）**：广泛分发，任何人都可以用来验证某个实体的身份。
**私钥（Private key）**：机密信息，仅所有者知道，用于加密和签名消息。

**椭圆曲线密码学（Elliptic curve cryptography, ECC）** 是一种特定类型的公钥密码学，利用椭圆曲线的数学特性来创建更小、更高效的密钥。这在像以太坊这样的资源受限环境中尤为有利。在以太坊中，**椭圆曲线数字签名算法（Elliptic Curve Digital Signature Algorithm, ECDSA）** 用于验证已提交交易的合法性。

让我们考虑一个现实场景来理解 ECDSA 的实际运作方式。

Alice，一位勤奋的女商人，被绑架并囚禁在一个偏远岛屿上。绑匪要求支付 100 万美元的巨额赎金才能释放她。由于通讯方式有限，他们只提供了一张明信片，让她指示她的同事 Bob 进行资金转账。

Alice 考虑写上赎金金额并像支票一样在明信片上签名。然而，这种方法存在重大风险：绑匪可以轻易伪造明信片，夸大金额，欺骗 Bob 向他们汇更多钱。

Alice 需要一个健壮的方法来实现：

1. Bob 能够验证转账确实是她授权的，并
2. 确保明信片上的消息未被篡改。

本练习的目标是设计一种方法，让 Alice 创建一个只有她自己知道的 **密钥 🔑**。这个密钥对于她证明自己的身份以及向 Bob 确保消息的真实性至关重要。

一如既往，数学来救场。通过巧妙运用**椭圆曲线**，让我们探索 Alice 如何生成**密钥 🔑**。

## 椭圆曲线

椭圆曲线是由以下**方程描述的曲线**：

$$
y^2 = x^3 + ax+b
$$

其中 $4a^3 + 27b^2 \ne 0$ 以确保曲线是非奇异的（non-singular）。
上述方程被称为长方程的 **Weierstrass 标准形式（Weierstrass normal form）**：

$$
y^2 + a_1 xy + a_3 y = x^3 + a_2 x^2 + a_4 x + a_6
$$

示例：

<img src="https://epf.wiki/images/elliptic-curves/examples.gif" width="500"/>

观察到椭圆曲线关于 x 轴对称。

以太坊使用一条标准曲线，称为 [secp256k1](http://www.secg.org/sec2-v2.pdf)，参数为 $a=0$，$b=7$；即曲线：
$$y^2=x^3+7$$

<img src="https://epf.wiki/images/elliptic-curves/secp256k1.png" width="500"/>

## 群与域

### 群（Group）
在数学中，**群** 是一个集合 $G$，至少包含两个元素，且在二元运算下封闭，该运算通常称为**加法**（$+$）。当一个运算的结果也是该集合的成员时，我们说该集合在运算下是封闭的。

实数集合 $\\mathbb{R}$ 是群的一个熟悉例子，因为两个实数的算术加法是封闭的。

$$
 3 \\in \\mathbb{R},  5 \\in \\mathbb{R} \\\\
 3 + 5 = 8 \\in \\mathbb{R}
$$

## 域（Field）
类似地，**域** 是一个集合 $F$，至少包含两个元素，且在两种二元运算下封闭，通常称为**加法**（$+$）和**乘法**（$\\times$）。

换句话说，**域** 在加法和乘法下都是一个**群**。

椭圆曲线的有趣之处在于，曲线上的点构成一个群，即两个点的"加法"结果仍保持在曲线上。这种几何加法与算术加法不同，涉及通过选定的点（**P** 和 **Q**）画一条直线，并将得到的曲线交点（**R'**）关于 x 轴反射，得到它们的和（**R**）。

<br />
<img src="https://epf.wiki/images/elliptic-curves/addition.gif" width="500"/>

一个点（**P**）也可以与自身相加（$P+P$），此时直线变为 **P** 处的切线，反射得到其和（**2P**）。

<br />
<img src="https://epf.wiki/images/elliptic-curves/scalar-multiplication.png" width="500"/>

重复的点加法被称为**标量乘法（scalar multiplication）**：

$$
nP = \\underbrace{P + P + \\cdots + P}_{n\\ \\text{次}}
$$

## 离散对数问题

让我们利用标量乘法来生成**密钥 🔑**。这个密钥记为 $K$，表示基点 $G$ 与自身相加的次数，得到最终的公开点 $P$：

$$
P = K*G
$$

给定 $P$ 和 $G$，理论上可以通过有效逆转乘法来推导出密钥 $K$，类似于**对数问题（logarithm problem）**。

我们需要确保标量乘法不会泄露我们的**密钥 🔑**。换句话说，标量乘法应该在一个方向上是"容易"的，而在反方向上是"不可追踪"的。

时钟的类比有助于说明所需的单向性质。想象一个任务从中午 12 点开始，在 3 点结束。只知道最终时间（3）而不了解其他信息，就无法确定确切的持续时间。这是因为**模运算（modular arithmetic）**引入了"回绕"效应。该任务可能花了 3 小时、15 小时甚至 27 小时，模 12 后都得到相同的最终时间。

<br />
<img src="https://epf.wiki/images/elliptic-curves/clock.gif" width="500"/>

在**素数模（prime modulus）**下，这尤其困难，被称为**离散对数问题（discrete logarithm problem）**。

## 有限域上的椭圆曲线

到目前为止，我们隐含地假设了有理数域（$\\mathbb{R}$）上的椭圆曲线。通过离散对数问题确保**密钥 🔑** 的安全性，需要过渡到由**素数模**定义的有限域上的椭圆曲线。这本质上是通过对特定素数执行模约简（modular reduction），将曲线上的点限制在一个有限集合内。

为了本次讨论，我们将考虑定义在素数模为 **997** 的**任意有限域**上的 **secp256k1** 曲线：

$$
y^2 = x^3 + 7 \\pmod {997}
$$

<img src="https://epf.wiki/images/elliptic-curves/finite-field.png" width="500"/>

虽然有限域中曲线的几何表示与连续曲线相比可能显得抽象，但其对称性保持不变。此外，标量乘法仍然封闭，尽管"切线"现在由于模的性质而"回绕"。

<br />
<img src="https://epf.wiki/images/elliptic-curves/finite-scalar-multiplication.gif" width="500"/>

## 生成密钥对

Alice 终于可以使用有限域上的椭圆曲线生成密钥对了。

让我们在 [Sage](https://www.sagemath.org/) 中定义素数模 997 的有限域上的椭圆曲线。

```python
sage: E = EllipticCurve(GF(997),[0,7])
Elliptic Curve defined by y^2 = x^3 + 7 over Finite Field of size 997
```

通过在曲线上选择一个任意点来定义生成元点 $G$。

```python
sage: G = E.random_point()
(174 : 487 : 1)
```

椭圆曲线上的标量乘法定义了一个循环**阶为 $n$ 的子群（subgroup of order $n$）**。这意味着将子群中的任意点重复加 $n$ 次会得到无穷远点（$O$），它充当单位元（identity element）。

$$
nP  = O
$$

```python
sage: n = E.order()
1057
# 说明 n*G（或任意点）等于 O，用 (0 : 1 : 0) 表示。
sage: n*G
(0 : 1 : 0)
```

密钥对包括：

1. **密钥 🔑**（$K$）：从子群阶数 $n$ 中选取的随机整数。确保只有 Alice 能够生成有效签名。

Alice 随机选择 **42** 作为**密钥 🔑**。

```python
sage: K = 42
```

2. **公钥**（$P$）：曲线上的一个点，是**密钥 🔑**（$K$）与生成元点（$G$）进行标量乘法的结果。允许任何人验证 Alice 的签名。

```python
sage: P = K*G
(858 : 832 : 1)
```

我们已经确定了 Alice 的密钥对 $=[P, K] = [(858, 832), 42]$。

## ECDSA 实战

ECDSA 是数字签名算法（Digital Signature Algorithm, DSA）的一种变体。它基于消息的"指纹"（使用密码学哈希）来创建签名。

为了让 ECDSA 正常工作，Alice 和 Bob 必须建立一组公共的域参数（domain parameters）。本示例的域参数为：

| 参数                             | 值           |
| ------------------------------------- | --------------- |
| 椭圆曲线方程。          | $y^2 = x^3 + 7$ |
| 有限域的素数模。 | 997             |
| 生成元点，$G$。             | (174, 487)      |
| 子群的阶，$n$。       | 1057            |

重要的是，Bob 确信公钥 $P = (858, 832)$ 确实属于 Alice。

### 签名

Alice 打算按以下步骤对消息 **"Send $1 million"**进行签名：

1. 计算密码学哈希 **$m$**。

```python
sage: m = hash("Send $1 million")
-7930066429007744594
```

2. 对于每个签名，生成一个随机的**临时密钥对（ephemeral key pair） [$eK$, $eP$]** 以缓解暴露她的**密钥 🔑** 的[攻击](https://youtu.be/DUGGJpn2_zY?si=4FZ3ZlQZTG9-eah9&t=2117)。

```python
# 随机选择的临时密钥。
sage: eK = 10
# 临时公钥。
sage: eP = eK*G
(215 : 295 : 1)
```

临时密钥对 $=[eK, eP] = [10, (215, 295)]$。

3. 计算签名分量 **$s$**：

$$ s = k^{−1} (e + rK ) \\pmod n$$

其中 $r$ 是临时公钥 **(eP)** 的 x 坐标，即 **215**。请注意，签名同时使用了 Alice 的**密钥 🔑（$K$）**和临时密钥对 **[$eK$, $eP$]**。

```python
# 临时公钥的 x 坐标。
sage: r = int(eP[0])
215
# 签名分量 s。
sage: s = mod(eK**-1 * (m + r*K), n)
160
```

元组 $(r,s) =  (215, 160)$ 就是**签名对（signature pair）**。

然后 Alice 将消息和签名写在明信片上。

<img src="https://epf.wiki/images/elliptic-curves/postcard.jpg" width="500"/>

### 验证

Bob 通过从签名对 **$(r,s)$**、消息和 Alice 的公钥 **$P$** 中独立计算**完全相同的临时公钥**来验证签名：

1. 计算密码学哈希 **$m$**。

```python
sage: m = hash("Send $1 million")
-7930066429007744594
```

2. 计算临时公钥 **$R$**，并与 **$r$** 比较：

$$R =  (es^{−1} \\pmod n)*G + (rs^{−1} \\pmod n)*P$$

```python
sage: R = int(mod(m*s^-1,n)) * G  + int(mod(r*s^-1,n)) * P
(215 : 295 : 1)
# 比较临时公钥的 x 坐标。
sage: R[0] == r
True # 签名有效 ✅
```

如果 Alice 的绑匪修改了消息，密码学哈希就会改变，导致与原始签名不匹配而验证失败。

```python
sage: m = hash("Send $5 million")
7183426991750327432 # 哈希不同！
sage: R = int(mod(m*s^-1,n)) * G  + int(mod(r*s^-1,n)) * P
(892 : 284 : 1)
# 比较临时公钥的 x 坐标。
sage: R[0] == r
False # 签名无效 ❌
```

签名的验证让 Bob 确信消息的真实性，使他能够转账并解救 Alice。椭圆曲线拯救了这一天！

## 总结

就像 Alice 一样，[以太坊上的每个账户都使用 ECDSA 对交易签名](https://web.archive.org/web/20240229045603/https://lsongnotes.wordpress.com/2018/01/14/signing-an-ethereum-transaction-the-hard-way/)。然而，以太坊中的 ECC（椭圆曲线密码学）涉及额外的安全考量。虽然核心原理保持不变，但我们使用如 keccak256 等安全哈希函数以及更大的素数域，拥有 78 位数字：$2^{256}-2^{32}-977$。


本讨论是对椭圆曲线密码学的初步介绍。如需更细致的理解，请参考以下资源。

最后一点：**永远不要自己实现加密算法！** 使用经过信任的库和协议来保护您的数据和交易。

> ℹ️ 注意  
> ECDSA 面临来自量子计算机的潜在淘汰——了解[后量子密码学如何应对这一挑战。](/wiki/Cryptography/post-quantum-cryptography.md)

## 延伸阅读

**椭圆曲线密码学**

- 📝 Standards for Efficient Cryptography Group (SECG), ["SEC 1: Elliptic Curve Cryptography."](http://www.secg.org/sec1-v2.pdf)
- 📝 Standards for Efficient Cryptography Group (SECG), ["SEC 2: Recommended Elliptic Curve Domain Parameters."](http://www.secg.org/sec2-v2.pdf)
- 📘 Alfred J. Menezes, Paul C. van Oorschot and Scott A. Vanstone, [Handbook of Applied Cryptography](https://cacr.uwaterloo.ca/hac/)
- 🎥 Fullstack Academy, ["Understanding ECC through the Diffie-Hellman Key Exchange."](https://www.youtube.com/watch?v=gAtBM06xwaw)
- 📝 Andrea Corbellini, ["Elliptic Curve Cryptography: a gentle introduction."](https://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction/)
- 📝 William A. Stein, ["Elliptic Curves."](https://wstein.org/simuw06/ch6.pdf)
- 📝 Khan Academy, ["Modular Arithmetic."](https://www.khanacademy.org/computing/computer-science/cryptography/modarithmetic/a/what-is-modular-arithmetic)
- 🎥 Khan Academy, ["The discrete logarithm problem."](https://www.youtube.com/watch?v=SL7J8hPKEWY)

**椭圆曲线数学**

- 📘 Joseph H. Silverman, ["The Arithmetic of Elliptic Curves."](https://books.google.co.in/books?id=6y_SmPc9fh4C&redir_esc=y)
- 📝 Joseph H. Silverman, ["An Introduction to the Theory of Elliptic Curves."](https://www.math.brown.edu/johsilve/Presentations/WyomingEllipticCurve.pdf)
- 📘 Neal Koblitz, ["A Course in Number Theory and Cryptography."](https://link.springer.com/book/10.1007/978-1-4419-8592-7)
- 📝 Ben Lynn, ["Stanford Crypto: Elliptic Curves."](https://crypto.stanford.edu/pbc/notes/elliptic/)
- 📝 Rareskills.io, ["Elliptic Curve Point Addition."](https://www.rareskills.io/post/elliptic-curve-addition)
- 📝 John D. Cook, ["Finite fields."](https://www.johndcook.com/blog/finite-fields/)

**实用工具**

- 🎥 Tommy Occhipinti, ["Elliptic curves in Sage."](https://www.youtube.com/watch?v=-fRWR_QKzuI)
- 🎥 Desmos, ["Introduction to the Desmos Graphing Calculator."](https://www.youtube.com/watch?v=RKbZ3RoA-x4)
- 🧮 Andrea Corbellini, ["Interactive Elliptic Curve addition and multiplication."](https://andrea.corbellini.name/ecc/interactive/reals-add.html)

## 致谢

- 感谢 Michael Driscoll 为[动画椭圆曲线](https://github.com/syncsynchalt/animated-curves)所做的工作。
