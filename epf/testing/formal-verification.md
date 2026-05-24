# 形式化验证简要介绍 (A brief introduction to formal verification)

## 概述 (Overview)

形式化方法 (Formal methods) 是用于软件和硬件系统数学分析的技术。形式化方法的哲学根源可以追溯到古希腊，柏拉图在其《智者篇》中对形式理论的探讨；在整个 17 世纪，数学家通过抽象代数进一步发展了这一概念。德国博学家戈特弗里德·莱布尼茨 (Gottfried Leibniz) 的远见为我们现在所说的形式化推理奠定了基础。在 19 世纪，[乔治·布尔 (George Boole)](https://en.wikipedia.org/wiki/George_Boole) 在分析方面的工作以及 [戈特洛布·弗雷格 (Gottlob Frege)](https://en.wikipedia.org/wiki/Gottlob_Frege) 在命题逻辑 (propositional logic) 方面的工作为形式化方法奠定了基础。

在形式化方法中，形式化验证 (formal verification) 是一种*验证技术*，它有助于找到一个简单问题的答案：“系统是否正确地满足了其要求的规范？”。它通过将[系统抽象 (abstracting a system)](https://in.mathworks.com/discovery/abstract-interpretation.html)为数学模型并证明或证伪其正确性来实现这一点。

“系统”被定义为能够执行其外部接口所提供的所有功能的机制。对于一个系统而言，“不变量 (invariant)”是指无论其当前状态如何都保持不变的属性。例如，自动售货机的一个不变量是：任何人都不应该能够免费分发商品。形式化验证通过检查系统所有不变量是否保持正确来测试系统的正确性。

这种对系统进行严格审查的方法在 [EAL 评估保障等级 (Evaluation Assurance Level, EAL)](https://en.wikipedia.org/wiki/Evaluation_Assurance_Level#EAL7:_Formally_Verified_Design_and_Tested) 上获得了最高排名 (EAL7)，这标志着它对安全的深远影响。

形式化验证的类型：

- **模型检测 / 基于断言的检测 (Model checking / assertion-based checking)**：将系统建模为有限状态机 (finite state machine)，并使用命题逻辑 (propositional logic) 验证其正确性和活性 (liveness)。
- **时序逻辑 (Temporal logic)**：对命题随时间变化的系统进行建模。
- **等价性检查 (Equivalence checking)**：验证同一规范但不同实现的两个模型是否产生相同的结果。

## 流行工具 (Popular tools)

### Coq

[Coq](https://coq.inria.fr/) 是一个被广泛采用的开源证明管理系统。它曾被用于规范并形式化验证 C 语言的 CompCert 编译器。该编译器被用于[开发对安全性要求极高的程序](https://www.inria.fr/en/compcert-software-program-receives-prestigious-award)，如飞机、汽车和核电站。

### TLA+

[TLA+](https://lamport.azurewebsites.net/tla/tla.html) 是由图灵奖获得者莱斯利·兰伯特 (Leslie Lamport) 开发的一种形式化规范语言。它主要用于对并发和分布式系统进行建模。亚马逊网络服务 [使用 TLA+ (uses TLA+)](https://www.amazon.science/publications/how-amazon-web-services-uses-formal-methods) 来验证其分布式系统的鲁棒性。

### Alloy

[Alloy](https://alloytools.org/) 是一种用于软件建模的开源语言和分析器。值得注意的是，闪存文件系统设计曾使用 Alloy [针对 POSIX 标准进行了分析](https://eskang.github.io/assets/papers/ijsi09_kang_jackson.pdf)。

### Z3

[Z3](https://www.microsoft.com/en-us/research/project/z3-3/) 是由微软研究院开发的符号逻辑求解器 (symbolic logic solver)。它被广泛应用于软件工程应用程序中，涵盖[程序验证 (program verification)](https://www.aon.com/cyber-solutions/aon_cyber_labs/exploring-soliditys-model-checker/)、编译器验证、测试、模糊测试 (fuzzing) 和优化。

## 示例 (Example)

系统形式化验证的第一步是选择性地对系统进行抽象，以创建用于正确性测试的专注模型。

[迪杰斯特拉 (Dijkstra)](https://en.wikipedia.org/wiki/Edsger_W._Dijkstra) 优雅地描述了这一过程：

> 我逐渐将一个程序视为一组有序的珍珠，即一串“项链”。最上面的珍珠在最抽象的层次上描述了程序，在所有较低的珍珠中，上面使用的一个或多个概念都在其下方的珍珠中得到了解释（精化），而最底部的珍珠最终用标准接口（=机器）解释了仍需解释的内容。这一族珍珠成为了一个给定的集合，可以串成一条合适的项链。

用于对交通控制器进行建模的 TLA+ 规范：

```bash
-------------- MODULE TrafficController --------------

CONSTANTS MaxCars
VARIABLES carsWaiting, greenSignal

Init == /\ carsWaiting = 0
        /\ greenSignal = FALSE

Arrive(car) == IF carsWaiting < MaxCars THEN carsWaiting' = carsWaiting + 1 ELSE UNCHANGED carsWaiting

Depart == IF carsWaiting > 0 THEN carsWaiting' = carsWaiting - 1 ELSE UNCHANGED carsWaiting

ChangeSignal == /\ carsWaiting > 0
                /\ greenSignal' = TRUE

Next == \/
         \E car \in {0, 1}: Arrive(car)
         \/ Depart
         \/ ChangeSignal

Invariant == carsWaiting <= MaxCars

Spec == Init /\ [][Next]_<<carsWaiting, greenSignal>> /\ []Invariant

=======================================================
```

理解 TLA+ 的语义对本讨论并不重要。以下是它所做工作的简要说明：

`Init` 初始化系统，使其没有车辆等待。`Arrive` 模拟车辆到达，如果未达到最大容量，则增加等待车辆的计数。相反，`Depart` 模拟车辆离开控制器，如果有等待车辆，则减少其计数。最后，`ChangeSignal` 规定如果有车辆等待，交通信号灯转为绿灯。

不变量 `Invariant == carsWaiting <= MaxCars` 确保等待的车辆数量永远不会超过 `MaxCars`（一个定义的常量）。

请注意，这种抽象是如何方便地忽略了红绿灯处所有不相关的交互（有人按喇叭吗？）。

**高效的抽象是一门艺术。**

## 以太坊与形式化验证 (Ethereum and formal verification)

安全性和活性保证对于以太坊的去中心化基础设施至关重要。形式化验证在验证以下内容的正确性方面起着关键作用：

- 协议的[执行层 (execution) 规范](/wiki/EL/el-specs.md)和[共识层 (consensus) 规范](/wiki/CL/cl-specs.md)。
- [客户端 (client)](/wiki/EL/el-clients.md) 的实现。
- 最终用户交互的链上智能合约 (smart contract) 应用程序。

### 协议验证 (Protocol verification)

形式化验证被 [Runtime Verification 团队](https://github.com/runtimeverification) 用于验证 [信标链规范 (beacon chain specification)](https://runtimeverification.com/blog/a-formal-model-in-k-of-the-beacon-chain-ethereum-2-0s-primary-proof-of-stake-blockchain) 以及 [Gasper 最终性机制 (Gasper finality mechanism)](https://runtimeverification.com/blog/formally-verifying-finality-in-gasper-the-core-of-the-beacon-chain)。

[KEVM](https://github.com/runtimeverification/evm-semantics) 基于 [K 框架 (K framework)](https://kframework.org/) 构建，用于起草形式语义并对 [以太坊虚拟机 (EVM)](/wiki/EL/evm.md) 规范进行正确性验证。

形式化验证是测试套件中必不可少的工具，曾被用于发现状态转换组件中微妙的[运行时数组越界错误 (array-out-of-bound runtime error)](https://consensys.io/blog/formal-verification-of-ethereum-2-0-part-1-fixing-the-array-out-of-bound-runtime-error)。

![Formal verification as part of testing suite](./img/fv-and-testing.jpg)
> 形式化验证是测试套件中瑞士奶酪模型 (Swiss cheese model) 的一片 – [来源：Codasip](https://codasip.com/2023/09/19/formal-verification-best-practices-to-reach-your-targets/)。

### 验证优化 (Verifying optimizations)

等价性检查 (Equivalence checking) 被广泛用于软件优化。例如，可以针对优化后的智能合约的前一版本测试其正确性，以确认优化没有引入任何意外行为。

### 智能合约验证 (Smart contract verification)

智能合约中的漏洞或 Bug 可能会产生灾难性的后果，导致资金损失并损害用户信任。像 [Certora Prover](https://docs.certora.com/en/latest/docs/prover/index.html) 和 [halmos](https://github.com/a16z/halmos) 这样的形式化验证工具有助于识别这些问题。

例如，Runtime Verification 形式化验证了[存款智能合约应用](https://runtimeverification.com/blog/formal-verification-of-ethereum-2-0-deposit-contract-part-1)并发现了一个[微妙的漏洞 (subtle bug)](https://github.com/ethereum/deposit_contract/issues/26)。

形式化验证一直是以太坊 [Solidity](https://soliditylang.org/) 语言不可分割的一部分。这里有来自 Solidity 团队的 Christian 在早期工作坊中的分享：

<iframe width="560" height="315" src="https://www.youtube.com/embed/rx0NPckEWGI?si=GYGPPGGA7aY2k4Ci" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

Solidity 编译器还实现了一种[基于 SMT（可满足性模理论，Satisfiability Modulo Theories）和 Horn 求解的形式化验证方法 (SMTChecker)](https://docs.soliditylang.org/en/latest/smtchecker.html)。

来自以太坊基金会形式化验证团队的 Leo 解释了如何使用这一功能：

<iframe width="560" height="315" src="https://www.youtube.com/embed/QQbWpN76HEg?si=CI0cPCVgAkfAM_V2" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## 形式化验证的挑战 (Challenges with formal verification)

形式化验证很难。该过程本身可能非常[复杂且耗时](https://www.hillelwayne.com/post/why-dont-people-use-formal-methods)，需要专业的技能和工具。此外，形式化验证只能保证模型本身的正确性，而不一定能保证底层的实现本身。模型与代码之间转换过程中的错误仍然可能会引入漏洞。

形式化验证依赖于对系统的高效抽象。而抽象很难。如果您在抽象中遗漏了一个重要细节，就可能会引入安全问题。出于这个原因，[工程师们通常会使用模糊测试 (fuzzing) 等互补的模拟方法](https://blog.trailofbits.com/2024/03/22/why-fuzzing-over-formal-verification/)，使用随机输入来测试系统。

尽管存在这些挑战，形式化验证仍然是一种设计安全高效系统的强大技术。我们用迪杰斯特拉的这句富有洞察力的名言来结束本章：

> “测试只能证明 Bug 的存在，而无法证明其不存在！”

## 资源 (Resources)

### 🗣️ 演讲 (Talks)

- Grigore R., [Formal Verification of Smart Contracts and Protocols: What, Why, How](https://www.youtube.com/watch?v=xggtkB7w3es)
- Roberto S.,[Formal Specification and Verification of the Distributed Validator Technology protocol](https://www.youtube.com/watch?v=xdEo5Tc6TiY)
- Grant P., [Towards Imandra Contracts: Formal verification for Ethereum](https://www.youtube.com/watch?v=xeg_Q5uN73Q)
- Rikard H., [Formal Methods for the Working DeFi Dev](https://www.youtube.com/watch?v=ETlNhV9jYJw)
- Dimitar D. et al., [Formal Verification of Smart Contracts Made Easy](https://www.youtube.com/watch?v=tq5XH3JedqM)
- Yoichi H., [Formal Verification of Smart Contracts](https://www.youtube.com/watch?v=cCUGMAnCh7o)
- Yan M., [Ethereum Bugs Through the Lens of Formal Verification](https://www.youtube.com/watch?v=Ru6X043Q63U)
- Pawel S., [Formal verification applied (with TLA+)](https://www.youtube.com/watch?v=l9XZYI3jta0)

### 📄 出版物 (Publications)

- NASA, [What is formal methods](https://shemesh.larc.nasa.gov/fm/fm-what.html)
- Andrew H., [Formal Verification, Casually Explained](https://ahelwer.ca/post/2018-02-12-formal-verification/)
- Bernie C., [A Brief History of Formal Methods](https://www.researchgate.net/publication/233960390_A_Brief_History_of_Formal_Methods)
- Martin D., [Martin Davis on Computability, Computational Logic, and Mathematical Foundations](https://link.springer.com/book/10.1007/978-3-319-41842-1)
- Ashish D., [A Brief History of Formal Verification](http://homepages.cs.ncl.ac.uk/brian.randell/NATO/nato1969.PDF)
- Serokell, [Formal Verification: History and Methods](https://serokell.io/blog/formal-verification-history)
- Amazon, [How Amazon Web Services Uses Formal Methods](https://cacm.acm.org/research/how-amazon-web-services-uses-formal-methods/)
- Codasip, [Formal verification best practices to reach your targets](https://codasip.com/2023/09/19/formal-verification-best-practices-to-reach-your-targets/)
- Siemens, [How Can You Say That Formal Verification Is Exhaustive?](https://blogs.sw.siemens.com/verificationhorizons/2021/09/16/how-can-you-say-that-formal-verification-is-exhaustive/)
- Siemens, [3 Notable Formal Verification Conference Papers of 2020](https://blogs.sw.siemens.com/verificationhorizons/2021/02/09/3-notable-formal-verification-conference-papers-of-2020/)
- Siemens, [Formal Verification Methods](https://verificationacademy.com/topics/formal-verification/)
- Stanford, [Introduction to First Order Logic](https://plato.stanford.edu/entries/logic-classical/)
- NYU, [Introduction to Satisfiability Modulo Theories](https://cs.nyu.edu/~barrett/pubs/BT14.pdf)
- Sebastian U, [A Formal Verification of Rust's Binary Search Implementation](https://kha.github.io/2016/07/22/formally-verifying-rusts-binary-search.html)
- Jack V., [Primer on TLA+](https://jack-vanlightly.com/blog/2023/10/10/a-primer-on-formal-verification-and-tla)
- Martin L., [Symbolic execution for hevm](https://fv.ethereum.org/2020/07/28/symbolic-hevm-release)
- The Royal Society, [Formal verification: will the seedling ever flower?](https://royalsocietypublishing.org/doi/10.1098/rsta.2015.0402)
